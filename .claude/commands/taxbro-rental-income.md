Analyze foreign rental property income for US Schedule E reporting.

Background:
- Foreign rental income is taxable in the US and reported on Schedule E
- Convert foreign currency to USD using IRS annual average exchange rate (IRS Pub 54)
- Foreign taxes paid on rental income → Form 1116 credit (passive or general basket)
- Depreciation on foreign property is deductible; basis = purchase price at acquisition-date exchange rate
- India: tenant may deduct TDS at 31.2% (if tenant is a company/firm under Section 194-I)

Steps:
1. Get source folder from $ARGUMENTS or ~/claude/taxbro/.current-session.

2. Read CLAUDE.md for:
   - Rental property location and ownership structure (sole / co-owned / percentage)
   - Whether property is co-owned with spouse or others (affects income split)
   - Any known purchase price / acquisition date

3. Find rental income documents. Look in:
   - AIS/TIS files (India AIS shows rental income under "Income from other sources" or "Income from house property")
   - Bank statements (look for regular INR credits consistent with rent)
   - Rental agreement or rent receipts
   - Form 16A / TDS certificates from tenant

4. Extract per property:
   - Gross rent received (local currency, by month if available)
   - TDS deducted (if any, from Form 16A or AIS)
   - Net rent received (gross - TDS)
   - Filer's ownership share (e.g., 50% if co-owned equally)
   - Filer's share of gross rent

5. Currency conversion:
   - Use IRS Pub 54 annual average INR/USD rate for the tax year
   - Convert: filer's share of gross rent (INR) → USD
   - Convert: TDS withheld → USD (for Form 1116)

6. Schedule E items:
   - Gross rental income (USD)
   - Deductible expenses to ask about:
     * Property taxes paid in India (if any)
     * Repairs/maintenance paid
     * Property management fees
     * Mortgage interest (if property is mortgaged)
     * Depreciation (see below)
   - Net rental income or loss

7. Depreciation note:
   - Foreign residential property: 30-year straight-line depreciation (vs 27.5 years for US)
   - Basis: purchase price × filer's ownership % × USD rate at acquisition date
   - Annual depreciation = basis ÷ 30
   - Flag if depreciation basis has not been established (common oversight for foreign property)

8. Write output to {SOURCE_FOLDER}/TAXBRO/rental-income.md:
   - Property description (city/country, ownership %)
   - Monthly rent table (INR) with annual total
   - TDS deducted
   - USD conversion workings (rate used, source)
   - Schedule E income summary: gross rent, expenses, net income
   - Depreciation calculation (or flag if basis unknown)
   - Form 1116 amount to carry to /taxbro-foreign-tax-credit

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/rental-income.md only.
Never write financial data to ~/claude/taxbro/.
