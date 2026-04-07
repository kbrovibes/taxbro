Analyze all foreign bank account statements and determine FBAR / Form 8938 filing requirements.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Analyzing foreign accounts for tax year {ACTIVE_YEAR}. (Override: /taxbro-check-fbar 2024)"

## Analysis

Read {SOURCE_FOLDER}/CLAUDE.md for expected foreign accounts.
Find all foreign account statement files for ACTIVE_YEAR using file-index.md (category: Foreign bank, year: ACTIVE_YEAR).

For each account, extract:
- Institution name and country
- Account type (savings / NRE / NRO / Fixed Deposit / brokerage)
- Currency
- Highest balance during the calendar year (any single day)
- Balance on December 31
Convert to USD using IRS Pub 54 annual average exchange rates.

FBAR determination (FinCEN 114):
- Aggregate max balances (USD) across ALL foreign accounts
- If > $10,000 at any point → FBAR required; due April 15, auto-extension to Oct 15

Form 8938 (FATCA):
- MFJ US resident: >$100,000 year-end OR >$150,000 at any point
- Includes foreign mutual funds (PFICs) — not just bank accounts

NRE vs NRO flags:
- NRE interest: tax-free in India, TAXABLE in US → must appear on Schedule B
- NRO interest: taxable in both countries; TDS withheld → Form 1116 credit available

## Write output
Write to {OUTPUT_DIR}/fbar-summary.md:
- Table: Institution | Country | Type | Currency | Max Balance (local) | Max Balance (USD) | Dec 31 (USD)
- FBAR determination (required / not required)
- Form 8938 determination (required / not required)
- Schedule B Part III account list
- Any missing statement flags

Update {OUTPUT_DIR}/CLAUDE.md — change the FBAR row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/fbar-summary.md only.
