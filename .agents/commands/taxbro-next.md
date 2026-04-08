# TaxBro Next — State-aware command orchestrator

Identify the current project state and execute (or show) the next logical tax analysis command.
This skill uses {SOURCE_FOLDER}/TAXBRO/agent-log.md as the primary ledger for cross-agent state.

Usage:
  /taxbro-next         — Identify and execute the next eligible command
  /taxbro-next --show  — Show the current state checklist and the next steps without executing

---

## Step 1 — State Discovery

1. Read `.current-session` for `SOURCE_FOLDER`.
2. If `SOURCE_FOLDER` is missing: **Next = /taxbro-init**. Stop and suggest init.
3. Read `{SOURCE_FOLDER}/TAXBRO/agent-log.md` if it exists.
4. Read `{SOURCE_FOLDER}/TAXBRO/session-notes.md` (from /taxbro-init).
5. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` (from /taxbro-extract).
6. Read `{SOURCE_FOLDER}/TAXBRO/US-checklist.md` (from /taxbro-checklist).
7. List `{SOURCE_FOLDER}/TAXBRO/` **with modification timestamps** to see all generated artifacts.
   Use: `ls -lt "{SOURCE_FOLDER}/TAXBRO/"` (or equivalent) to capture file modification times.

### Staleness Detection
If `US-knowledge-graph.md` was modified AFTER any downstream artifact (US-*-summary.md,
US-worksheets.md, US-checklist.md, US-knowledge-graph.html), that artifact is **stale** —
it was built from an older version of the knowledge graph.

Mark stale artifacts as `[!] STALE` in the status checklist instead of `[X]`.
When determining the next command, treat stale artifacts as if they don't exist
(i.e., re-run the corresponding skill).

---

## Step 2 — Build the Status Checklist (ASCII)

Generate an ASCII status table of the project lifecycle.
Status indicators:
[X] = Completed (artifact exists and log entry is 'complete')
[!] = Flagged / Attention Needed (critical issues in checklist or log entry 'partial')
[ ] = Pending / Eligible
[-] = Not Applicable (e.g., category not found in Knowledge Graph)
[?] = Unknown (needs /taxbro-extract to determine)

| Step | Command | Status | Artifact |
|:---|:---|:---:|:---|
| 1. Initialize | /taxbro-init | [ ] | session-notes.md |
| 2. Extract Data | /taxbro-extract | [ ] | US-knowledge-graph.md |
| 3. Status Review | /taxbro-checklist | [ ] | US-checklist.md |
| 4. Specific Analysis | | | |
|    - W-2 Wages | /taxbro-check-w2s | [ ] | US-w2-summary.md |
|    - FBAR/Foreign | /taxbro-check-fbar | [ ] | US-fbar-summary.md |
|    - PFIC/Funds | /taxbro-check-pfic | [ ] | US-pfic-summary.md |
|    - Foreign Tax Credit| /taxbro-foreign-tax-credit | [ ] | US-ftc-summary.md |
|    - Childcare | /taxbro-check-childcare | [ ] | US-childcare-summary.md |
|    - Rental Income | /taxbro-rental-income | [ ] | US-rental-income.md |
| 5. Worksheets | /taxbro-generate-worksheets| [ ] | US-worksheets.md |
| 6. Validation | /taxbro-validate-return | [ ] | US-validation-report.md |
| 7. Visualize | /taxbro-visualize | [ ] | US-knowledge-graph.html |

---

## Step 3 — Determine Next Eligible Command

Follow this priority list (from top to bottom):

1. **If no `TAXBRO/session-notes.md`:** Next = `/taxbro-init`
2. **If no `TAXBRO/US-knowledge-graph.md`:** Next = `/taxbro-extract`
3. **If no `TAXBRO/US-checklist.md`:** Next = `/taxbro-checklist`
4. **Check analysis skills:** For each analysis command in Step 2.4:
   - Check `US-knowledge-graph.md` (or `US-checklist.md`) to see if the category is applicable.
   - If applicable AND the artifact (e.g., `US-w2-summary.md`) is missing: Next = that specific command.
   - If multiple are missing, pick the first one listed.
5. **If all relevant analysis is done but no `US-worksheets.md`:** Next = `/taxbro-generate-worksheets`
6. **If worksheets exist but no `US-validation-report.md`:** Next = `/taxbro-validate-return`
7. **If all relevant analysis and worksheets exist but no `US-knowledge-graph.html`:** Next = `/taxbro-visualize`
8. **If everything is complete:** Tell the user the return is ready for final review and suggest opening the visualization dashboard.

---

## Step 4 — Execution

If `--show` is present:
1. Print the ASCII Checklist.
2. List the **"Recommended Next Action"**.
3. Stop.

If `--show` is NOT present:
1. Print the ASCII Checklist.
2. State the command being triggered: "Triggering next action: [COMMAND]".
3. **Execute the next command** by following its corresponding skill instructions in `.agents/commands/`.

---

## Step 5 — Finalize

After execution (or if just showing), append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md`:
```markdown
## {YYYY-MM-DD HH:MM} — {agent-id} — /taxbro-next
- **Status:** complete
- **Action:** {showed state | triggered [command]}
- **Next suggested:** {command}
```
