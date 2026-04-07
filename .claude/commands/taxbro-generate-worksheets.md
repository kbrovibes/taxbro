Generate pre-filled worksheets for all applicable IRS forms based on source documents.
Outputs structured, line-item-ready reference documents your preparer or tax software can enter directly.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Generating worksheets for tax year {ACTIVE_YEAR}. (Override: /taxbro-generate-worksheets 2024)"

## Check available analysis outputs

Read from OUTPUT_DIR (skip if missing, note as unavailable):
- fbar-summary.md → drives FinCEN 114 + Form 8938 worksheets
- ftc-summary.md → drives Form 1116 worksheets
- pfic-summary.md → drives Form 8621 worksheets
- rental-income.md → drives Schedule E worksheet
- w2-summary.md → drives Form 1040 wages + excess SS
- childcare-summary.md → drives Form 2441 worksheet

For any missing file: note "⚠️ [file] not found — run /taxbro-[skill] first for complete worksheet" and skip that section.

## Generate worksheets

Write each as a clearly labeled section in {OUTPUT_DIR}/worksheets.md:

---
### WORKSHEET: FinCEN 114 (FBAR)
Per account: institution name/address, account type, account number, max value (USD), account owner, country.
Note: file free at bsaefiling.fincen.treas.gov

### WORKSHEET: Form 8938 (FATCA)
Part I — Foreign deposit/custodial accounts: institution, account number, max value, year-end value, income (Y/N + type)
Part II — Other foreign financial assets (mutual funds): description, issuer, max/year-end value, income
Total asset value vs filing threshold

### WORKSHEET: Form 1116 — Passive Basket
Line 1a/1b: country(ies) and income description
Lines 2–6: taxes withheld per country/income type (itemized)
Line 9: net foreign income
Limitation fraction: foreign income ÷ total income (estimate)

### WORKSHEET: Form 1116 — General Basket (if applicable)
Same structure, separate from passive

### WORKSHEET: Form 8621 (one per PFIC fund with reportable activity)
Fund name, address, reference ID (ISIN), tax year
Method check box: default / QEF / MTM
Part I (default): distributions received, prior year earnings, excess distribution calc
If no distributions: "Section 1298(f) annual reporting only"

### WORKSHEET: Schedule E (per foreign rental property)
Property address, days rented / personal use
Line 3: rents received (USD)
Lines 5–19: advertising, insurance, mortgage interest, other interest, repairs, taxes, depreciation, other
Line 21: total expenses
Line 22: net income / (loss)

### WORKSHEET: Form 2441 (Childcare)
Part I: provider name, address, EIN, amount paid
Part II: qualifying expenses, earned income (filer + spouse), DCFSA offset (Box 10), credit %
Line 15: estimated credit amount. FLAG if spouse has no earned income.

### WORKSHEET: Form 1040 — Key Lines Summary
Line 1a: W-2 wages (total Box 1)
Line 2b: taxable interest (US + NRE/NRO)
Line 3b/3a: ordinary / qualified dividends
Line 7: capital gain or loss
Schedule 1 Line 8: rental net income (if positive)
Line 13: Foreign tax credit (from Form 1116)
Line 11: Excess SS credit (from w2-summary)
Line 19: Child tax credit
---

At the end of worksheets.md, add "What your preparer still needs":
- US tax before credits (Line 16) — needed for Form 1116 limitation
- Final AGI — needed for Form 2441 credit rate
- Capital gains detail (short/long) — from 1099-B / Schedule D
- State return items

Print summary: which worksheets generated, which skipped (missing analysis).

## Update CLAUDE.md
Update {OUTPUT_DIR}/CLAUDE.md — change the Worksheets row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/worksheets.md only.
