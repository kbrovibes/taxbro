Validate a completed or draft tax return PDF against source documents.
Cross-checks every key line item and flags discrepancies with the exact source document reference.

Usage: /validate-return [path-to-return.pdf]
  - If a PDF path is given in $ARGUMENTS, use that as the return to validate.
  - Otherwise look for a PDF in {SOURCE_FOLDER}/TAXBRO/ with a name suggesting a draft return
    (e.g., "draft", "return", "1040", "prepared"). Ask user to confirm if ambiguous.

Steps:
1. Get source folder from .current-session (or from $ARGUMENTS if it contains a folder path).

2. Identify the return PDF to validate:
   - If $ARGUMENTS contains a .pdf path, use it.
   - Otherwise search {SOURCE_FOLDER}/TAXBRO/ for a draft return PDF.
   - Ask the user if no clear candidate is found.

3. Read the return PDF. Identify which forms/schedules are present:
   - Form 1040
   - Schedule B (interest and dividends)
   - Schedule E (rental income)
   - Schedule D / Form 8949 (capital gains)
   - Form 1116 (foreign tax credit) — one or more
   - Form 2441 (childcare)
   - Form 8938 (FATCA)
   - Form 8621 (PFIC) — one or more
   - Any state returns

4. Also read all TaxBro analysis outputs that exist in {SOURCE_FOLDER}/TAXBRO/:
   w2-summary.md, fbar-summary.md, ftc-summary.md, pfic-summary.md,
   rental-income.md, childcare-summary.md, worksheets.md

5. Run the following validations. For each check, record:
   - PASS ✅ — numbers match within rounding tolerance ($1–2)
   - FLAG ⚠️ — difference worth investigating (>$10)
   - MISMATCH ❌ — clear discrepancy (>$50 or directional error)
   - SKIP — form not present in return or source data unavailable

---

VALIDATION CHECKS:

**Form 1040 — Income**
- [ ] Line 1a (W-2 wages): return value vs sum of Box 1 across all W-2s
- [ ] Line 2b (taxable interest): return value vs sum of all 1099-INTs + NRE/NRO interest certificates
- [ ] Line 3b (ordinary dividends): return value vs sum of all 1099-DIV Box 1a
- [ ] Line 3a (qualified dividends): return value vs sum of all 1099-DIV Box 1b
- [ ] Line 7 (capital gain/loss): return value vs net cap gain from all 1099-Bs

**Schedule B**
- [ ] Part I interest: each line item traceable to a specific 1099-INT or interest certificate
- [ ] Part II dividends: each line item traceable to a specific 1099-DIV
- [ ] Part III: foreign accounts disclosed — check that all accounts in FBAR match Schedule B Part III

**Schedule E (rental)**
- [ ] Gross rents received: return value vs rental-income.md USD amount
- [ ] Depreciation: return value vs depreciation calculated in rental-income.md
- [ ] Foreign taxes on rental: matches ftc-summary.md rental TDS entry

**Form 1116 — Passive Basket**
- [ ] Foreign income (Line 9): matches sum of NRO interest + foreign dividends + rental (if passive) from ftc-summary.md
- [ ] Foreign taxes (Lines 2–5): matches total TDS on NRO accounts from interest certificates
- [ ] Correct basket used (passive vs general): NRO interest → passive

**Form 1116 — General Basket (if present)**
- [ ] Advance tax payments included: matches ₹75,000 (or whatever total) from advance tax payment docs
- [ ] Rental income taxes (if active): matches TDS on rental from ftc-summary.md

**Form 2441 (childcare)**
- [ ] Provider names + EINs: match childcare provider statements
- [ ] Total expenses claimed: ≤ amounts paid per provider statements
- [ ] Box 10 DCFSA offset applied correctly
- [ ] Earned income test: flag if spouse has no earned income and no exception noted on return

**Form 8938 (FATCA)**
- [ ] All foreign accounts listed: cross-reference with FBAR account list
- [ ] Max values: within range of fbar-summary.md figures
- [ ] Foreign financial assets (mutual funds) included if PFIC holdings above threshold

**Form 8621 (PFIC)**
- [ ] One form per fund with reportable activity
- [ ] Fund names match CAMS CAS / pfic-summary.md
- [ ] If no distributions and no elections: Section 1298(f) annual reporting confirmed
- [ ] No phantom income reported for funds with no distributions (under default method)

**W-2 / Excess SS**
- [ ] If excess SS withheld: credit appears on Form 1040 Schedule 3 Line 11
- [ ] Amount matches w2-summary.md excess SS calculation

**HSA**
- [ ] Form 8889 present if HSA distributions taken
- [ ] Contribution limit not exceeded (per 5498-SA vs W-2 Box 12 Code W)

---

6. Write output to {SOURCE_FOLDER}/TAXBRO/validation-report.md:
   - Header: return filename validated, date of validation, tax year
   - Results table: Form/Line | Return Value | Source Value | Source Document | Status
   - Summary: N passed, N flagged, N mismatched, N skipped
   - Flagged items section: for each FLAG or MISMATCH, explain the discrepancy and suggest resolution
   - "Items to confirm with preparer" section: anything that could not be validated due to missing source data

7. Print a brief summary to the user: total checks run, pass/flag/mismatch counts, and top issues to review.

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/validation-report.md only.
Never write financial data to ~/claude/taxbro/.
