Generate an interactive HTML visualization of the TaxBro Knowledge Graph.

Usage:
  /taxbro-visualize          — generate or refresh the HTML dashboard
  /taxbro-visualize --open   — generate and tell user to open the file

Output: {SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.html
Opens in any browser. Self-contained — no server required.

⚠️ NEVER DELETE ANY FILE. Only write to TAXBRO/US-knowledge-graph.html.

---

## Steps

1. Read `.current-session` → SOURCE_FOLDER.
2. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` — primary data source.
3. Read `{SOURCE_FOLDER}/TAXBRO/file-index.md` if it exists.
4. **Read `{SOURCE_FOLDER}/TAXBRO/US-validation-report.md` if it exists.**
   This file contains return-specific figures (filed values per form/line) and validation
   check results (PASS/FLAG/MISMATCH). Use it to populate the "Return Δ" tab.
5. Extract the following structured data from the knowledge graph:
   - Issues & Flags table (severity, area, issue, detail)
   - Identity fields (tax year, filing status, filers, residency)
   - W-2 wages table
   - Interest income (US + India)
   - 1099-B summary (proceeds, gain/loss)
   - HSA summary
   - IRA distribution
   - Foreign accounts table (institution, type, owner, max balance USD, Dec 31 USD)
   - PFIC fund table (fund name, AMC, units, est. value USD, redemptions, dividends)
   - FTC summary (basket, income, taxes)
   - Prior year carryforwards
   - Source document registry (file, category, status)
6. Extract the following from US-validation-report.md (if present):
   - Filed return name, preparer, date
   - Validation results table: form/line, return value, source value, status (PASS/FLAG/MISMATCH)
   - Flagged items detail
   - Items to confirm with preparer
7. Build a `return_vs_source` array pairing source-document values with filed return values.
   For each row include: form, description, source_label, ret_label, source (number), ret (number|null),
   delta_na (bool — true when delta is not meaningful e.g. gross vs. taxable), status, note.
8. Write a self-contained HTML file to `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.html`.

9. Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` recording: agent-id (e.g., `gemini-2.0-flash`), skill-name, status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

---

## HTML Design Requirements

The dashboard should be a single self-contained HTML file (no external files needed):
- Dark-mode design (#0f172a background, slate color palette)
- Uses libraries loaded from CDN (vis-network for graph, Chart.js for charts, Alpine.js for interactivity)
- Gracefully degrades if offline (CDN fails) — all data still visible as tables

### Layout: Sidebar + Main panel

**Left sidebar (fixed, ~240px):**
- App title "TaxBro" with tax year badge
- Navigation links: Overview, Issues, Return Δ, Documents, Income, Accounts, PFIC, Graph
- Issue count badges by severity (🔴 N critical, ⚠️ N attention)
- Return validation summary: N pass / N flag / N mismatch
- "Generated: YYYY-MM-DD" footer

**Main content area (scrollable, tabbed):**

#### Tab 1 — Overview
- Summary cards row: Total Income (W-2 + interest + CG), FTC Claimed, Balance Due, PFIC Gap
- Cross-border exposure cards: India (FBAR ✓, PFIC ⚠️, FTC, NRE gap), Canada (FBAR ✓, FTC ✓)
- Top issues (first 3–6 critical flags in red cards)
- Output files status grid (knowledge graph, validation, checklist, FBAR, PFIC, etc.)

#### Tab 2 — Issues & Flags
- Full flags table, filterable by severity
- Each row: severity badge, area tag, issue summary, detail (expandable)
- Color coding: 🔴 = red bg, ⚠️ = amber bg, ℹ️ = blue bg

#### Tab 3 — Return Δ (NEW — requires US-validation-report.md)
- Header: return filename, preparer, date validated
- Summary row: N ✅ PASS / N ⚠️ FLAG / N ❌ MISMATCH
- Full delta table with columns: Form/Line | Source Value | Return Value | Δ | Status | Note (expandable)
  - Green row: PASS
  - Amber row: FLAG (difference worth investigating)
  - Red row: MISMATCH (clear discrepancy)
  - Gray: SKIP (no data for comparison)
- Delta column: signed dollar amount (+$X / -$X); "—" when delta_na = true; "Not Filed" when ret = null
- "Items to confirm with preparer" callout section at bottom
- If US-validation-report.md is absent: show a message prompting user to run /taxbro-validate-return

#### Tab 4 — Documents
- Searchable/filterable document registry table
- Columns: File, Category, Status (✓/✗/⚠️), Notes
- Status colors: green = extracted, red = not read, amber = flagged
- Click row to expand notes

#### Tab 5 — Income
- W-2 wages card (employer, wages, withholding) — show return value inline where available
- Interest income breakdown (US + India, side by side) — show return line 2b alongside sources
- Capital gains summary (ST/LT, proceeds/basis/net) — show return line 7
- 1099-DIV summary
- HSA and IRA rows — note if form absent from return

#### Tab 6 — Accounts (FBAR/8938)
- Foreign accounts table: institution, type, owner, max bal, Dec 31 bal (USD)
- FBAR threshold indicator (aggregate max vs $10K threshold)
- Form 8938 threshold indicator (year-end vs $100K/$200K)
- Visual: horizontal bar chart of account max balances

#### Tab 7 — PFIC
- Fund table: fund name, AMC, owner, units, Dec 31 value, redemptions, dividends
- Aggregate value vs $25K threshold (progress bar)
- Per-fund threshold check
- Action required: Form 8621 count

#### Tab 8 — Graph (Network)
- vis-network graph showing document → tax topic relationships
- Nodes: source documents (color by status), tax topics/forms (color by flag status)
- Edges: document contributes to → form/topic
- Click a node to highlight connected nodes and show info panel
- Legend: document types, form types, flag status

Node types and colors:
- Source document (read ✓): #166534 (green-800)
- Source document (flagged ⚠️): #92400e (amber-800)
- Source document (not read ✗): #475569 (slate-600)
- US form / schedule: #1e40af (blue-800)
- Foreign form: #581c87 (purple-900)
- Missing form (❌): #7f1d1d (red-900)

Edge example connections to build:
- W-2 → Form 1040 Line 1a
- T4 → Form 1116 General
- Fidelity 1099 → Schedule B, Schedule D, Form 8949
- Marcus 1099-INT → Schedule B
- ICICI NRE stmt → Schedule B (NRE interest ⚠️ MISSING)
- ICICI NRO stmt → Schedule B, Form 1116 Passive
- SBI NRO stmt → Schedule B, Form 1116 Passive
- Scotiabank stmt → FBAR
- AIS (Karthik) → Schedule B cross-check
- AIS (Vinaya) → Schedule B cross-check, FBAR
- CAS / MF stmt → Form 8621 (MISSING ⚠️)
- Canada return + NOA → Form 1116 General
- India computation → Schedule B (NRE interest gap ⚠️)
- 1099-SA → Form 8889 (MISSING ⚠️)
- 1099-R → Form 8606
- Cost sheets → Schedule E depreciation

---

## Code Quality Requirements

- All data is embedded as a JSON object (`TAX_DATA`) in a `<script>` block at the top
- No external file dependencies (CSS and JS from CDN only)
- Works when opened via file:// protocol (no CORS issues)
- Responsive layout (works on MacBook screen)
- Each tab content is rendered from the embedded JSON — not hardcoded HTML strings
- Include a "Copy data as JSON" button for debugging
- Footer: "Generated by TaxBro on {date} | /taxbro-visualize"

---

## Privacy Note
The HTML file will contain financial figures extracted from the knowledge graph and validation
report. It is written to the source folder (not the taxbro repo) and should be treated with the
same care as other tax documents. Never commit this file to git.
