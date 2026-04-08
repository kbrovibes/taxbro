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
- Professional dark-mode design (#0f172a background, slate/blue/green palette)
- Uses libraries loaded from CDN (vis-network for graph, Chart.js for charts, Alpine.js for interactivity)
- **High-Signal UI:** Prioritize the "bottom line" (Refund vs. Owe) and "Action Required" items.

### Layout: Sidebar + Main panel

**Left sidebar (fixed, ~240px):**
- App title "TaxBro" with tax year badge
- Navigation links: 🎯 Summary, ✅ Checklist, 💰 Income & Tax, 🌍 Global Assets, 📐 Return Δ, 📄 Documents
- **Outcome Badge:** Large "EST. REFUND" or "EST. OWE" amount shown prominently in the sidebar
- Issue count badges by severity (🔴 N critical, ⚠️ N attention)
- "Generated: YYYY-MM-DD" footer

**Main content area (scrollable, tabbed):**

#### Tab 1 — 🎯 Summary (Condensed)
- **Bottom Line Card:** Big, bold calculation: (Payments + Credits) - (Estimated Tax) = Refund/Owe.
- **Top 3 Actions:** The most critical red-flag items from the knowledge graph.
- **Exposure Map:** Simple cards for India/Canada/US highlighting if FBAR/FATCA/FTC apply.
- **Status Progress:** A progress bar showing % of tax documents processed and validated.

#### Tab 2 — ✅ Checklist
- **Command Status:** An interactive version of the /taxbro-next checklist.
- [X] Extract | [X] W-2s | [X] FBAR | [ ] Validation | etc.
- **Open Issues:** Grouped list of all 🔴 and ⚠️ flags with "Fix/Verify" instructions.

#### Tab 3 — 💰 Income & Tax
- **Income Breakdown:** Doughnut chart of income sources.
- **Tax vs. Payments:** Side-by-side bar chart showing (W-2 Withholding + Estimates) vs. (Ordinary Tax + NIIT + Addl Medicare Tax).
- **Credits & Deductions:** List of key tax savers (Excess SS, FTC, Itemized Deductions).

#### Tab 4 — 🌍 Global Assets (FBAR/PFIC)
- **FBAR Table:** Simplified with "Action Needed" column.
- **PFIC Status:** Aggregate value vs $50K threshold (Progress bar).
- **Threshold Alerts:** Clear indicators for Form 8938 and FBAR filing requirements.

#### Tab 5 — 📐 Return Δ (Validation)
- Header: Return filename and verification status.
- **Delta Table:** Condensed view of Form/Line | Source | Return | Δ.
- **Color-coded rows:** PASS (Green), FLAG (Amber), MISMATCH (Red).
- **Preparer Follow-up:** List of items that need manual verification by the CPA.

#### Tab 6 — 📄 Documents
- Searchable registry of all 70+ documents.
- Status indicator for each (Read/Flagged/Skipped).

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
