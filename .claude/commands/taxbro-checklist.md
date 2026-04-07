Run a full status checklist across all tax issues for the loaded source folder.

Steps:
1. Get source folder:
   - If $ARGUMENTS is provided, use that path.
   - Otherwise read ~/claude/taxbro/.current-session for the path.
   - If neither exists, ask: "Please run /taxbro-init first, or pass your tax folder path as an argument."

2. Read {SOURCE_FOLDER}/CLAUDE.md to understand this filer's specific situation, applicable forms, and flagged issues.

3. For each issue listed in CLAUDE.md (or the standard checklist below if no CLAUDE.md), check:
   a. Is the required source document present in the folder?
      Use `find {SOURCE_FOLDER} -follow -type f` to discover all files including those reachable via symlinks.
   b. Has TaxBro already generated an output file for it in TAXBRO/?
   c. Are there any open flags or risks noted?

Standard checklist items (adapt based on CLAUDE.md):
- [ ] W-2(s) — all employers present; excess SS check done
- [ ] 1099-B/DIV/INT — all brokerages present; wash sale review needed
- [ ] 1098(s) — mortgage interest; $750K acquisition debt limit check
- [ ] Childcare — provider statements present; Form 2441 earned-income eligibility confirmed
- [ ] HSA — 1099-SA + 5498-SA present; contribution limit check done
- [ ] Foreign accounts — all statements present; FBAR threshold check done; Form 8938 threshold check done
- [ ] PFIC/Mutual funds — CAS/statements present; Form 8621 determination done
- [ ] Foreign interest — certificates present; Form 1116 (passive basket) ready
- [ ] Rental income — income documented; Schedule E amounts calculated
- [ ] Foreign tax payments — advance payments documented; Form 1116 (general basket) ready
- [ ] Canada — T4 check done; confirm no residual withholding obligation
- [ ] Schedule B Part III — foreign account disclosure ready

4. Output a status table with three columns: Issue | Documents Present | Analysis Done | Risk/Flag

5. Write the output to {SOURCE_FOLDER}/TAXBRO/checklist.md with today's date as header.

6. Summarize: how many items are ✅ clear, ⚠️ need attention, ❌ blocked by missing documents.

IMPORTANT: Never write any PII, account numbers, or financial figures to ~/claude/taxbro/.
All output goes to {SOURCE_FOLDER}/TAXBRO/checklist.md only.
