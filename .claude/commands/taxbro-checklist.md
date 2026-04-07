Run a full status checklist across all tax issues for the loaded source folder.

## Get source folder
- If $ARGUMENTS contains a folder path, use it.
- Otherwise read ~/claude/taxbro/.current-session.
- If neither exists: "Please run /taxbro-init first."

## Determine active year
1. Read {SOURCE_FOLDER}/TAXBRO/file-index.md.
   - If missing: "Please run /taxbro-init first."
2. If $ARGUMENTS contains a 4-digit year (2020–2030), use it as ACTIVE_YEAR.
   Otherwise use the most recent year listed in file-index.md.
3. OUTPUT_DIR = {SOURCE_FOLDER}/TAXBRO/{ACTIVE_YEAR}/
   Create if it doesn't exist.
4. Tell user: "Running checklist for tax year {ACTIVE_YEAR}. (Override: /taxbro-checklist 2024)"

## Run the checklist

Read {SOURCE_FOLDER}/CLAUDE.md and {OUTPUT_DIR}/CLAUDE.md for filer context.

For each applicable issue, check:
a. Are required source documents present? (use file-index.md — filter for ACTIVE_YEAR)
b. Has TaxBro already run analysis? (check if output file exists in OUTPUT_DIR)
c. Any open risk flags?

Standard items (adapt from CLAUDE.md):
- [ ] W-2(s) — all employers present; excess SS check done
- [ ] 1099-B/DIV/INT — all brokerages present; wash sale review needed
- [ ] 1098(s) — mortgage interest; $750K acquisition debt limit check
- [ ] Childcare — provider statements present; Form 2441 earned-income eligibility confirmed
- [ ] HSA — 1099-SA + 5498-SA present; contribution limit check done
- [ ] Foreign accounts — all statements present; FBAR threshold check done; Form 8938 check done
- [ ] PFIC/Mutual funds — CAS/statements present; Form 8621 determination done
- [ ] Foreign interest — certificates present; Form 1116 passive basket ready
- [ ] Rental income — income documented; Schedule E amounts calculated
- [ ] Foreign tax payments — advance payments documented; Form 1116 general basket ready
- [ ] Canada — T4 check done; residual withholding confirmed
- [ ] Schedule B Part III — foreign account disclosure ready

Output a status table: Issue | Documents Present | Analysis Done | Risk/Flag

Summarize: N ✅ clear, N ⚠️ need attention, N ❌ blocked.

## Write output
Write to {OUTPUT_DIR}/checklist.md with today's date as header.

Update the checklist row in {OUTPUT_DIR}/CLAUDE.md:
- Change "⏳ not run" to "✅ complete" for the checklist row
- Set "Last updated" to today's date

IMPORTANT: Never write PII, account numbers, or financial figures to ~/claude/taxbro/.
All output goes to {OUTPUT_DIR}/checklist.md only.
