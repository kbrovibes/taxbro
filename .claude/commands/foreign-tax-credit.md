Consolidate all foreign taxes paid for Form 1116 (Foreign Tax Credit).

Background:
- Form 1116 required when foreign taxes are paid/accrued on foreign income
- Two main income baskets: (1) Passive (interest, dividends, rental) and (2) General (wages, rental if active)
- Indian TDS on NRO interest/savings → passive basket
- Indian TDS on rental income → could be passive or general depending on rental activity level
- Advance tax payments to India → general basket (on total India income including salary/other)
- NRE interest: NOT taxable in India → no TDS → no Form 1116 credit for NRE interest

Steps:
1. Get source folder from $ARGUMENTS or ~/claude/taxbro/.current-session.

2. Find all documents containing foreign tax information:
   - Interest certificates (Indian banks: TDS deducted amounts)
   - AIS/TIS (India Annual Information Statement — shows TDS credits)
   - Advance tax payment receipts
   - Rental income TDS certificates (Form 16A from tenant if applicable)
   - Any other foreign country tax documents (Canadian T4/NR4 if applicable)

3. For each foreign tax item, extract:
   - Country
   - Type of income (interest / rental / dividend / other)
   - Gross income (local currency + USD)
   - Tax withheld or paid (local currency + USD)
   - Form 1116 basket (passive / general)
   - Source document

4. Convert all amounts to USD using IRS Pub 54 rates.

5. Passive basket summary:
   - NRO interest income → gross + TDS
   - Dividend income from Indian funds (if any) → gross + TDS
   - Rental income (if passive) → gross + TDS

6. General basket summary:
   - Advance tax payments (India) → allocate to income types
   - Rental income (if active) → gross + tax
   - Canadian taxes (if any)

7. Form 1116 limitation note:
   - FTC limited to: (Foreign income / Total income) × US tax
   - Excess FTC carries forward 10 years
   - Flag if India taxes likely exceed the FTC limitation (common for high-bracket US filers)

8. Write output to {SOURCE_FOLDER}/TAXBRO/ftc-summary.md:
   - Table per item: Country | Income Type | Gross Income (USD) | Foreign Tax (USD) | Basket | Source Doc
   - Subtotals by basket
   - Form 1116 filing recommendation
   - Any carryforward notes

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/ftc-summary.md only.
Never write financial data to ~/claude/taxbro/.
