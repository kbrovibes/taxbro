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

## Step 2 — Extract by Category

Read each document category below. Extract ONLY the specified facts — do not copy full content.
For every fact, record the source filename as a relative path from SOURCE_FOLDER.

### W-2 (Employment Income)

For each W-2 PDF found:
- Employer name and EIN
- Box 1: Wages
- Box 2: Federal income tax withheld
- Box 4: Social Security tax withheld
- Box 6: Medicare tax withheld
- Box 10: Dependent care FSA (DCFSA) — if non-zero
- Box 12 codes: W (employer HSA), D (401k), DD (health insurance premiums), AA (Roth 401k)
- Box 16/17: State wages and state income tax withheld (per state)
- Box 14: Other (RSU, union dues, etc.) — note what's there

Do NOT extract employee SSN, address, or other PII.

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
  CRITICAL: RSU sales in Box B or E often show $0 basis. Scan for "Fidelity Supplemental" or "2025-TAX-RETURN-SUMMARY.md" in the source folder to find the actual adjusted cost basis (FMV at vest). Always flag $0 basis for preparer adjustment if supplemental basis isn't found.
  NOTE: Do NOT list individual trades. Extract totals only. Note if Form 8949 required.
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
- Box 3: Refund of overpaid interest (if any)
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
If 12 monthly statements exist (like Scotiabank), read all 12 to find the peak balance; report only the peak.

### Indian Interest Certificates / Form 16A / TDS Certificates

For each certificate:
- Institution and account type
- Account owner
- Total interest credited (INR)
- TDS deducted (INR) — this is the foreign tax paid for Form 1116
- Certificate period
- Whether account is NRE (no TDS in India) or NRO (TDS applies)

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
| Mortgage cap | Sum all 1098 Box 2 outstanding principal. If > $750,000, interest deduction is proportionally limited | ⚠️ Flag with total |
| Form 2441 earned income | If spouse has no W-2 income and no 1099-NEC / self-employment, childcare credit = $0 unless exception | 🔴 Flag |
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
| Residency | US resident full year | |
| Tax year ref in docs | | |

---

## Income

### W-2 Wages
| Employer | Box 1 Wages | Box 2 Fed Wh | Box 4 SS | Box 6 Med | Box 10 DCFSA | Box 12W HSA-emp | State Wh | Source |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |
| **Total** | | | | | | | | |

### Brokerage Income (1099 Consolidated)
| Institution | Ord Div | Qual Div | ST CG | LT CG | Interest | Fed Wh | 1099-B Net G/L | Wash Sale Disallowed | Source |
|---|---|---|---|---|---|---|---|---|---|
| | | | | | | | | | |

_Note: 1099-B detail → Form 8949 required: yes / no_

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
| Institution | Type | Owner | Interest (INR) | TDS Withheld (INR) | Interest (USD) ~ | TDS (USD) ~ | NRE? | Source |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |
| **Total** | | | | | | | | |

_NRE note: NRE interest = no Indian TDS, but fully taxable in US. No FTC credit available._

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
| Gross rent received (INR) | ✗ | Not found |
| Gross rent (USD) ~ | | |
| TDS deducted by tenant (INR) | ✗ | Not found |
| AIS Section 7 rental entry | | AIS file |
| Depreciation basis established | | |

---

## Deductions

### Mortgage Interest (Form 1098)
| Lender | Box 1 Interest | Box 2 Principal (Jan 1) | Box 10 Prop Tax | Source |
|---|---|---|---|---|
| | | | | |
| **Total** | | | | |

_Combined outstanding principal: $? — limit check: {'✓ under $750k' or '⚠️ exceeds $750k — deduction limited'}_

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
| Form 2441 credit | **Likely $0** — earned income test fails |

---

## Foreign Accounts (FBAR / Form 8938)

### Account Summary
| Institution | Type | Owner | Acct (last 4) | Max Bal (local) | Max Bal (USD) ~ | Dec 31 Bal (local) | Dec 31 Bal (USD) ~ | Source |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |
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
| Passive | Indian bank interest (NRO) | | | Interest certs |
| Passive | Indian FD interest (NRO) | | | Interest certs |
| General | India advance tax payments | | | Challans |
| General | Indian rental TDS (if applicable) | | | Challan/AIS |

_Note: FTC cannot exceed (foreign income / total income) × US tax. Excess carries forward 10 years._

### India Advance Tax Payments
| Date | Amount (INR) | Amount (USD) ~ | Assessment Year | Source |
|---|---|---|---|---|
| | | | | |
| **Total** | | | | |

---

## Federal Tax Payments (Withholding)
| Source | Amount | Source Doc |
|---|---|---|
| W-2 withholding (total across all W-2s) | | |
| 1099 withholding (total across all 1099s) | | |
| **Total federal withholding** | | |

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

---

## Source Document Registry
<!-- Every document found: was it read and what was extracted? -->

| File | Category | Extracted | Notes |
|---|---|---|---|
| | | ✓ / ✗ / [unreadable] | |

---

## Computed Totals (Single Source of Truth)
<!-- CRITICAL: All downstream skills (/taxbro-generate-worksheets, /taxbro-visualize, /taxbro-checklist)
     MUST read these totals instead of recomputing from raw sections. This prevents agent drift. -->

### Income Aggregation
| Line | Description | Amount | Formula / Source |
|---|---|---|---|
| Wages (1040 Line 1a) | W-2 Box 1 total | $ | Sum of W-2 table |
| Taxable Interest (1040 Line 2b) | US + foreign interest | $ | US 1099-INT total + NRE interest (USD) + NRO interest (USD) |
| Ordinary Dividends (1040 Line 3b) | | $ | 1099 consolidated Ord Div total |
| Qualified Dividends (1040 Line 3a) | | $ | 1099 consolidated Qual Div total |
| Capital Gain (1040 Line 7) | Net, after basis adjustments | $ | **Use adjusted 1099-B gain, NOT face value.** If RSU basis adjusted: state adjusted amount explicitly |
| Rental Income (Sch 1 Line 5) | Net after depreciation + PAL | $ | Gross rent (USD) − depreciation − expenses + prior PAL applied |
| **Total Income (1040 Line 9)** | | **$** | Sum of all above |

### Key Adjustments
| Item | Amount | Notes |
|---|---|---|
| HSA deduction (8889 Line 13) | $ | 5498-SA total − employer contributions (Box 12W) |
| **Adjusted Gross Income (Line 11)** | **$** | Total Income − adjustments |

### Tax Computation Reference
| Item | Amount | Notes |
|---|---|---|
| Standard Deduction (MFJ 2025) | $30,000 | Or itemized if larger |
| Itemized — Mortgage interest (deductible portion) | $ | Total interest × ($750K ÷ outstanding principal); show ratio |
| Itemized — SALT (capped) | $10,000 | Property tax $X but capped at $10K |
| Itemized — Charitable | $ | If applicable |
| **Estimated Itemized Total** | **$** | |
| **Better of Standard/Itemized** | **$** | |
| **Taxable Income (Line 15)** | **$** | AGI − deduction |

### Tax & Credits Estimate
| Item | Amount | Notes |
|---|---|---|
| Regular tax (from brackets) | $ | Use 2025 MFJ brackets: 10/12/22/24/32/35/37% |
| Additional Medicare Tax (0.9%) | $ | 0.9% × (wages − $250K MFJ threshold) |
| Net Investment Income Tax (3.8%) | $ | 3.8% × min(NII, MAGI − $250K); NII = cap gains + dividends + interest + rental |
| **Estimated Total Tax** | **$** | Regular + AMT + Additional Medicare + NIIT |

### Payments & Credits
| Item | Amount | Source |
|---|---|---|
| W-2 withholding (total) | $ | |
| 1099 withholding (total) | $ | |
| Estimated tax payments (1040-ES) | $ | |
| Excess SS credit (Schedule 3) | $ | |
| Foreign tax credit (Form 1116) | $ | |
| Child tax credit | $ | $2,000 per qualifying child under 17 |
| **Total Payments + Credits** | **$** | |

### Bottom Line
| Item | Amount |
|---|---|
| Estimated Total Tax | $ |
| Total Payments + Credits | $ |
| **Estimated Refund / (Owe)** | **$** |
| Form 2210 penalty estimate | $ |
| **Net Refund / (Owe) after penalty** | **$** |

_⚠️ These are ESTIMATES using simplified bracket math. Actual return will differ due to:_
_qualified dividend/LTCG preferential rates, AMT calculation, Form 1116 limitation, Schedule D worksheet._
_Purpose: provide a directional "are we in refund or owe territory" answer and catch gross errors._
```

---

## Step 5 — Update Supporting Files

After writing the knowledge graph:

1. **Append to `{SOURCE_FOLDER}/TAXBRO/session-notes.md`** (do NOT overwrite):
   ```
   ## Extraction — {DATE}
   - Documents read: N of M
   - Sections complete: [list]
   - Issues found: N (🔴 X critical, ⚠️ Y attention)
   - IRS Pub 54 rate used: {rate}
   ```
   Structural metadata only — no financial figures in this log.

2. **Tell the user**:
   "Knowledge graph built. N documents read. X critical issues, Y items needing attention.
   Run /taxbro-checklist to review the full picture, or ask me to walk through any specific topic."

3. **Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md`** recording: agent-id (e.g., `gemini-2.0-flash`), skill-name, status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

---

## Extraction Rules (Always Apply)

- **Extract facts, not content.** Never copy paragraphs, transaction lists, or statement tables into the graph.
- **Link, don't embed.** Every value gets a `source:` citation (relative filename). The user can open the file for detail.
- **Totals only for 1099-B.** Individual trade rows belong in Form 8949; the graph only needs net gain/loss and wash sale totals.
- **Balances only for bank statements.** Max balance and year-end balance. Not transactions.
- **Holdings snapshot for CAMS CAS.** Year-end units/NAV/value and any distributions. Not full transaction history.
- **AIS category totals.** Not individual entries.
- **No PII.** No full account numbers, SSNs, PANs, or names beyond what's structurally needed.
- **No deletions.** Never run rm, rmdir, or any file deletion command. Only write to TAXBRO/.
- **Skip .snapshots/.** Never read from or write to TAXBRO/.snapshots/.
- **Mark unreadable files.** If a PDF is encrypted, corrupted, or unparseable, note it in the Source Registry as [unreadable] and flag it in Issues.
