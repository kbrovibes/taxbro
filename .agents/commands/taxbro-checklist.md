Run a comprehensive tax status review for the loaded source folder.

This skill is knowledge-graph-first: it reads US-knowledge-graph.md when available (built by /taxbro-extract),
and falls back to raw document scanning only if the graph has not been built yet.

⚠️ COMPUTATION RULE: When citing dollar amounts, use values from the "Computed Totals" section of the
knowledge graph. Do NOT independently recompute. Use ADJUSTED capital gain (after RSU basis correction).
Total interest = US 1099-INT + India NRE (USD) + India NRO (USD).

Steps:
1. Get source folder:
   - If $ARGUMENTS is provided, use that path.
   - Otherwise read .current-session for the path.
   - If neither exists, ask: "Please run /taxbro-init first, or pass your tax folder path as an argument."

2. Check for knowledge graph:
   - If {SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md exists: READ IT. This is the primary data source.
     Use the structured facts, flags, and cross-checks already computed by /taxbro-extract.
   - If it does NOT exist: inform the user —
     "Knowledge graph not found. Run /taxbro-extract first for comprehensive analysis.
      Falling back to document discovery for a basic structural check."
     Then use `find {SOURCE_FOLDER} -follow -type f -not -path "*/TAXBRO/.snapshots/*"` to discover files.

3. Read {SOURCE_FOLDER}/CLAUDE.md to understand this filer's specific situation, applicable forms, and flagged issues.
   (Even if the knowledge graph exists, CLAUDE.md provides intent/context that the graph may not capture.)

4. Produce the checklist:

   When knowledge graph IS available — deep analysis mode:
   Read each section of US-knowledge-graph.md and assess:
   a. Income completeness — all expected W-2s, 1099s present and extracted?
   b. Cross-document checks — read the "Issues & Flags" section; surface all 🔴 critical and ⚠️ warning items
   c. Form readiness — for each applicable form, is the data sufficient to complete it?
   d. Missing or uncertain items — marked ✗ or ⚠️ in the graph?

   When knowledge graph is NOT available — structural mode:
   For each issue listed in CLAUDE.md (or the standard checklist below), check:
   a. Is the required source document present in the folder?
   b. Has TaxBro already generated an output file for it in TAXBRO/?
   c. Are there any open flags or risks noted?

5. Standard checklist items (adapt based on CLAUDE.md or knowledge graph):

   **Income & Withholding**
   - [ ] W-2(s) — all employers present; excess SS check done
   - [ ] 1099-B/DIV/INT — all brokerages present; wash sale review needed
   - [ ] Form 2210 Risk — check if 1040-ES payments (from US-knowledge-graph.md) were made timely (Apr/Jun/Sep/Jan) or if a penalty is likely due to Q4-only payments.
   - [ ] 1099-SA / 5498-SA — HSA distribution and contribution limit check

   **Deductions & Credits**
   - [ ] 1098(s) — mortgage interest; $750K acquisition debt limit check
   - [ ] Childcare — provider statements present; Form 2441 earned-income eligibility confirmed

   **Foreign Accounts & Assets**
   - [ ] Foreign accounts — all statements present; FBAR threshold check done; Form 8938 threshold check done
   - [ ] PFIC/Mutual funds — CAS/statements present; Form 8621 determination done
   - [ ] Foreign interest — certificates present; Form 1116 (passive basket) ready
   - [ ] Rental income — income documented; Schedule E amounts calculated
   - [ ] Foreign tax payments — advance payments documented; Form 1116 (general basket) ready
   - [ ] Schedule B Part III — foreign account disclosure ready

   **Canada / other**
   - [ ] Canada — T4 check done; confirm no residual withholding obligation

6. Format the output as a topic-by-topic review, organized by priority:

   **Priority 1 — Critical issues (🔴)**: Things that will cause a return to be wrong or incomplete.
   **Priority 2 — Action required (⚠️)**: Items needing more data, decisions, or calculations.
   **Priority 3 — On track (✅)**: Issues that are fully documented and analyzed.
   **Priority 4 — Not applicable**: Issues confirmed as not relevant for this filer.

   For each item, show:
   - Status icon (🔴 / ⚠️ / ✅ / —)
   - Issue name
   - Documents present: yes/no
   - Analysis done: yes/no (check if corresponding US-*.md output exists)
   - Key finding or flag (one line, no PII)
   - Suggested next action

7. Write the output to {SOURCE_FOLDER}/TAXBRO/US-checklist.md with today's date as header.

8. Summarize at the bottom:
   - How many items are 🔴 critical, ⚠️ need attention, ✅ clear, — not applicable
   - If knowledge graph was used: note when it was last built (check graph header)
   - Recommended next step (e.g., "Run /taxbro-generate-worksheets — all data present"
     or "Run /taxbro-extract first — graph is missing")

9. Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` recording: agent-id (e.g., `gemini-2.0-flash`), skill-name, status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

IMPORTANT: Never write any PII, account numbers, or financial figures to ~/claude/taxbro/.
All output goes to {SOURCE_FOLDER}/TAXBRO/US-checklist.md only.
