Analyze all foreign bank account statements and determine FBAR / Form 8938 filing requirements.

Steps:
1. Get source folder from $ARGUMENTS or .current-session.

2. Read CLAUDE.md to identify all expected foreign accounts (institution, country, account type).

3. Find all foreign account statement files. Look for:
   - Files with bank names (ICICI, SBI, HDFC, Scotiabank, etc.)
   - Files numbered/labeled as foreign account statements
   - Subdirectory of monthly statements

4. For each account, extract:
   - Institution name and country
   - Account type (savings / NRE / NRO / Fixed Deposit / brokerage / checking)
   - Currency
   - Highest balance during the calendar year (any single day)
   - Balance on December 31
   Convert all balances to USD using IRS Publication 54 annual average exchange rates:
   - INR: use the average rate for the tax year (check IRS Pub 54 or Treasury fiscal data)
   - CAD: use the average rate for the tax year
   Note: For FBAR, use the Treasury's Financial Management Service rate (or a reasonable published rate).

5. FBAR determination:
   - Aggregate ALL foreign account max balances (USD equivalent)
   - If aggregate > $10,000 at any point: FBAR required (FinCEN 114)
   - FBAR due April 15; auto-extension to October 15

6. Form 8938 determination (FATCA):
   - MFJ US resident thresholds: >$100,000 year-end OR >$150,000 at any point
   - Check aggregate against both thresholds
   - Note: Form 8938 includes foreign financial accounts AND other foreign financial assets (mutual funds, etc.)

7. NRE vs NRO flags:
   - NRE: interest tax-free in India, but TAXABLE in US — must be reported on Schedule B
   - NRO: interest taxable in India (TDS withheld) + taxable in US — Form 1116 credit available

8. Write output to {SOURCE_FOLDER}/TAXBRO/US-fbar-summary.md:
   - Table per account: Institution | Country | Type | Currency | Max Balance (local) | Max Balance (USD) | Dec 31 Balance (USD)
   - FBAR filing determination (required / not required)
   - Form 8938 filing determination (required / not required)
   - List of accounts to include on Schedule B Part III
   - Flag any account where statement is missing or balance unclear

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/US-fbar-summary.md only.
Never write financial data to ~/claude/taxbro/.
