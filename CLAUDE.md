# TaxBro — Claude Tax Analysis Assistant

## What this is
A Claude Code project for **preparing and validating US Federal Tax Returns** (Form 1040 + international schedules).
Generic enough to work for any filer; personalized through the source folder's own `CLAUDE.md`.

**Current scope: US returns only.** Foreign income, accounts, and taxes from any country are used only as inputs to US reporting requirements (Schedule E, Form 1116, FBAR, Form 8938, Form 8621, etc.). All analysis output files are prefixed with `US-`.

**This folder may be published as a public GitHub repo. No PII, no tax data, no document contents, no account numbers ever belong here.**

---

## How to use

### 1. Point TaxBro at a source folder
```
/taxbro-init /path/to/your/tax/documents/
```

### 2. Extract and analyze everything in one pass
```
/taxbro-extract
```
This reads every document, builds the knowledge graph with full analysis and computed totals.

### 3. Visualize
```
/taxbro-visualize
```
Opens an interactive HTML dashboard in your browser.

### 4. Deep dives and filing
```
/taxbro-compliance          — FBAR/PFIC/FTC deep dive (if flagged)
/taxbro-worksheets          — preparer-ready IRS form line items
/taxbro-validate-return     — cross-check a draft return PDF
```

All outputs go to `{SOURCE_FOLDER}/TAXBRO/` — never into this directory.

---

## ABSOLUTE FILE SAFETY RULES — NON-NEGOTIABLE

### NEVER DELETE ANY FILE. EVER.

**TaxBro must NEVER delete, remove, or destroy any file under any circumstance.**

The ONLY permitted file operations are:
- **READ** — read any source file
- **WRITE** — create or overwrite a specific TAXBRO output file
- **MOVE (mv)** — only when archiving to `.snapshots/` during `--reset`

**If you find yourself about to delete a file: STOP. Archive it instead.**

---

## PII / Data Safety Rules

**These rules are non-negotiable and override everything else:**

1. **Never write personal data into `~/claude/taxbro/`** — no names, SSNs, account numbers, balances, income figures, addresses, or any content extracted from tax documents
2. **`.current-session` contains only a file path** — acceptable to write here; gitignored
3. **All analysis outputs** go to `{SOURCE_FOLDER}/TAXBRO/` only
4. **Never commit TAXBRO output folders** — they may contain PII
5. **Memory/learnings that are stored** must contain only structural/methodological insights, never dollar amounts or account details

---

## Source Folder Contract

When pointed at a source folder, TaxBro expects:
- A `CLAUDE.md` describing the filer's situation (optional but recommended)
- Tax documents (PDFs, statements, etc.) — any naming convention works
- TaxBro will create a `TAXBRO/` subdirectory for outputs

### Output files (all in `{SOURCE_FOLDER}/TAXBRO/`):
| File | Produced by |
|------|-------------|
| `US-knowledge-graph.md` | `/taxbro-extract` — all facts, cross-checks, flags, computed totals |
| `US-knowledge-graph.html` | `/taxbro-visualize` — interactive dashboard |
| `US-compliance-summary.md` | `/taxbro-compliance` — FBAR, PFIC, FTC deep dive |
| `US-worksheets.md` | `/taxbro-worksheets` — preparer-ready IRS form line items |
| `US-validation-report.md` | `/taxbro-validate-return` — cross-check of return vs sources |
| `session-notes.md` | `/taxbro-init` — document inventory and session log |

---

## Skill Reference

| Command | Purpose |
|---------|---------|
| `/taxbro-init [path]` | Load source folder, discover documents; `--reset` to snapshot and start fresh |
| `/taxbro-extract` | **Read all documents → build knowledge graph with full analysis** (run this first) |
| `/taxbro-visualize` | Interactive HTML dashboard — tax waterfall, per-item impact, threshold bars |
| `/taxbro-compliance` | Deep dive: FBAR + Form 8938 + PFIC/Form 8621 + FTC/Form 1116 |
| `/taxbro-worksheets` | Preparer-ready IRS form line items (reads computed totals from knowledge graph) |
| `/taxbro-validate-return [pdf]` | Cross-check a completed/draft return PDF against source documents |

Archived skills (in `.agents/commands/archive/`) are available for reference but no longer part of the standard workflow.

---

## Extending TaxBro

To add a new skill, create `.agents/commands/your-skill.md` (Claude reads via `.claude/commands/` symlink).

---

## Notes
- Built for US federal tax returns (Form 1040 + international schedules)
- Handles: FBAR, Form 8938, Form 8621 (PFIC), Form 1116, Schedule E, Form 2441, Form 2210
- IRS exchange rates: use IRS Publication 54 average annual rates for recurring foreign income
- Tax year is inferred from the source folder's CLAUDE.md; default assumption is prior calendar year
- DCFSA exclusion (IRC §129): if spouse has $0 earned income, DCFSA amount added back to income
