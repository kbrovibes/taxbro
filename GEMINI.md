# TaxBro — Gemini CLI Configuration

## At the Start of Every Session

1. Read `.current-session` → `SOURCE_FOLDER` path.
2. Tell the user: "Session loaded. Source: {folder}."

---

## COMPUTATION RULES — Read Before Every Skill

### Rule 1: Use Adjusted Values, Never Face Values
- **Capital gains**: Use the **adjusted** 1099-B gain (after RSU basis correction), NOT the face value.
- **Interest income**: Total = US interest (1099-INT) + India NRE interest (USD) + India NRO interest (USD).
- **Rental income**: Use the **net** figure (after depreciation and PAL carryforward), not gross rent.
- **DCFSA**: If spouse earned income = $0 and no exception, DCFSA exclusion = $0 — add Box 10 back to income (IRC §129).

### Rule 2: Read "Computed Totals" First
If `US-knowledge-graph.md` has a "Computed Totals" section, use those numbers as-is.
Do NOT recompute from raw sections.

### Rule 3: Show Your Math
When writing calculated amounts, show the formula inline.

### Rule 4: Cross-Check Before Writing
Before writing output, verify your totals match the Computed Totals section.

---

## Invoking Skills

When the user types `/skill-name` (e.g. `/taxbro-extract`):

1. Read `.agents/commands/skill-name.md` in full.
2. Execute all steps exactly as written. Do not summarize or skip steps.

**Available skills:**

| Skill | File | Best agent |
|-------|------|------------|
| `/taxbro-init` | `.agents/commands/taxbro-init.md` | Either |
| `/taxbro-extract` | `.agents/commands/taxbro-extract.md` | **Gemini** (large context, 30-40 PDFs) |
| `/taxbro-visualize` | `.agents/commands/taxbro-visualize.md` | **Claude** (complex HTML generation) |
| `/taxbro-compliance` | `.agents/commands/taxbro-compliance.md` | Either |
| `/taxbro-worksheets` | `.agents/commands/taxbro-worksheets.md` | Either |
| `/taxbro-validate-return` | `.agents/commands/taxbro-validate-return.md` | **Claude** (nuanced cross-checking) |

---

## Core Data Safety Rules (Non-Negotiable)

1. **NO PII IN APP DIRECTORY:** Never write personal data into `~/claude/taxbro*/`.
2. **ISOLATED OUTPUT:** All analysis outputs go to `{SOURCE_FOLDER}/TAXBRO/` only.
3. **SOURCE FOLDER CONTRACT:** Always read `.current-session` for `SOURCE_FOLDER`.
4. **NO DELETIONS EVER:** Never run `rm`, `rmdir`, or any equivalent.

---

*Skills live in `.agents/commands/`. Claude reads via `.claude/commands/` (symlink). Gemini reads directly.*
