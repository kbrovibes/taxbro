# TaxBro

A Claude Code project for analyzing US tax returns using source documents.
Point it at a folder of tax PDFs and statements — TaxBro reads, extracts, and summarizes everything you need for your preparer or for self-filing.

---

## How it works

TaxBro is a set of **Claude Code slash commands** (skills) that each do one focused job: analyze W-2s, check FBAR thresholds, identify PFIC exposure, consolidate foreign tax credits, and so on. Skills read from your documents and write their findings to a `TAXBRO/` folder inside your documents directory. Nothing with personal data ever touches the TaxBro repo itself.

### Personalization via CLAUDE.md

TaxBro is generic by design, but gets smarter when you add a `CLAUDE.md` to your tax documents folder. This file describes your specific situation — filer status, which accounts exist, which forms are expected, known edge cases. Every skill reads it first and adapts its analysis accordingly. **Your CLAUDE.md takes priority over TaxBro's defaults.**

If no CLAUDE.md is present, TaxBro falls back to a standard US filer checklist.

---

## Install

### Prerequisites

- [Claude Code](https://claude.ai/code) (the CLI)
- A folder of tax documents (PDFs, statements, exports — any naming convention)

### Setup

```bash
git clone https://github.com/kbrovibes/taxbro ~/claude/taxbro
cd ~/claude/taxbro
claude
```

That's it. Claude Code auto-discovers the `.claude/commands/` directory and registers all skills.

---

## Quick start (first-time walkthrough)

This walkthrough assumes a relatively simple US filer: one or two W-2s, maybe some 1099-INTs, no foreign accounts. Takes about 5 minutes.

**Step 1 — Organize your documents**

Put all your tax documents in one folder. Sub-folders are fine. Example structure:

```
~/Documents/taxes-2025/
├── W2-employer1.pdf
├── W2-employer2.pdf
├── 1099-INT-bank.pdf
├── 1099-DIV-brokerage.pdf
├── 1099-B-brokerage.pdf
├── 1098-mortgage.pdf
└── childcare-receipt.pdf
```

**Step 2 — (Optional but recommended) Add a CLAUDE.md**

Create `~/Documents/taxes-2025/CLAUDE.md` describing your situation:

```markdown
# Tax Return 2025 — Context

## Filers
- Primary: [Your name] — W-2 employee, single employer
- Filing status: MFJ

## Key situations
- [x] Two W-2s (job change mid-year)
- [x] Mortgage interest (single home)
- [x] Childcare (1 child, daycare)
- [ ] HSA
- [ ] Foreign accounts
- [ ] Foreign rental property
- [ ] PFIC / foreign mutual funds
```

**Step 3 — Initialize**

```
/taxbro-init ~/Documents/taxes-2025/
```

TaxBro reads your CLAUDE.md, lists all discovered documents grouped by type, flags anything that looks missing, and creates the `TAXBRO/` output folder.

**Step 4 — Run the checklist**

```
/tax-checklist
```

Shows a full status table: which issues apply to you, whether source documents are present, whether analysis has been run, and any risk flags. Start here every session — it tells you what still needs attention.

**Step 5 — Run the relevant skills**

For a simple filer:

```
/check-w2s          # Analyzes all W-2s, checks for excess SS withholding
/check-childcare    # Analyzes childcare expenses, Form 2441 eligibility
```

**Step 6 — Generate worksheets**

```
/generate-worksheets
```

Produces a single `worksheets.md` with pre-filled line items for every applicable IRS form — ready to hand to a preparer or enter into tax software.

**Step 7 — Validate a draft return (optional)**

Once your preparer sends a draft PDF:

```
/validate-return ~/Documents/taxes-2025/draft-return.pdf
```

Cross-checks every key line against your source documents and flags discrepancies.

---

## All commands

| Command | Purpose |
|---------|---------|
| `/taxbro` | Overview: all commands, session status, output file reference |
| `/taxbro-init [path]` | Load source folder, read CLAUDE.md, discover and categorize documents |
| `/tax-checklist` | Full status sweep: documents present, analysis done, risk flags |
| `/check-w2s` | W-2 analysis: wages, withholding, excess SS credit, DCFSA, HSA employer contribs |
| `/check-fbar` | Foreign accounts: max and year-end balances, FBAR (FinCEN 114) and Form 8938 thresholds |
| `/check-pfic` | Foreign mutual funds: PFIC determination, de minimis check, Form 8621 per fund |
| `/foreign-tax-credit` | Consolidate all foreign taxes paid by basket → Form 1116 |
| `/check-childcare` | Childcare expenses: provider EINs, DCFSA offset, Form 2441 eligibility + earned-income test |
| `/rental-income` | Foreign rental income: INR→USD conversion, TDS, Schedule E summary, depreciation |
| `/generate-worksheets` | Pre-filled worksheets for all applicable IRS forms (FBAR, 1116, 8621, Schedule E, 2441, 1040 summary) |
| `/validate-return [pdf]` | Cross-check a completed or draft return PDF against all source documents |

---

## Command details

### `/taxbro-init [path]`

Loads a source folder and prepares TaxBro for the session.

- Reads `{SOURCE_FOLDER}/CLAUDE.md` if present and summarizes the filer's situation
- Lists all discovered documents, grouped by type (W-2s, 1099s, 1098s, foreign statements, etc.)
- Flags document types that appear missing based on what CLAUDE.md describes
- Creates `{SOURCE_FOLDER}/TAXBRO/` if it doesn't exist
- Writes a structural session log to `TAXBRO/session-notes.md` (no financial data)
- Saves the folder path to `~/claude/taxbro/.current-session` for use by subsequent skills

After running this, all other skills can be invoked without a path argument.

---

### `/tax-checklist`

Runs a full status check across all applicable tax issues.

Checks for each issue:
- Are the required source documents present?
- Has TaxBro already run analysis on it?
- Are there any open risks or flags?

Standard checklist items (trimmed or extended based on your CLAUDE.md):

| Issue | Key question |
|-------|-------------|
| W-2(s) | All employers present; excess SS check done |
| 1099-B/DIV/INT | All brokerages present; wash sale noted |
| 1098 (mortgage) | All properties present; $750K acquisition debt limit checked |
| Childcare | Provider statements present; Form 2441 earned-income eligibility confirmed |
| HSA | 1099-SA + 5498-SA present; contribution limit checked |
| Foreign accounts | All statements present; FBAR and Form 8938 thresholds checked |
| PFIC / mutual funds | CAS/statements present; Form 8621 determination done |
| Foreign interest | Certificates present; Form 1116 passive basket ready |
| Rental income | Income documented; Schedule E amounts calculated |
| Foreign tax payments | Advance payments documented; Form 1116 general basket ready |
| Schedule B Part III | Foreign account disclosure ready |

Output: a status table with ✅ clear, ⚠️ needs attention, ❌ blocked by missing documents.

---

### `/check-w2s`

Analyzes all W-2 forms found in the source folder.

**Extracts per W-2:** employer name, Box 1 (wages), Box 2 (federal withholding), Box 4 (SS tax), Box 6 (Medicare), Box 10 (DCFSA), Box 12 codes (401k, HSA employer, health insurance).

**Calculations:**
- **Excess Social Security**: if you worked for multiple employers and total Box 4 across all W-2s exceeds the annual cap ($10,918.20 for 2025 at the $176,100 wage base), the excess is a refundable credit on Form 1040
- **DCFSA coordination**: Box 10 amounts are passed to `/check-childcare` for Form 2441 offset
- **HSA**: Box 12 Code W employer contributions are flagged for contribution limit checking

Output: `TAXBRO/w2-summary.md`

---

### `/check-fbar`

Determines FBAR (FinCEN 114) and Form 8938 (FATCA) filing requirements.

**For each foreign account:**
- Extracts highest balance during the year and December 31 balance
- Converts to USD using IRS Publication 54 annual average exchange rates
- Identifies account type: savings, NRE, NRO, Fixed Deposit, brokerage

**FBAR (FinCEN 114):** required if aggregate of all foreign account maximums exceeds $10,000 at any point during the year. Filed separately at bsaefiling.fincen.treas.gov (free, no preparer needed).

**Form 8938 (FATCA):** MFJ US resident threshold is $100,000 year-end or $150,000 at any point. Note: includes foreign mutual funds (PFICs), not just bank accounts.

**NRE vs NRO flag:** NRE interest is tax-free in India but fully taxable in the US — must appear on Schedule B. NRO interest is taxable in both countries; TDS withheld is creditable on Form 1116.

Output: `TAXBRO/fbar-summary.md`

---

### `/check-pfic`

Analyzes foreign mutual fund holdings for PFIC (Passive Foreign Investment Company) compliance.

Indian mutual funds are almost certainly PFICs. Every PFIC typically requires a Form 8621 filed with your return.

**Extracts per fund:** fund name, AMC, ISIN, units held, NAV and value on December 31, any redemptions or dividends during the year.

**Determinations:**
- **De minimis exception**: if total USD value of all PFICs is ≤ $50,000 at year-end and no prior QEF/MTM elections have been made, Form 8621 may not be required (always confirm with your preparer)
- **Election method**: tracks whether default (excess distribution), QEF, or MTM method applies
- **Redemptions**: flags any fund with withdrawals during the year — these trigger excess distribution calculations

Output: `TAXBRO/pfic-summary.md`

---

### `/foreign-tax-credit`

Consolidates all foreign taxes paid for Form 1116, organized by income basket.

**Passive basket** (typically): NRO interest TDS, NRO savings TDS, foreign dividend withholding, rental TDS (if passive activity)

**General basket** (typically): advance tax payments to India, Canadian withholding on wages (if any)

**Per tax item extracted:** country, income type, gross income (local + USD), tax paid (local + USD), Form 1116 basket, source document.

Also flags the FTC limitation: the credit cannot exceed `(foreign income / total income) × US tax`. For high-bracket US filers with significant India income, this limit is often binding — excess carries forward 10 years.

Output: `TAXBRO/ftc-summary.md`

---

### `/check-childcare`

Analyzes childcare expenses for Form 2441 (Child and Dependent Care Credit).

**Extracts per provider:** name, address, EIN (required on Form 2441), total amount paid, period of care.

**Key calculations:**
- Qualifying expense cap: $3,000 for 1 child, $6,000 for 2+ children
- DCFSA offset: W-2 Box 10 amounts are pre-tax dollars — they must be subtracted from qualifying expenses (no double-dipping)
- Credit rate: 20%–35% of net qualifying expenses depending on AGI; 20% for AGI above $43,000

**Earned income test (critical):** both spouses must have earned income to claim the credit. If your CLAUDE.md indicates the spouse does not work and no exception applies (full-time student, disability), TaxBro flags this prominently. Note: even if the credit is disallowed, the DCFSA pre-tax benefit in Box 10 remains valid.

Output: `TAXBRO/childcare-summary.md`

---

### `/rental-income`

Analyzes foreign rental property income for Schedule E.

Looks for income in: AIS/TIS statements (India), bank statement credits consistent with rent, rental agreements, Form 16A / TDS certificates from tenant.

**Per property:**
- Gross rent in local currency, converted to USD using IRS Pub 54 annual average rate
- TDS deducted (carried to `/foreign-tax-credit`)
- Filer's ownership share (if co-owned)
- Schedule E expense items: property taxes, repairs, management fees, mortgage interest, depreciation

**Depreciation:** foreign residential property depreciates over 30 years (not 27.5 like US property). Basis = purchase price × ownership % × USD rate at acquisition date. TaxBro flags if the depreciation basis has not been established — a common and costly oversight.

Output: `TAXBRO/rental-income.md`

---

### `/generate-worksheets`

Produces pre-filled line-item worksheets for every applicable IRS form, ready to hand to a preparer or enter into tax software directly.

Requires prior analysis runs. If any source file is missing, TaxBro tells you which skill to run first and generates worksheets for everything it does have.

**Worksheets generated:**
- **FinCEN 114 (FBAR)** — all required fields per account, filing instructions
- **Form 8938 (FATCA)** — Part I (bank/custodial accounts) and Part II (other foreign assets)
- **Form 1116 — Passive basket** — income lines, taxes paid, limitation fraction estimate
- **Form 1116 — General basket** — advance tax payments, active rental if applicable
- **Form 8621 (per PFIC fund)** — method election, distribution analysis, Section 1298(f) annual reporting
- **Schedule E (per rental property)** — all line items including depreciation
- **Form 2441** — provider table, DCFSA offset, credit calculation, earned-income flag
- **Form 1040 summary** — key line aggregations from all sources

Output: `TAXBRO/worksheets.md`

---

### `/validate-return [pdf]`

Cross-checks a completed or draft return PDF against all your source documents and TaxBro analysis outputs.

Pass a PDF path directly, or TaxBro will look for a draft return in your `TAXBRO/` folder.

**Checks every key line:**

| Area | What gets validated |
|------|-------------------|
| Form 1040 income | W-2 wages, interest, dividends, capital gains vs source docs |
| Schedule B | Each line traceable to a specific 1099-INT or interest certificate; Part III foreign accounts match FBAR list |
| Schedule E | Gross rents, depreciation, TDS vs rental-income.md |
| Form 1116 | Foreign income and taxes by basket vs ftc-summary.md |
| Form 2441 | Provider EINs, amounts, DCFSA offset vs childcare-summary.md |
| Form 8938 | All accounts listed, max values in range vs fbar-summary.md |
| Form 8621 | One form per PFIC fund, method consistency vs pfic-summary.md |
| Excess SS | Credit on Schedule 3 Line 11 vs w2-summary.md calculation |

Each check returns: ✅ PASS, ⚠️ FLAG (>$10 difference), ❌ MISMATCH (>$50 or directional error), or SKIP (form not present).

Output: `TAXBRO/validation-report.md`

---

## Output files

All output goes to `{SOURCE_FOLDER}/TAXBRO/`. Keep this folder private — it contains your financial data.

| File | Produced by | Contents |
|------|-------------|---------|
| `session-notes.md` | `/taxbro-init` | Initialization log, document counts, missing doc flags |
| `checklist.md` | `/tax-checklist` | Full status table with dates |
| `w2-summary.md` | `/check-w2s` | W-2 table, excess SS calc, DCFSA/HSA flags |
| `fbar-summary.md` | `/check-fbar` | Account balances, FBAR/8938 determinations |
| `pfic-summary.md` | `/check-pfic` | Fund table, de minimis determination, Form 8621 per fund |
| `ftc-summary.md` | `/foreign-tax-credit` | Foreign taxes by basket, FTC limitation note |
| `childcare-summary.md` | `/check-childcare` | Provider table, DCFSA offset, Form 2441 eligibility |
| `rental-income.md` | `/rental-income` | Monthly rent table, USD conversion, Schedule E summary |
| `worksheets.md` | `/generate-worksheets` | Pre-filled line items for all applicable forms |
| `validation-report.md` | `/validate-return` | Line-by-line cross-check results |

---

## Personalizing with CLAUDE.md

Add a `CLAUDE.md` to your tax documents folder to make TaxBro smarter. This file is read by every skill before running analysis. It takes priority over TaxBro's defaults.

What to include:

```markdown
# Tax Return [YEAR] — Context

## Filers
- Primary: [Name] — [employer / employment type]
- Spouse: [Name] — [working / not working / student]
- Filing status: MFJ / Single / MFS / HoH

## Key situations
- [ ] Multiple W-2s (job change)
- [ ] Mortgage (single / multiple properties)
- [ ] Childcare (N children, provider name)
- [ ] HSA
- [ ] Foreign accounts (list countries and institutions)
- [ ] Foreign rental property (location, ownership %)
- [ ] PFIC / Indian mutual funds (CAMS CAS on file)
- [ ] Foreign tax credit (India advance tax, NRO TDS)
- [ ] Canada / other country history

## Foreign accounts
[For each account]
- Institution: [name]
- Country: [country]
- Type: NRE / NRO / savings / FD / brokerage
- Expected statements: [yes/no]

## Notes
[Any edge cases, prior year elections, flags from last year's return]
```

The more detail here, the more targeted TaxBro's analysis will be — especially for missing document detection and checklist prioritization.

---

## Privacy and data safety

These rules are absolute:

- **This repo contains zero personal data** — only skill prompts and instructions
- **All outputs go to `{SOURCE_FOLDER}/TAXBRO/`** — your documents folder, which you control
- **`.current-session`** contains only a folder path — it is gitignored
- **Never commit your `TAXBRO/` folder** — it contains your financial data
- **Memory/learnings stored by TaxBro** contain only methodological notes (e.g., "NRE interest is US-taxable") — never amounts, names, or account details

---

## Supported forms

| Form | Handled by |
|------|-----------|
| Form 1040 (key lines) | `/generate-worksheets` |
| W-2 excess SS credit (Schedule 3) | `/check-w2s` |
| Schedule B (interest, dividends, Part III) | `/tax-checklist`, `/validate-return` |
| Schedule E (foreign rental) | `/rental-income` |
| Schedule D / Form 8949 | `/tax-checklist` (presence check), `/validate-return` |
| Form 1098 (mortgage interest) | `/tax-checklist` |
| Form 2441 (childcare credit) | `/check-childcare` |
| FinCEN 114 (FBAR) | `/check-fbar` |
| Form 8938 (FATCA) | `/check-fbar` |
| Form 8621 (PFIC) | `/check-pfic` |
| Form 1116 (foreign tax credit) | `/foreign-tax-credit` |
| Form 8889 (HSA) | `/tax-checklist` (presence check) |

---

## Extending TaxBro

Add a new skill by creating `.claude/commands/your-skill.md`:

```markdown
Get source folder from $ARGUMENTS or ~/claude/taxbro/.current-session.
Read {SOURCE_FOLDER}/CLAUDE.md for filer context (your CLAUDE.md takes priority).

[Your analysis instructions]

Write output to {SOURCE_FOLDER}/TAXBRO/your-output.md.
Never write PII or document contents to any file inside ~/claude/taxbro/.
```

---

## Disclaimer

TaxBro is an analysis aid, not tax advice. Always review outputs with a qualified tax preparer, especially for complex situations: PFIC/Form 8621, FBAR, foreign tax credit, dual-country filings, and excess distribution calculations. IRS rules change annually — verify thresholds and rates for your tax year.
