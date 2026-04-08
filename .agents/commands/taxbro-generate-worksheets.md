Generate pre-filled worksheets for all applicable IRS forms based on source documents.
Outputs structured, line-item-ready reference documents your preparer or tax software can enter directly.

⚠️ COMPUTATION RULE: Read the "Computed Totals" section of US-knowledge-graph.md FIRST.
Use those pre-computed values for total income, AGI, taxable income, total tax, total payments,
and refund/owe. Do NOT independently recompute these from raw sections — this causes agent drift.
Always use ADJUSTED capital gain (after RSU basis correction), not the face 1099-B value.
Total interest = US 1099-INT + India NRE (USD) + India NRO (USD).

Steps:
1. Get source folder from $ARGUMENTS or .current-session.

2. Read CLAUDE.md to understand which forms apply to this filer.

2b. **Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` — specifically the "Computed Totals" section.**
    All line amounts in the Form 1040 worksheet MUST match the Computed Totals. If Computed Totals
    is missing, compute the values yourself following the formulas in the extract schema and note "⚠️ no Computed Totals — verify".

3. Check which TaxBro analysis outputs already exist in {SOURCE_FOLDER}/TAXBRO/:
   - US-fbar-summary.md → drives FinCEN 114 worksheet
   - US-ftc-summary.md → drives Form 1116 worksheets
   - US-pfic-summary.md → drives Form 8621 worksheets
   - US-rental-income.md → drives Schedule E worksheet
   - US-w2-summary.md → drives Form 1040 wages section + excess SS credit
   - US-childcare-summary.md → drives Form 2441 worksheet

   For any missing analysis file, run the corresponding skill first and notify the user:
   "Missing [skill output] — please run /[skill] first, then re-run /generate-worksheets."
   Continue generating worksheets for forms where analysis is already complete.

4. Generate the following worksheets as applicable. Write each as a clearly labeled section
   in {SOURCE_FOLDER}/TAXBRO/US-worksheets.md:

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
- Line 18: Depreciation (from US-rental-income.md calculation)
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

### WORKSHEET: Form 2210 (Underpayment of Estimated Tax)
Required for high-income earners with significant non-wage income (stocks/rental).
- **Quarterly Income Attribution:** 
  - Q1 (Jan–Mar): List significant realized gains/bonuses.
  - Q2 (Apr–May): List significant realized gains/bonuses.
  - Q3 (Jun–Aug): List significant realized gains/bonuses.
  - Q4 (Sep–Dec): List significant realized gains/bonuses.
- **Estimated Tax Payments:**
  - Date and Amount for each payment (must match 1040-ES receipts).
- **Penalty Calculation:**
  - **Required Annual Payment:** The *lesser* of (a) 90% of current year tax or (b) 110% of prior year tax (for high earners).
  - Estimate penalty = (Required Annual Payment - Withholding) × IRS interest rate (approx 8% for 2025) weighted by quarter.
  - **Critical:** If income was earned late in the year (e.g., Q4 stock gains), note that the **Annualized Income Method (Schedule AI)** will likely reduce this penalty significantly.

---

### WORKSHEET: Form 1040 — Key Lines Summary
...
- Line 34: Overpayment (Refund)
- Line 38: Estimated Penalty (from Form 2210 worksheet)
- **Net Outcome:** Overpayment - Penalty.

---

5. At the end of US-worksheets.md, add a "What your preparer still needs to fill in" section:
   - US tax before credits (Line 16 of 1040) — needed to finalize Form 1116 limitation
   - AGI — needed to finalize Form 2441 credit rate
   - Capital gains detail (short vs long term) — from 1099-B Schedule D
   - State return items (state-specific)

6. Print a summary of which worksheets were generated and which were skipped (missing analysis).

7. Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` recording: agent-id (e.g., `gemini-2.0-flash`), skill-name, status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/US-worksheets.md only.
Never write financial data to ~/claude/taxbro/.
