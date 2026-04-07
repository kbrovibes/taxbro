Analyze all W-2 forms in the source folder.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Analyzing W-2s for tax year {ACTIVE_YEAR}. (Override: /taxbro-check-w2s 2024)"

## Analysis

Find all W-2 files for ACTIVE_YEAR using file-index.md (category: W-2, year: ACTIVE_YEAR).

For each W-2, extract:
- Employer name
- Box 1: Wages, tips, other compensation
- Box 2: Federal income tax withheld
- Box 4: Social Security tax withheld
- Box 6: Medicare tax withheld
- Box 10: Dependent care benefits (DCFSA)
- Box 12 codes (D=401k, W=HSA employer, DD=health insurance)
- Box 16/17: State wages and state tax

Calculations:
a. Sum all Box 4 across W-2s.
   - 2025 SS wage base: $176,100; max employee SS = $10,918.20
   - If sum > cap → excess SS = refundable credit on Form 1040 Line 11
b. Sum Box 10 → for Form 2441 coordination with /taxbro-check-childcare
c. Sum Box 12 Code W (HSA employer) → for HSA limit check
d. If CLAUDE.md notes spouse has no earned income → flag Form 2441 risk

## Write output
Write to {OUTPUT_DIR}/w2-summary.md:
- Table: Employer | Box1 Wages | Box2 Fed Withheld | Box4 SS | Box10 DCFSA
- Excess SS calculation (if applicable)
- Form 2441 earned-income flag (if applicable)
- Notable Box 12 items

Update {OUTPUT_DIR}/CLAUDE.md — change the W-2 summary row from "⏳ not run" to "✅ complete", set today's date.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/w2-summary.md only.
