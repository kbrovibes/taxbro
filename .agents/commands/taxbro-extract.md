Extract all tax-relevant information from source documents and build the TaxBro Knowledge Graph.

**This is the foundational skill. Run it after /taxbro-init and before any other analysis.**
It reads every document once, extracts only the key facts needed for tax analysis, and writes
a compact structured knowledge graph. All downstream skills read from the graph — not from
raw documents. This means iterating on analysis is fast: extract once, analyze many times.

Usage:
  /taxbro-extract              — full extraction (reads all documents)
  /taxbro-extract --update     — re-extract only files that are new or changed since last run
  /taxbro-extract --section W2 — re-extract one section only (W2, FBAR, PFIC, FTC, CHILDCARE, RENTAL)

⚠️ ABSOLUTE RULE: NEVER DELETE ANY FILE. NEVER run rm, rmdir, or any deletion.
   Only permitted operations: read files, write TAXBRO/US-knowledge-graph.md.

⚠️ MANDATORY OUTPUT: The Computed Totals section MUST be written. If it is missing, ALL
   downstream skills (/taxbro-worksheets, /taxbro-visualize, /taxbro-checklist) will produce
   wrong results. The extraction is NOT complete until Computed Totals is written and verified.

---

## Step 1 — Setup

1. Read `.current-session` for SOURCE_FOLDER. If missing, stop and ask user to run /taxbro-init.
2. Read `{SOURCE_FOLDER}/TAXBRO/file-index.md` if it exists (produced by /taxbro-init).
   This gives you the document inventory. If missing, run a fresh discovery:
   `find "{SOURCE_FOLDER}" -follow -type f -not -path "*/TAXBRO/*" -not -name ".DS_Store"`
3. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` if it exists (for --update mode only).
4. Read `{SOURCE_FOLDER}/CLAUDE.md` and any subfolder CLAUDE.md (e.g. TAX-RETURNS-2026/US Taxes/CLAUDE.md)
   to understand the filer context: filing status, expected accounts, known edge cases, tax year.
5. Determine IRS Pub 54 average annual exchange rate for relevant currencies for the tax year
   (e.g., INR/USD, CAD/USD). Use IRS Publication 54 Appendix. If unavailable, note "rate pending"
   and mark converted values as (~est).

---

## Step 1b — Two-Pass Triage (Speed Optimization)

**Most documents do NOT need full multi-page parsing.** Use a two-pass approach:

### Pass 1: Triage (read first 1-2 pages only)
For EVERY document, read only the first 1-2 pages (or first 50 lines for text/CSV/XLS).
Classify each into one of:

| Classification | Action | Examples |
|---|---|---|
| **FULL-PARSE** | Read entire document in Pass 2 | W-2, 1099-B with many transactions, CAMS CAS, AIS, bank statements with 12 months |
| **HEADER-ONLY** | First page has all needed data — done | 1099-INT, 1099-SA, 5498-SA, 1098, simple 1-page forms |
| **SUMMARY-ONLY** | Skip to summary/totals page | 1099 consolidated (skip trade detail, read summary), Fidelity supplemental |
| **REFERENCE** | Note existence but extract nothing | Informational docs, duplicates, irrelevant files |
| **WORKING-FILE** | Pre-computed summary by user/agent | `2025-TAX-RETURN-SUMMARY.md`, working spreadsheets — use as verification source |

**Key triage rules:**
- **W-2 PDFs**: Always FULL-PARSE. They are 1 page each. Read EVERY box.
- **1099-B trade detail (multi-page)**: SUMMARY-ONLY — skip individual trades, read category totals page.
  Only go back to trades if you need quarterly attribution or wash sale detail.
- **1099-INT / 1099-SA / 5498-SA / 1098**: HEADER-ONLY — all data is on page 1.
- **Bank statements (12 monthly PDFs)**: Read first and last page of each to find peak and Dec 31 balances.
  Do NOT read every transaction page.
- **CAMS CAS**: SUMMARY-ONLY — skip transaction history, read portfolio valuation summary.
- **AIS**: FULL-PARSE — need category totals from multiple sections.
- **Childcare receipts**: HEADER-ONLY — provider name, EIN, annual total.
- **Prior year return**: SUMMARY-ONLY — skip all forms, read only carryforward items.

### Pass 2: Deep Extract
Process only FULL-PARSE and SUMMARY-ONLY documents. For HEADER-ONLY documents,
extraction is already complete from Pass 1.

**This approach typically reads ~30% of total page volume vs. reading everything.**

---

## Step 2 — Extract by Category

### Parallel Extraction (Claude Code)

When running in Claude Code, use the Agent tool to extract categories in parallel.
The categories below are **independent** — they read different documents and produce
separate knowledge graph sections. Launch them as parallel subagents:

**Batch 1 (launch simultaneously):**
- Agent 1: W-2 + 1099 Consolidated + 1099-INT + 1099-SA + 5498-SA + 1099-R
- Agent 2: Foreign Bank Statements + Interest Certificates + Form 16A / TDS Certificates
- Agent 3: CAMS CAS / Mutual Fund Statements (PFIC) + AIS/TIS
- Agent 4: 1098 (Mortgage) + Childcare Provider Statements + Advance Tax Challans + Prior Year Return

Each agent receives: SOURCE_FOLDER path, file list for its category, exchange rate,
filer context from CLAUDE.md, AND the triage classification from Step 1b.
Each agent returns its extracted section as markdown text.

**After all agents complete:**
- Merge all sections into the knowledge graph schema
- Run cross-document checks (Step 3) — these require data from multiple categories
- Compute the "Computed Totals" section (Step 4b) — **MANDATORY, NOT OPTIONAL**
- Write the final `US-knowledge-graph.md`

### Sequential Extraction (Gemini)

When running in Gemini CLI, process all categories sequentially in a single pass.
Gemini's large context window can hold all 73 documents at once, so parallelism
is unnecessary — just read everything and extract in order.

---

Read each document category below. Extract ONLY the specified facts — do not copy full content.
For every fact, record the source filename as a relative path from SOURCE_FOLDER.

### W-2 (Employment Income)

For each W-2 PDF found:
- Employer name and EIN
- Box 1: Wages
- Box 2: Federal income tax withheld
- Box 3: Social Security wages
- Box 4: Social Security tax withheld
- Box 5: Medicare wages and tips
- Box 6: Medicare tax withheld
- Box 10: Dependent care FSA (DCFSA) — if non-zero
- Box 12 codes: W (employer HSA), D (401k), C (GTL), DD (health insurance premiums), AA (Roth 401k)
- Box 14: Other (RSU, union dues, WAPFL, etc.) — note what's there
- Box 16/17: State wages and state income tax withheld (per state)

Do NOT extract employee SSN, address, or other PII.

#### 🔴 MANDATORY W-2 CROSS-CHECKS (run for every W-2)

These arithmetic checks catch PDF misreads. If ANY check fails, RE-READ the W-2 PDF
carefully and correct the values before proceeding.

| Check | Formula | Expected | Action if fails |
|---|---|---|---|
| SS tax rate | Box 4 ÷ Box 3 | = 6.20% (±0.01%) | Box 4 is wrong — recalculate or re-read |
| Medicare tax rate | Box 6 ÷ Box 5 | ≥ 1.45% (may be higher with Additional Medicare) | Box 6 is wrong — re-read |
| Wages vs Medicare | Box 5 ≥ Box 1 | Always true (Box 5 includes pre-tax 401k/FSA) | Box 1 and Box 5 may be swapped — verify |
| SS wage cap | Box 3 ≤ $176,100 (2025) | Always true | Box 3 is wrong |
| Fed Wh sanity | Box 2 ÷ Box 1 | 15%–40% typical for high earners | If outside range, verify — could indicate supplemental rate |
| Box 1 vs Box 5 gap | Box 5 − Box 1 | ≈ Box 12D (401k) + Box 12 other pre-tax | If gap doesn't reconcile, a box is misread |

**If Box 3 = $176,100 for employer (SS wage cap reached), then Box 4 MUST = $10,918.20.**
This is the single most important check — it catches the most common extraction error.

### 1099 Consolidated (Brokerage — Fidelity, Schwab, Chase, etc.)

For each consolidated 1099:
- Payer (institution name + account last 4)
- Box 1a: Total ordinary dividends
- Box 1b: Qualified dividends
- Box 2a: Total capital gain distributions
- Box 2b: Unrecaptured Section 1250 gain (if any)
- Box 3: Nondividend distributions
- Section 1256 gains (if any)
- 1099-INT embedded: Box 1 (interest), Box 4 (federal withholding), Box 8 (tax-exempt interest)
- 1099-B summary: total proceeds, total cost basis (covered), net gain/loss, wash sale disallowed amount

  **🔴 MANDATORY: Extract SHORT-TERM and LONG-TERM gains SEPARATELY.**
  The 1099-B summary always breaks down by:
  - Box A: Short-term, basis reported to IRS
  - Box B: Short-term, basis NOT reported (often RSU — flag for adjustment)
  - Box D: Long-term, basis reported to IRS
  - Box E: Long-term, basis NOT reported (often RSU — flag for adjustment)

  Record each category's proceeds, basis, wash sale, and net G/L independently.
  The ST/LT split is CRITICAL for tax computation — LTCG is taxed at 15-20% vs ordinary rates.
  **Failure to split ST/LT can cause ~$20K+ tax computation errors on large portfolios.**

  CRITICAL: RSU sales in Box B or E often show $0 basis. Scan for "Fidelity Supplemental" or
  "2025-TAX-RETURN-SUMMARY.md" in the source folder to find the actual adjusted cost basis
  (FMV at vest). Always flag $0 basis for preparer adjustment if supplemental basis isn't found.

  NOTE: Do NOT list individual trades. Extract category totals only. Note if Form 8949 required.

- **Quarterly capital gain attribution (for Form 2210):** From the 1099-B trade dates, determine
  which quarter(s) the bulk of capital gains were realized:
  - Q1: Jan–Mar | Q2: Apr–May | Q3: Jun–Aug | Q4: Sep–Dec
  - Record approximate gain per quarter (e.g., "Q4: ~$280K of $288K total")
  - This is critical for determining whether the annualized income method (Schedule AI)
    can reduce the underpayment penalty when income is concentrated in later quarters.
- Total federal withholding across the form

### 1099-INT (Standalone interest income)

- Payer name
- Box 1: Interest income
- Box 4: Federal withholding (if any)

### 1099-SA (HSA Distributions)

- Box 1: Gross distribution
- Box 5: Distribution type (1=normal, 2=excess, 3=disability, 4=death)
  Flag if type ≠ 1 (normal).

### 5498-SA (HSA Contributions)

- Box 2: Total HSA contributions for the year
- Note if contributions were made in prior year for this tax year (Box 3)

### 1099-R (IRA / Retirement Distributions) — if present

- Box 1: Gross distribution
- Box 2a: Taxable amount
- Box 7: Distribution code (important — determines taxability)

### 1098 (Mortgage Interest)

For each 1098:
- Lender name
- Box 1: Mortgage interest received
- Box 2: Outstanding mortgage principal (as of January 1)
- Box 3: Mortgage origination date
- Box 10: Property taxes paid through escrow (if any)
- Property address (optional — for Schedule E if rental)

### Childcare Provider Statements

For each provider:
- Provider name and address
- EIN (required for Form 2441 — flag if missing)
- Total amount paid during the year
- Period of care (months)
- Number of children

### Foreign Bank Account Statements (India: ICICI, SBI, HDFC, etc. / Canada: Scotiabank)

For each account, extract ONLY:
- Institution name, country, account type (savings/NRE/NRO/FD/checking)
- Account owner (Karthik / Vinaya / Joint)
- Account number last 4 digits only
- **Highest balance during the year** (in local currency + USD converted)
- **December 31 balance** (in local currency + USD converted)
- **Total interest credited during the year** (in local currency)
  Note: NRE accounts earn interest that is tax-free in India but taxable in US.
  Note: NRO accounts earn interest subject to Indian TDS — TDS rate is typically 30.9%.
- If a Fixed Deposit (FD) exists: note the FD amount, rate, maturity date (do not extract full history)

Do NOT list individual transactions. Do NOT copy statement tables.
If 12 monthly statements exist (like Scotiabank), read first and last page of each
to find the peak balance; report only the peak.

### Indian Interest Certificates / Form 16A / TDS Certificates

For each certificate:
- Institution and account type
- Account owner
- **Per-account breakdown** of interest and TDS (not just totals)
  - List EACH account number with its interest and TDS separately
  - Distinguish NRE (no TDS, no FTC) from NRO (TDS applies, FTC available)
- Total interest credited (INR) and total TDS deducted (INR)
- Certificate period
- Whether account is NRE (no TDS in India) or NRO (TDS applies)

**Also check for non-ICICI interest (SBI, HDFC, etc.):**
If bank statements show interest credits but no certificate exists, extract from statement
or from the working summary file (2025-TAX-RETURN-SUMMARY.md) and note the source.

### CAMS CAS / Mutual Fund Statements (PFIC)

For each fund in the consolidated statement:
- Fund name (full)
- AMC (Axis, HDFC, Mirae, SBI, etc.)
- ISIN (if shown)
- Folio number
- Owner (Karthik / Vinaya)
- Units held as of December 31
- NAV as of December 31 (INR)
- Value as of December 31 (INR + USD converted)
- **Any redemptions during the year**: date, units redeemed, proceeds (INR)
  A redemption triggers an excess distribution calculation — flag prominently.
- **Any dividends/distributions during the year**: date, amount (INR)
  A dividend also triggers excess distribution — flag prominently.
- If NO transactions during the year: note "no distributions — default PFIC, no excess distribution"

Do NOT list the full transaction history. Extract only the year-end snapshot and any redemptions/dividends.

### AIS / TIS (India Annual Information Statement)

The AIS is the India IT Department's view of your income. Extract category totals:
- Section 1: Salary (should be 0 for non-India-salary filers)
- Section 2: Interest from savings accounts — total (INR)
- Section 3: Interest from deposits (FDs) — total (INR)
- Section 4: Dividend from equities/MFs — total (INR)
- Section 5: Securities transactions (redemptions, sales) — total proceeds (INR)
- Section 7: Rental income — total (INR) — if non-zero, this is the rental income source
- Section 9: Foreign remittances received — note if present
- TDS deducted by category: interest TDS, dividend TDS, rental TDS

Cross-check AIS interest totals against interest certificates.
If AIS shows rental income > 0 and no rental statement was found, flag this.

### Advance Tax Payments / Challans (India)

For each challan:
- Date paid
- Amount (INR) + USD converted
- Assessment year it applies to
- BSR code (for verification)

### Prior Year US Return

Read the prior year return only for carryforward items:
- Capital loss carryforward (Schedule D)
- Foreign tax credit carryforward (Form 1116 Part III)
- Any PFIC elections previously made (QEF or MTM) — if yes, note which funds
- AMT credit carryforward (Form 8801) if applicable
- Prior FBAR filing confirmation

Do NOT extract income figures from the prior return (those are prior year facts, not current).

---

## Step 3 — Cross-Document Checks

After extracting all sections, run these checks. Add each issue to the Issues & Flags section.

| Check | Rule | Action |
|---|---|---|
| Excess SS | Sum all W-2 Box 4 values. If total > $10,918.20 (6.2% × $176,100 for 2025), excess is a refundable credit | 🔴 Flag with amount |
| Mortgage cap | Sum all 1098 Box 2 outstanding principal. If > $750,000, interest deduction is proportionally limited. **Compute the ratio and deductible amount.** | 🔴 Flag with ratio and deductible amount |
| Form 2441 earned income | If spouse has no W-2 income and no 1099-NEC / self-employment, childcare credit = $0 unless exception | 🔴 Flag |
| DCFSA exclusion (IRC §129) | If spouse earned income = $0 and no student/disability exception, DCFSA exclusion = $0. The full W-2 Box 10 amount must be **added back to taxable income** (Box 1 wages are understated). This is separate from the Form 2441 credit test. | 🔴 Flag with amount to add back |
| Form 2210 quarterly timing | Extract trade dates from 1099-B (or note which quarter bulk of capital gains occurred). If estimated payments were only made in Q4 but income was earned throughout, penalty applies. Note whether annualized income method (Schedule AI) could reduce penalty. | ⚠️ Flag with quarterly attribution |
| HSA over-contribution | Compare 5498-SA Box 2 vs limit ($8,300 family / $4,150 individual for 2025) + employer Box 12W | ⚠️ Flag if close or over |
| FBAR threshold | Sum all foreign account max balances (USD). If aggregate > $10,000 at any point, FBAR is required | 🔴 Flag if required |
| Form 8938 threshold | Sum all foreign financial asset year-end values (USD). MFJ US resident: required if > $100,000 year-end or > $150,000 any point | ⚠️ Flag if close or required |
| PFIC de minimis | Sum all PFIC year-end values (USD). If aggregate > $50,000 OR any single fund > $25,000, Form 8621 per fund is required | ⚠️ Flag with total |
| PFIC redemptions | Any redemption or dividend from a PFIC fund → excess distribution calculation required in year of distribution | 🔴 Flag per fund |
| NRE interest | NRE interest is NOT subject to Indian TDS but IS taxable on the US return. No FTC available for NRE interest. | ⚠️ Note per account |
| AIS reconciliation | Compare AIS interest total vs sum of interest certificates. If gap > ₹1,000, flag discrepancy | ⚠️ Flag with gap |
| AIS rental income | If AIS Section 7 shows rental income and no rental statement found → flag missing document | 🔴 Flag |
| PFIC prior elections | If prior year return shows no QEF/MTM elections and PFIC funds are held → default excess distribution method applies | ⚠️ Note |
| Missing documents | For every document type expected from CLAUDE.md that was not found, flag it | ⚠️ or 🔴 depending on importance |
| Child Tax Credit | Check for qualifying children under 17. CTC = $2,000 per child (phases out at $400K MFJ AGI). | ℹ️ Note amount |

Severity guide:
- 🔴 = Must resolve before filing. Material impact or compliance requirement.
- ⚠️ = Needs attention or verification. May be fine but requires confirmation.
- ℹ️ = Informational. No action required but good to know.

---

## Step 4 — Write the Knowledge Graph

Write `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` using this exact schema.
Use `?` for values not yet extracted. Use confidence markers inline:
  ✓ = directly read from document
  ~ = calculated/converted (show formula or rate used)
  ⚠️ = flagged for attention
  ✗ = expected but not found

Do NOT copy document content into the graph. Facts and figures only.
Every row in every table must have a source citation (relative path from SOURCE_FOLDER).
If a value could not be extracted (unreadable PDF, etc.), write `[unreadable]` and note the file.

---

### Knowledge Graph Schema

```markdown
# TaxBro Knowledge Graph — Tax Year {YEAR}
<!-- Source: {SOURCE_FOLDER} -->
<!-- Extracted: {DATE} | Mode: full / update -->
<!-- Documents: {N} read of {M} total | Status: complete / partial -->
<!-- IRS Pub 54 rate used: 1 USD = {rate} INR (2025 annual avg) -->

---

## ⚠️ Issues & Flags
> Address these before filing. Sorted by severity.

| # | Severity | Area | Issue | Detail |
|---|---|---|---|---|
| 1 | 🔴 | ... | ... | ... |

---

## Identity
| Field | Value | Source |
|---|---|---|
| Tax year | | |
| Filing status | | |
| Primary filer | | |
| Spouse | | |
| Dependents | | |
| Residency | US resident full year | |

---

## Income

### W-2 Wages
| Employer | Box 1 Wages | Box 2 Fed Wh | Box 3 SS Wages | Box 4 SS Tax | Box 5 Med Wages | Box 6 Med Tax | Box 10 DCFSA | Box 12W HSA | Box 12D 401k | State | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| | | | | | | | | | | | |
| **Total** | | | | | | | | | | | |

_W-2 Cross-Check Results: [PASS/FAIL per employer — show Box 4 ÷ Box 3 rate]_

### Brokerage Income (1099 Consolidated)
| Institution | Ord Div | Qual Div | Interest | Fed Wh | Source |
|---|---|---|---|---|---|
| | | | | | |

#### Capital Gains Breakdown (🔴 MANDATORY — must split ST/LT)
| Category | Proceeds | Cost Basis | Wash Sale | Net G/L |
|---|---|---|---|---|
| Short-term Box A (basis reported) | | | | |
| Short-term Box B (basis NOT reported — RSU) | | | | |
| Long-term Box D (basis reported) | | | | |
| Long-term Box E (basis NOT reported — RSU) | | | | |
| **Total Short-Term** | | | | |
| **Total Long-Term** | | | | |
| **Adjusted Net Capital Gains** | | | | |

_Quarterly attribution: Q1 $__ | Q2 $__ | Q3 $__ | Q4 $___

### Interest Income (US)
| Payer | Amount | Fed Wh | Source |
|---|---|---|---|
| | | | |

### HSA Summary
| Item | Amount | Source |
|---|---|---|
| Employer contributions (W-2 Box 12W, sum) | | |
| Employee + employer total (5498-SA Box 2) | | |
| Distributions (1099-SA Box 1) | | |
| Distribution type (code) | | |
| 2025 family limit | $8,300 | IRS |
| Over-contribution | | |

---

## Foreign Income

### Indian Bank Interest (Schedule B + Form 1116 Passive Basket)
| Institution | Acct | Type | Owner | Interest (INR) | TDS (INR) | Interest (USD) ~ | TDS (USD) ~ | NRE? | Source |
|---|---|---|---|---|---|---|---|---|---|
| | | | | | | | | | |
| **Total** | | | | | | | | | |

_NRE note: NRE interest = no Indian TDS, but fully taxable in US. No FTC credit available._
_List ALL accounts with interest — including SBI, HDFC, and other banks (not just ICICI)._

### AIS Cross-Check
| AIS Category | AIS Amount (INR) | Certificates Total (INR) | Gap | Status |
|---|---|---|---|---|
| Savings interest | | | | |
| FD/deposit interest | | | | |
| Mutual fund dividends | | | | |
| Rental income | | | | |
| Securities transactions | | | | |

### Rental Income — India (Schedule E)
| Item | Value | Source |
|---|---|---|
| Property description | | CLAUDE.md |
| Ownership % (Karthik / Vinaya) | | CLAUDE.md |
| Gross rent received (INR) | | |
| Gross rent (USD) ~ | | |
| TDS deducted by tenant (INR) | | |
| AIS Section 7 rental entry | | AIS file |
| Depreciation | | |
| PAL carryforward | | Prior return |
| **Net rental income** | | |

---

## Deductions

### Mortgage Interest (Form 1098)
| Lender | Box 1 Interest | Box 2 Principal (Jan 1) | Box 3 Origination | Box 10 Prop Tax | Source |
|---|---|---|---|---|---|
| | | | | | |
| **Total** | | | | | |

_Combined outstanding principal: $?_
_Deductible fraction: $750,000 / $? = ?%_
_**Deductible mortgage interest: ~$?**_
_Non-deductible interest: ~$?_

### Itemized Deductions (Schedule A)
| Item | Amount | Notes |
|---|---|---|
| Deductible mortgage interest | | From cap calculation above |
| SALT (property tax, capped) | $10,000 | Actual $? but MFJ cap = $10,000 |
| Charitable | | If applicable |
| **Total Itemized** | | |
| Standard deduction (MFJ) | $30,000 | 2025 |
| **Better of** | | Itemize / Standard |

### Childcare (Form 2441)
| Provider | EIN | Amount Paid | Period | Children | Source |
|---|---|---|---|---|---|
| | | | | | |
| **Total qualifying** | | | | | |

| Item | Value |
|---|---|
| DCFSA pre-tax (W-2 Box 10 total) | |
| Net qualifying expenses (total - DCFSA) | |
| Max qualifying (1 child: $3k / 2+: $6k) | |
| Vinaya earned income | $0 — no W-2 |
| Disability/student exception | ✗ / ✓ (check CLAUDE.md) |
| Form 2441 credit | **$0** — earned income test fails |
| **DCFSA exclusion valid?** | **🔴 NO if spouse $0 earned income** — IRC §129(b)(2) |
| DCFSA add-back to income | $? — amount to add to Line 1 wages |

---

## Foreign Accounts (FBAR / Form 8938)

### Account Summary
| Institution | Type | Owner | Acct (last 4) | Max Bal (local) | Max Bal (USD) ~ | Dec 31 Bal (local) | Dec 31 Bal (USD) ~ | Source |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

### Threshold Analysis
| Test | Threshold | Result | Status |
|---|---|---|---|
| FBAR — aggregate max > $10,000 | $10,000 | $? | Required / Not required |
| Form 8938 — year-end (MFJ US) | $100,000 | $? | Required / Not required |
| Form 8938 — any point (MFJ US) | $150,000 | $? | Required / Not required |

---

## Indian Mutual Funds (PFIC — Form 8621)

| Fund Name | AMC | Owner | Folio | Dec 31 Units | Dec 31 NAV | Dec 31 Val (INR) | Dec 31 Val (USD) ~ | Redemptions? | Dividends? | Prior Election | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| | | | | | | | | | | None | |

### PFIC Threshold Analysis
| Item | Value |
|---|---|
| Total aggregate year-end value (USD) | |
| De minimis threshold (aggregate) | $50,000 |
| De minimis threshold (individual) | $25,000 |
| Form 8621 required? | |
| Recommended approach | Default excess distribution (no elections made) |

---

## Credits

### Foreign Tax Credit (Form 1116)
| Basket | Income Type | Gross Foreign Income (USD) | Foreign Tax Paid (USD) | Source |
|---|---|---|---|---|
| Passive | Indian bank interest (NRO only) | | | Interest certs |
| Passive | Indian FD interest (NRO only) | | | Interest certs |
| General | Indian rental TDS (if applicable) | | | Challan/AIS |

_Note: NRE interest has NO FTC available — zero TDS in India._
_FTC cannot exceed (foreign income / total income) × US tax. Excess carries forward 10 years._

---

## Federal Tax Payments (Withholding)
| Source | Amount | Source Doc |
|---|---|---|
| W-2 withholding (total across all W-2s) | | |
| 1099 withholding (total across all 1099s) | | |
| Estimated tax payments (1040-ES) | | |
| **Total federal payments** | | |

---

## Prior Year References
| Item | Value | Source |
|---|---|---|
| Capital loss carryforward | | Prior return |
| FTC carryforward (passive) | | Prior return |
| FTC carryforward (general) | | Prior return |
| AMT credit carryforward | | Prior return |
| PFIC elections (QEF/MTM) | None | Prior return |
| FBAR filed for prior year | | Prior return |
| Prior year total tax (for safe harbor) | | Prior return |

---

## Source Document Registry
<!-- Every document found: was it read and what was extracted? -->

| File | Category | Triage | Extracted | Notes |
|---|---|---|---|---|
| | | FULL-PARSE / HEADER-ONLY / SUMMARY-ONLY / REFERENCE | ✓ / ✗ / [unreadable] | |

---

## Computed Totals (Single Source of Truth)
<!-- 🔴🔴🔴 THIS SECTION IS MANDATORY. DO NOT SKIP IT. 🔴🔴🔴
     All downstream skills (/taxbro-worksheets, /taxbro-visualize, /taxbro-checklist)
     MUST read these totals instead of recomputing from raw sections. This prevents agent drift.
     If this section is missing, the knowledge graph extraction is INCOMPLETE. -->

### Income Aggregation
| Line | Description | Amount | Formula / Source |
|---|---|---|---|
| Wages (1040 Line 1a) | W-2 Box 1 total | $ | Sum of W-2 table |
| Taxable Interest (1040 Line 2b) | US + foreign interest | $ | US 1099-INT total + all foreign interest (USD) |
| Ordinary Dividends (1040 Line 3b) | Non-qualified | $ | Total ordinary − qualified |
| Qualified Dividends (1040 Line 3a) | Preferential rate | $ | From 1099 |
| **Short-Term Capital Gain** | Taxed as ordinary income | **$** | Box A + Box B (adjusted) |
| **Long-Term Capital Gain** | Taxed at preferential rates | **$** | Box D + Box E (adjusted) |
| Net Capital Gain (1040 Line 7) | ST + LT combined | $ | Sum of above two lines |
| Rental Income (Sch 1 Line 5) | Net after depreciation + PAL | $ | From rental section |
| **Total Income (1040 Line 9)** | | **$** | Sum of all above |

### Key Adjustments
| Item | Amount | Notes |
|---|---|---|
| HSA deduction (8889 Line 13) | $ | 5498-SA total − employer contributions (Box 12W) |
| **Adjusted Gross Income (Line 11)** | **$** | Total Income − adjustments |

### Tax Computation Reference
| Item | Amount | Notes |
|---|---|---|
| Itemized — Mortgage interest (deductible portion) | $ | Total interest × ($750K ÷ outstanding principal); show ratio |
| Itemized — SALT (capped) | $10,000 | Property tax $X but capped at $10K |
| Itemized — Charitable | $ | If applicable |
| **Estimated Itemized Total** | **$** | |
| Standard Deduction (MFJ 2025) | $30,000 | Or itemized if larger |
| **Better of Standard/Itemized** | **$** | |
| **Taxable Income (Line 15)** | **$** | AGI − deduction |

_Taxable income breakdown (CRITICAL for correct tax computation):_
- _Ordinary taxable: $__ (wages + ST gains + interest + non-QD divs + rental − deductions)_
- _LTCG + QD at preferential rates: $___

### Tax & Credits Estimate
| Item | Amount | Notes |
|---|---|---|
| Ordinary income tax (from brackets) | $ | Use 2025 MFJ brackets: 10/12/22/24/32/35/37%. Apply to ORDINARY taxable income ONLY. |
| **LTCG + QD tax** | $ | Use preferential rates: 0/15/20% based on income stacking. At high incomes, typically 20%. |
| Additional Medicare Tax (0.9%) | $ | 0.9% × (wages − $250K MFJ threshold) |
| Net Investment Income Tax (3.8%) | $ | 3.8% × min(NII, MAGI − $250K); NII = cap gains + dividends + interest + rental |
| **Estimated Total Tax (before credits)** | **$** | Sum of all above |
| Excess SS credit (Schedule 3 Line 11) | −$ | Refundable |
| Foreign Tax Credit (Form 1116) | −$ | |
| Child Tax Credit | −$ | $2,000 per qualifying child under 17 |
| **Estimated Total Tax (after credits)** | **$** | |

### Payments & Credits
| Item | Amount | Source |
|---|---|---|
| W-2 withholding (total) | $ | |
| 1099 withholding (total) | $ | |
| Estimated tax payments (1040-ES) | $ | |
| **Total Payments** | **$** | |

### Bottom Line
| Item | Amount |
|---|---|
| Estimated Total Tax | $ |
| Total Payments | $ |
| **Estimated Refund / (Owe)** | **$** |
| Form 2210 penalty estimate | $ |
| **Net Refund / (Owe) after penalty** | **$** |

_⚠️ These are ESTIMATES using simplified bracket math. Actual return will differ due to:_
_qualified dividend/LTCG preferential rates (Schedule D worksheet), AMT calculation, Form 1116 limitation._
_Purpose: provide a directional "are we in refund or owe territory" answer and catch gross errors._
```

---

## Step 5 — Self-Validation (MANDATORY)

Before writing the final knowledge graph, run these sanity checks on your own output.
**If any check fails, fix the data before writing.**

| Check | Formula | Expected |
|---|---|---|
| Box 4 total × rate | Sum of all Box 4 ÷ (N_employers × $176,100) | ≈ 6.2% per employer |
| Federal withholding total | Sum of all W-2 Box 2 + 1099 Box 4 | Must equal Federal Tax Payments total |
| Capital gains | ST + LT | Must equal Net Capital Gain |
| Taxable income | AGI − deduction | Must equal stated taxable income |
| Total tax > total payments | Implies "owe" | Bottom line must say "owe" |
| Total tax < total payments | Implies "refund" | Bottom line must say "refund" |
| Computed Totals present | Section exists in output | **FAIL if missing** |

---

## Step 6 — Update Supporting Files

After writing the knowledge graph:

1. **Append to `{SOURCE_FOLDER}/TAXBRO/session-notes.md`** (do NOT overwrite):
   ```
   ## Extraction — {DATE}
   - Documents read: N of M (FULL-PARSE: X, HEADER-ONLY: Y, SUMMARY-ONLY: Z, REFERENCE: W)
   - Sections complete: [list]
   - Issues found: N (🔴 X critical, ⚠️ Y attention)
   - IRS Pub 54 rate used: {rate}
   - W-2 cross-checks: PASS/FAIL per employer
   - Self-validation: PASS/FAIL
   ```
   Structural metadata only — no financial figures in this log.

2. **Tell the user**:
   "Knowledge graph built. N documents read. X critical issues, Y items needing attention.
   Bottom line: estimated [refund $X / owe $X].
   Run /taxbro-worksheets to generate preparer-ready forms."

3. **Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md`** recording: agent-id, skill-name,
   status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

---

## Extraction Rules (Always Apply)

- **Extract facts, not content.** Never copy paragraphs, transaction lists, or statement tables into the graph.
- **Link, don't embed.** Every value gets a `source:` citation (relative filename). The user can open the file for detail.
- **Totals only for 1099-B.** Individual trade rows belong in Form 8949; the graph only needs net gain/loss by category (ST/LT) and wash sale totals.
- **Balances only for bank statements.** Max balance and year-end balance. Not transactions.
- **Holdings snapshot for CAMS CAS.** Year-end units/NAV/value and any distributions. Not full transaction history.
- **AIS category totals.** Not individual entries.
- **No PII.** No full account numbers, SSNs, PANs, or names beyond what's structurally needed.
- **No deletions.** Never run rm, rmdir, or any file deletion command. Only write to TAXBRO/.
- **Skip .snapshots/.** Never read from or write to TAXBRO/.snapshots/.
- **Mark unreadable files.** If a PDF is encrypted, corrupted, or unparseable, note it in the Source Registry as [unreadable] and flag it in Issues.
- **ST/LT split is NEVER optional.** Capital gains must always be broken into short-term and long-term. This affects tax computation by thousands of dollars.
- **W-2 cross-checks are NEVER optional.** Every W-2 must pass arithmetic validation before its numbers enter the knowledge graph.
- **Computed Totals is NEVER optional.** The extraction is incomplete without it.
