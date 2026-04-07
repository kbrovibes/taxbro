Analyze foreign rental property income for US Schedule E reporting.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Analyzing rental income for tax year {ACTIVE_YEAR}. (Override: /taxbro-rental-income 2024)"

## Background
- Foreign rental income is taxable in the US and reported on Schedule E
- Convert foreign currency to USD using IRS Pub 54 annual average exchange rate
- Foreign taxes paid on rental → Form 1116 credit (passive or general basket)
- Foreign residential property depreciates over 30 years (not 27.5 like US)
- India: tenant may deduct TDS at 31.2% (Section 194-I, company/firm tenants)

## Analysis

Read CLAUDE.md for: property location, ownership %, purchase price / acquisition date, co-owners.
Find rental documents for ACTIVE_YEAR using file-index.md (categories: AIS/TIS, Foreign bank, Interest certificate).

For each property, extract:
- Gross rent received (local currency, monthly if available)
- TDS deducted (from Form 16A or AIS)
- Net rent received
- Filer's ownership share
- Filer's share of gross rent

Currency conversion:
- IRS Pub 54 annual average INR/USD rate for ACTIVE_YEAR
- Convert: filer's share of gross rent (INR) → USD
- Convert: TDS withheld → USD (for Form 1116)

Schedule E items:
- Gross rental income (USD)
- Deductible expenses (ask if not in documents): property taxes, repairs, management fees, mortgage interest
- Depreciation: basis = purchase price × ownership % × USD rate at acquisition date; annual = basis ÷ 30
- Flag if depreciation basis not established (common oversight)

## Write output
Write to {OUTPUT_DIR}/rental-income.md:
- Property description (city/country, ownership %)
- Monthly rent table (INR) with annual total
- TDS deducted
- USD conversion workings (rate used, source)
- Schedule E income summary: gross rent, expenses, net income
- Depreciation calculation (or flag if basis unknown)
- Form 1116 amount to carry to /taxbro-foreign-tax-credit

Update {OUTPUT_DIR}/CLAUDE.md — change the Rental income row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/rental-income.md only.
