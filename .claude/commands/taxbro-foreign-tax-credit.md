Consolidate all foreign taxes paid for Form 1116 (Foreign Tax Credit).

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Consolidating foreign taxes for tax year {ACTIVE_YEAR}. (Override: /taxbro-foreign-tax-credit 2024)"

## Background
- Form 1116 required when foreign taxes are paid/accrued on foreign income
- Passive basket: NRO interest TDS, dividend withholding, rental TDS (if passive activity)
- General basket: advance tax payments to India, Canadian withholding on wages
- NRE interest: NOT taxable in India → no TDS → no Form 1116 credit for NRE interest

## Analysis

Find all foreign tax documents for ACTIVE_YEAR using file-index.md:
- Interest certificates (category: Interest certificate)
- AIS/TIS (category: AIS / TIS)
- Advance tax receipts (category: Tax payment)
- Rental TDS certificates (category: Interest certificate or Foreign bank)

Also read {OUTPUT_DIR}/rental-income.md if it exists — for rental TDS amounts.

For each foreign tax item, extract:
- Country
- Income type (interest / rental / dividend / other)
- Gross income (local + USD)
- Tax withheld or paid (local + USD)
- Form 1116 basket (passive / general)
- Source document

Convert to USD using IRS Pub 54 rates.

FTC limitation note:
- Credit limited to: (Foreign income / Total income) × US tax
- Excess carries forward 10 years
- Flag if India taxes likely exceed limitation (common for high-bracket US filers)

## Write output
Write to {OUTPUT_DIR}/ftc-summary.md:
- Table: Country | Income Type | Gross Income (USD) | Foreign Tax (USD) | Basket | Source Doc
- Subtotals by basket (passive, general)
- Form 1116 filing recommendation
- Carryforward notes if applicable

Update {OUTPUT_DIR}/CLAUDE.md — change the Foreign tax credit row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/ftc-summary.md only.
