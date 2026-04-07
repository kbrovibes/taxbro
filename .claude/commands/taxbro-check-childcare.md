Analyze childcare expenses and determine Form 2441 (Child and Dependent Care Credit) eligibility.

Steps:
1. Get source folder from $ARGUMENTS or .current-session.

2. Read CLAUDE.md for:
   - Number of qualifying children
   - Whether spouse has earned income (critical for Form 2441)
   - Whether employer DCFSA/FSA was used (W-2 Box 10 amount)

3. Find all childcare provider documents (statements, receipts, payment histories).

4. For each provider, extract:
   - Provider name and address
   - Provider EIN or SSN (required for Form 2441)
   - Total amount paid during the tax year
   - Period of care covered

5. Calculate qualifying expenses:
   - Maximum qualifying expenses: $3,000 for 1 child, $6,000 for 2+ children
   - If W-2 Box 10 (DCFSA) amount exists: subtract from qualifying expenses
     (DCFSA is pre-tax; you cannot double-dip on the same dollars)
   - Net qualifying expenses = min(total paid, max) - Box 10 DCFSA amount

6. Earned income test — CRITICAL:
   - Both spouses must have earned income to claim the credit
   - Exceptions: a spouse who is (a) a full-time student for at least 5 months, or (b) disabled
   - If CLAUDE.md indicates spouse does not work AND no exception applies:
     FLAG: "Form 2441 credit likely DISALLOWED — spouse has no earned income. Confirm exception status with tax preparer."
   - Note: even if credit is disallowed, DCFSA (Box 10) pre-tax benefit is still valid

7. Credit rate:
   - Credit is non-refundable, 20%–35% of qualifying expenses depending on AGI
   - At AGI > $43,000 (2025): rate is 20%
   - Estimate credit = net qualifying expenses × 20% (conservative for higher-income filers)

8. Write output to {SOURCE_FOLDER}/TAXBRO/US-childcare-summary.md:
   - Table: Provider | EIN | Amount Paid | Qualifying Amount
   - DCFSA offset calculation
   - Net qualifying expenses
   - Estimated credit (with earned-income caveat if applicable)
   - Form 2441 eligibility determination and any flags

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/US-childcare-summary.md only.
Never write financial data to ~/claude/taxbro/.
