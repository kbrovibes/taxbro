Analyze Indian (or other foreign) mutual fund holdings for PFIC compliance.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Analyzing PFIC holdings for tax year {ACTIVE_YEAR}. (Override: /taxbro-check-pfic 2024)"

## Background
- Indian mutual funds are almost certainly PFICs (Passive Foreign Investment Companies)
- Form 8621 required per PFIC fund per year (exceptions apply)
- De minimis: aggregate value ≤ $50,000 year-end AND no prior QEF/MTM elections
- Three methods: (1) Default excess distribution, (2) QEF election, (3) Mark-to-Market

## Analysis

Read CLAUDE.md for prior election history and who holds the funds.
Find mutual fund statements for ACTIVE_YEAR using file-index.md (category: Mutual fund / PFIC, year: ACTIVE_YEAR).

For each fund, extract:
- Fund name and AMC
- ISIN or scheme code
- Units held as of December 31
- NAV and total value (INR) as of December 31
- Any redemptions or withdrawals during the year
- Any dividends paid or reinvested

Convert total portfolio value to USD (IRS Pub 54 rate).

Determinations:
a. De minimis: total USD ≤ $50,000 AND no prior elections → Form 8621 may not be required (flag for preparer)
b. Redemptions occurred → excess distribution calculation needed for those funds
c. No redemptions, no distributions, default method → no income to report but Form 8621 still required if above de minimis
d. If prior QEF/MTM elections exist → confirm continuation; switching requires purging election (costly)

## Write output
Write to {OUTPUT_DIR}/pfic-summary.md:
- Table: Fund | AMC | Units | Dec 31 NAV (INR) | Dec 31 Value (INR) | Dec 31 Value (USD) | Redemptions | Form 8621 Required
- Total portfolio USD value
- De minimis determination
- Election status and recommendation per fund

Update {OUTPUT_DIR}/CLAUDE.md — change the PFIC row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/pfic-summary.md only.
