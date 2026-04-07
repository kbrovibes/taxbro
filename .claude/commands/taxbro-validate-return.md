Validate a completed or draft tax return PDF against source documents.
Cross-checks every key line item and flags discrepancies with the exact source document reference.

Usage: /taxbro-validate-return [path-to-return.pdf]
  - If a PDF path is given in $ARGUMENTS, use that as the return to validate.
  - Otherwise search {OUTPUT_DIR}/ for a PDF named draft/return/1040/prepared. Ask if ambiguous.

## Get source folder
Read ~/claude/taxbro/.current-session, or use path from $ARGUMENTS.
If missing: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md. If missing: run /taxbro-init first.
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/  (create if needed)
4. Tell user: "Validating return for tax year {ACTIVE_YEAR}. (Override: /taxbro-validate-return 2024)"

## Identify the return PDF
- If $ARGUMENTS contains a .pdf path, use it.
- Otherwise search OUTPUT_DIR for a draft return PDF.
- Ask user if no clear candidate found.

## Read source data
Read the return PDF. Note which forms/schedules are present.
Read all TaxBro outputs that exist in OUTPUT_DIR:
w2-summary.md, fbar-summary.md, ftc-summary.md, pfic-summary.md,
rental-income.md, childcare-summary.md, worksheets.md

## Validation checks

For each check: ✅ PASS (within $1–2) | ⚠️ FLAG (>$10 diff) | ❌ MISMATCH (>$50 or directional error) | SKIP (form absent)

**Form 1040 — Income**
- [ ] Line 1a (wages): vs sum of W-2 Box 1
- [ ] Line 2b (taxable interest): vs all 1099-INT + NRE/NRO certificates
- [ ] Line 3b (ordinary dividends): vs all 1099-DIV Box 1a
- [ ] Line 3a (qualified dividends): vs all 1099-DIV Box 1b
- [ ] Line 7 (capital gain/loss): vs net from all 1099-Bs

**Schedule B**
- [ ] Part I interest: each line traceable to a 1099-INT or interest certificate
- [ ] Part II dividends: each line traceable to a 1099-DIV
- [ ] Part III foreign accounts: all FBAR accounts match Schedule B Part III

**Schedule E**
- [ ] Gross rents: vs rental-income.md USD amount
- [ ] Depreciation: vs rental-income.md calculation
- [ ] Foreign taxes: vs ftc-summary.md rental TDS entry

**Form 1116 — Passive**
- [ ] Foreign income (Line 9): vs sum of NRO interest + dividends + rental (passive) from ftc-summary.md
- [ ] Foreign taxes (Lines 2–5): vs TDS on NRO accounts
- [ ] Correct basket used

**Form 1116 — General (if present)**
- [ ] Advance tax payments: vs advance tax docs
- [ ] Rental taxes (if active): vs TDS on rental

**Form 2441**
- [ ] Provider names + EINs: vs childcare provider statements
- [ ] Total expenses claimed: ≤ amounts paid
- [ ] Box 10 DCFSA offset applied correctly
- [ ] Earned income test: flag if spouse has no earned income

**Form 8938**
- [ ] All accounts listed: vs FBAR account list
- [ ] Max values: within range of fbar-summary.md
- [ ] Foreign financial assets (mutual funds) included if above threshold

**Form 8621**
- [ ] One form per fund with reportable activity
- [ ] Fund names vs pfic-summary.md
- [ ] Method consistency
- [ ] No phantom income for funds with no distributions (default method)

**Excess SS**
- [ ] Credit on Schedule 3 Line 11 vs w2-summary.md excess SS calculation

**HSA**
- [ ] Form 8889 present if HSA distributions taken
- [ ] Contribution limit not exceeded

## Write output
Write to {OUTPUT_DIR}/validation-report.md:
- Header: return filename, validation date, tax year
- Results table: Form/Line | Return Value | Source Value | Source Doc | Status
- Summary: N passed, N flagged, N mismatched, N skipped
- Flagged items: discrepancy explanation + suggested resolution
- "Items to confirm with preparer": anything not validatable due to missing data

Update {OUTPUT_DIR}/CLAUDE.md — change the Validation row from "⏳ not run" to "✅ complete", set today's date.

Print brief summary to user: total checks, pass/flag/mismatch counts, top issues.

IMPORTANT: Never write financial data to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/validation-report.md only.
