Generate preparer-ready IRS form worksheets from the knowledge graph.

This skill reformats data from US-knowledge-graph.md into IRS form layout that a
CPA or tax software can enter directly. The heavy computation is already done in the
knowledge graph's Computed Totals section — this skill organizes it per-form.

Usage:
  /taxbro-worksheets          — generate all applicable worksheets

Output: {SOURCE_FOLDER}/TAXBRO/US-worksheets.md

⚠️ NEVER DELETE ANY FILE. Only write to TAXBRO/US-worksheets.md.
⚠️ COMPUTATION RULE: Read "Computed Totals" from US-knowledge-graph.md. Use those values.
   Do NOT independently recompute. All line amounts MUST match Computed Totals.

---

## Steps

1. Read `.current-session` → SOURCE_FOLDER.
2. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` — the Computed Totals section is authoritative.
3. Read `{SOURCE_FOLDER}/CLAUDE.md` for filer context.
4. Read `{SOURCE_FOLDER}/TAXBRO/US-compliance-summary.md` if it exists (for FBAR/PFIC/FTC detail).

---

## Worksheets to Generate

### Form 1040 — Key Lines
Use Computed Totals. Show each line with source/formula.

| Line | Description | Amount | Source |
|------|-------------|--------|--------|
| 1a | Wages | | W-2 total (+ DCFSA add-back if applicable) |
| 2b | Taxable interest | | US + foreign interest |
| 3a | Qualified dividends | | |
| 3b | Ordinary dividends | | |
| 7 | Capital gain (ADJUSTED) | | After RSU basis correction |
| 8 | Other income (Sch 1) | | Rental, etc. |
| 9 | Total income | | Sum |
| 11 | AGI | | |
| 12 | Standard or itemized | | Show comparison |
| 15 | Taxable income | | |
| 16 | Tax | | From brackets (note: simplified) |
| 23 | Other taxes | | Add'l Medicare + NIIT |
| 24 | Total tax | | |
| 25a | W-2 withholding | | |
| 26 | Estimated payments | | |
| 33 | Total payments | | Including credits |
| 34 | Overpayment | | |
| 37 | Amount owed | | |
| 38 | Estimated penalty | | Form 2210 |

### Schedule E — Foreign Rental
- Property description, days rented
- Gross rent (USD), depreciation (ADS 30-yr), expenses
- Net income, PAL carryforward applied

### Form 1116 — Passive Basket
- Country, income type, gross income, taxes paid
- Limitation fraction, allowable credit
- Carryforward available

### Form 1116 — General Basket (if applicable)
Same structure as passive.

### FinCEN 114 (FBAR)
Per account: institution, type, account number, max value (USD), owner, country.
Note: file at bsaefiling.fincen.treas.gov.

### Form 8938 (FATCA)
Part I (deposit/custodial accounts) and Part II (other assets like PFIC funds).
Max value, year-end value, income generated.

### Form 8621 (PFIC) — per fund if required
Only generate if knowledge graph shows Form 8621 required (above de minimis).
Per fund: fund name, shares, value, method, distributions.

### Form 2441 (Childcare)
Provider table, DCFSA offset, earned income test result.
**Flag DCFSA add-back if spouse $0 earned income (IRC §129).**

### Form 2210 (Underpayment Penalty)
- Quarterly income attribution (from knowledge graph)
- Payment timing (when estimated payments were made)
- Safe harbor analysis (110% of prior year tax if AGI > $150K)
- Annualized income method note (if gains concentrated in Q4)
- Estimated penalty amount

---

## "What Your Preparer Still Needs" Section

At the end, list items the preparer must compute that TaxBro cannot:
- Exact tax from Qualified Dividends and Capital Gain Tax Worksheet
- AMT calculation (if applicable)
- Form 1116 exact limitation (requires final US tax figure)
- Mortgage interest deductibility ratio (requires 1098 Box 2 principal)
- State return items

---

## Privacy Note
All output goes to {SOURCE_FOLDER}/TAXBRO/US-worksheets.md only.
Never write financial data to ~/claude/taxbro/.
