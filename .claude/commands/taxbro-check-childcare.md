Analyze childcare expenses and determine Form 2441 (Child and Dependent Care Credit) eligibility.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Analyzing childcare for tax year {ACTIVE_YEAR}. (Override: /taxbro-check-childcare 2024)"

## Analysis

Read CLAUDE.md for: number of qualifying children, spouse employment status, DCFSA usage.
Also read {OUTPUT_DIR}/w2-summary.md if it exists — for Box 10 DCFSA amounts.
Find childcare documents for ACTIVE_YEAR using file-index.md (category: Childcare, year: ACTIVE_YEAR).

For each provider, extract:
- Provider name and address
- Provider EIN or SSN (required on Form 2441)
- Total paid during the tax year
- Period of care

Calculations:
- Qualifying expense cap: $3,000 for 1 child, $6,000 for 2+ children
- DCFSA offset: subtract W-2 Box 10 amount — pre-tax dollars, cannot double-dip
- Net qualifying expenses = min(total paid, cap) - Box 10 DCFSA

Earned income test — CRITICAL:
- Both spouses must have earned income to claim the credit
- Exceptions: full-time student (5+ months) or disabled spouse
- If CLAUDE.md indicates spouse has no earned income AND no exception:
  FLAG: "Form 2441 credit likely DISALLOWED — spouse has no earned income. DCFSA Box 10 benefit is still valid."

Credit estimate:
- AGI > $43,000: 20% rate (conservative for higher-income filers)
- Estimated credit = net qualifying expenses × 20%

## Write output
Write to {OUTPUT_DIR}/childcare-summary.md:
- Table: Provider | EIN | Amount Paid | Qualifying Amount
- DCFSA offset calculation
- Net qualifying expenses
- Estimated credit (with earned-income caveat if applicable)
- Form 2441 eligibility determination

Update {OUTPUT_DIR}/CLAUDE.md — change the Childcare row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/childcare-summary.md only.
