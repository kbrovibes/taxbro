Generate pre-filled worksheets for all applicable IRS forms based on source documents.
Outputs structured, line-item-ready reference documents your preparer or tax software can enter directly.

Steps:
1. Get source folder from $ARGUMENTS or ~/claude/taxbro/.current-session.

2. Read CLAUDE.md to understand which forms apply to this filer.

3. Check which TaxBro analysis outputs already exist in {SOURCE_FOLDER}/TAXBRO/:
   - fbar-summary.md → drives FinCEN 114 worksheet
   - ftc-summary.md → drives Form 1116 worksheets
   - pfic-summary.md → drives Form 8621 worksheets
   - rental-income.md → drives Schedule E worksheet
   - w2-summary.md → drives Form 1040 wages section + excess SS credit
   - childcare-summary.md → drives Form 2441 worksheet

   For any missing analysis file, run the corresponding skill first and notify the user:
   "Missing [skill output] — please run /[skill] first, then re-run /generate-worksheets."
   Continue generating worksheets for forms where analysis is already complete.

4. Generate the following worksheets as applicable. Write each as a clearly labeled section
   in {SOURCE_FOLDER}/TAXBRO/worksheets.md:

---

### WORKSHEET: FinCEN 114 (FBAR)
Required fields per account:
- Financial institution name and address
- Account type (bank / securities / other)
- Account number
- Maximum value during year (USD)
- Account owner (filer / spouse / joint)
- Country

Output as a table. Note: file at bsaefiling.fincen.treas.gov (free, no software needed).

---

### WORKSHEET: Form 8938 (FATCA Statement of Foreign Financial Assets)
Part I — Foreign deposit and custodial accounts:
- Institution name, address, account number
- Max value during year / year-end value
- Did account generate income? (Y/N) → income type and amount

Part II — Other foreign financial assets (mutual funds, etc.):
- Description and issuer
- Max value / year-end value
- Income generated

Total asset value for filing threshold check.

---

### WORKSHEET: Form 1116 — Passive Basket (interest, dividends, rental if passive)
Line 1a: Foreign country name(s)
Line 1b: Description of income
Lines 2–6: Taxes withheld (itemized per country/income type)
Line 9: Net foreign income (gross less deductible expenses)
Line 20: US tax before credits (carry from 1040 — leave blank for preparer)
Limitation fraction: foreign income ÷ total income (estimate)
Estimated allowable credit

---

### WORKSHEET: Form 1116 — General Basket (if advance tax payments or active rental)
Same structure as passive basket, separate column.

---

### WORKSHEET: Form 8621 (per PFIC fund with reportable activity)
For each fund requiring Form 8621:
- Fund name, address, reference ID (ISIN)
- Tax year of fund
- Shareholder's tax year
- Check box: which method applies (default / QEF / MTM)
- Part I (default method): distributions received this year; prior year earnings; excess distribution calculation
- If no distributions and no elections: note "Section 1298(f) annual reporting only"

---

### WORKSHEET: Schedule E (Rental Income — Foreign Property)
Per property:
- Property address (foreign)
- Days rented / days personal use
- Line 3: Rents received (USD)
- Line 5: Advertising
- Line 9: Insurance
- Line 11: Mortgage interest (if applicable)
- Line 12: Other interest
- Line 13: Repairs
- Line 16: Taxes (India property tax if paid)
- Line 18: Depreciation (from rental-income.md calculation)
- Line 19: Other expenses
- Line 21: Total expenses
- Line 22: Net income / (loss)

---

### WORKSHEET: Form 2441 (Child and Dependent Care)
Part I — Persons who provided care:
- Provider name, address, EIN
- Amount paid

Part II:
- Line 2: Qualifying expenses paid
- Line 3: Earned income (filer) / Earned income (spouse) — FLAG if spouse has no earned income
- Line 6: Dollar limit ($3,000 / $6,000)
- Line 9: Employer-provided benefits (W-2 Box 10)
- Line 11: Adjusted qualifying expenses
- Line 14: Credit percentage (based on AGI)
- Line 15: Estimated credit amount

---

### WORKSHEET: Form 1040 — Key Lines Summary
- Line 1a: W-2 wages (total Box 1)
- Line 2b: Taxable interest (US + foreign NRE/NRO)
- Line 3b: Ordinary dividends
- Line 3a: Qualified dividends
- Line 5b: IRA/pension distributions (if any)
- Line 7: Capital gain or loss (net from brokerages)
- Schedule 1 Line 8: Other income (rental net, if positive)
- Schedule A: Mortgage interest (if itemizing)
- Line 19: Child tax credit
- Line 13: Foreign tax credit (from Form 1116)
- Line 11: Excess SS tax withheld (from w2-summary)

---

5. At the end of worksheets.md, add a "What your preparer still needs to fill in" section:
   - US tax before credits (Line 16 of 1040) — needed to finalize Form 1116 limitation
   - AGI — needed to finalize Form 2441 credit rate
   - Capital gains detail (short vs long term) — from 1099-B Schedule D
   - State return items (state-specific)

6. Print a summary of which worksheets were generated and which were skipped (missing analysis).

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/worksheets.md only.
Never write financial data to ~/claude/taxbro/.
