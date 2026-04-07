Initialize TaxBro with a source tax documents folder.
Builds a full file index, creates per-year subfolders, and writes CLAUDE.md summary files at each level.
Supports --refresh to reindex without wiping existing analysis outputs.

Usage:
  /taxbro-init /path/to/tax/folder/           — full initialization
  /taxbro-init /path/to/tax/folder/ --refresh — reindex files, update knowledge graph, preserve outputs
  /taxbro-init --refresh                      — refresh using path from current session

---

## Parse arguments

Split $ARGUMENTS on whitespace:
- SOURCE_PATH: first non-flag token (path argument)
- REFRESH_MODE: true if "--refresh" is present

If no SOURCE_PATH and REFRESH_MODE is false: ask user for folder path and stop.
If no SOURCE_PATH and REFRESH_MODE is true: read path from ~/claude/taxbro/.current-session and proceed.

---

## Step 1 — Verify and resolve the folder path

1. Verify the folder exists. If not, tell user and stop.
2. Run `realpath "$SOURCE_PATH"` to canonicalize symlinks.
   If the folder is a symlink: note "folder is a symlink → resolves to [real path]"
3. Write the RESOLVED path to ~/claude/taxbro/.current-session (single line, no trailing newline).
4. SOURCE_FOLDER = resolved path. Use this everywhere below.

---

## Step 2 — Read filer context

Read {SOURCE_FOLDER}/CLAUDE.md if it exists. Extract (no PII):
- Filing status (MFJ / Single / etc.)
- Expected document types per year
- Tax years mentioned or implied
- Known edge cases

---

## Step 3 — Transitive file discovery

Run: `find {SOURCE_FOLDER} -follow -type f -not -path "*/TAXBRO/*"`

For each file collect:
- Path relative to SOURCE_FOLDER
- Size: `stat -f%z` (macOS) / `stat -c%s` (Linux)
- Modified: `stat -f%Sm -t "%Y-%m-%d %H:%M"` (macOS) / `stat -c%y` (Linux)
- Extension

Special cases:
- Files ending in `.alias`: collect separately, warn (Finder aliases can't be followed)
- Symlink files: categorize by target name/extension

Do NOT traverse TAXBRO/ — that is output, not source.

---

## Step 4 — Categorize every file and infer tax year

**Category from filename patterns:**
| Category | Signals |
|----------|---------|
| W-2 | w2, w-2, wage |
| 1099-B | 1099b, 1099-b, brokerage, proceeds |
| 1099-DIV | 1099div, 1099-div, dividend |
| 1099-INT | 1099int, 1099-int, interest |
| 1099-SA/5498-SA | 1099sa, 5498sa, hsa |
| 1098 | 1098, mortgage |
| Childcare | childcare, daycare, receipt |
| Foreign bank | hdfc, icici, sbi, axis, kotak, nre, nro, fd, fixed.deposit |
| Interest certificate | interest.cert, tds.cert, form16a, 16a |
| Mutual fund / PFIC | cams, karvy, cas, mutual.fund, mf, elss, sip |
| AIS / TIS | ais, tis, annual.information |
| Tax payment | challan, advance.tax, tds |
| Return / draft | return, 1040, draft, final |
| Other | (unmatched) |

**Tax year inference:** extract 4-digit years (2020–2030) from filename or path segments.
If a file has no year signal, assign it to UNKNOWN_YEAR.

Collect the set of all distinct tax years found: YEARS_DETECTED.

---

## Step 5 — Diff against existing file-index.md

If {SOURCE_FOLDER}/TAXBRO/file-index.md exists:
- Parse it to get previously recorded files (relative path + size + modified)
- Compute: new files, removed files, changed files (size or mtime differs), unchanged files

In REFRESH_MODE: report the diff prominently.
In full init: note diff silently in the index.

---

## Step 6 — Create TAXBRO folder structure

Create these directories (skip if already exist):
- {SOURCE_FOLDER}/TAXBRO/
- {SOURCE_FOLDER}/TAXBRO/{year}/ for each year in YEARS_DETECTED

If YEARS_DETECTED is empty (no year signals found): create {SOURCE_FOLDER}/TAXBRO/unknown/ and treat as the active year folder.

---

## Step 7 — Write file-index.md (top level)

Write to {SOURCE_FOLDER}/TAXBRO/file-index.md:

```
# TaxBro File Index
Last updated: {DATE}
Mode: {full init | refresh}
Source: {SOURCE_FOLDER}
Tax years detected: {comma-separated list}

## Summary
- Total files: N
- By year: 2025 (N), 2024 (N), unknown (N)
- By category: W-2 (N), 1099s (N), Foreign accounts (N), Mutual funds (N), ...
- New since last index: N
- Removed since last index: N
- Changed since last index: N
- Finder aliases (unreadable): N

## Filer context (structural — no PII)
{One paragraph from CLAUDE.md: filing status, expected forms, edge cases, years covered}

## Knowledge Graph — {year} (one section per year, most recent first)

### {year}
| File | Category | Inferred type | Size | Modified | Status |
|------|----------|--------------|------|----------|--------|
| path/file.pdf | W-2 | Form W-2 | 245 KB | 2026-01-15 | new/unchanged/changed |
...

### Unresolvable Finder Aliases
| File | Note |
(if any)

## Gap analysis
For each year, list expected document types from CLAUDE.md with no matching file:
"⚠️ {year} — Missing: [document type]"

## Change log
| Date | Mode | Added | Removed | Changed |
|------|------|-------|---------|---------|
| {DATE} | {mode} | N | N | N |
(append each run — never overwrite prior rows)
```

IMPORTANT: Structural metadata only — no balances, names, SSNs, or document contents.

---

## Step 8 — Write per-year CLAUDE.md

For each year in YEARS_DETECTED, write {SOURCE_FOLDER}/TAXBRO/{year}/CLAUDE.md:

```markdown
# TaxBro — {year}
Source: {SOURCE_FOLDER}
Created: {DATE}
Tax year: {year}

## Filer context
{Filer summary from CLAUDE.md, scoped to this year if multi-year context available}

## Source documents ({year})
{N} files — see [../file-index.md](../file-index.md) for full listing.

Key documents found for {year}:
- W-2s: {N} — {filenames if any}
- 1099s: {N} — {filenames if any}
- Foreign accounts: {N} — {filenames if any}
- Mutual fund statements: {N} — {filenames if any}
- Other: {N}

## Analysis outputs
| Output | File | Status | Last updated |
|--------|------|--------|-------------|
| Full checklist | [checklist.md](checklist.md) | ⏳ not run | — |
| W-2 summary | [w2-summary.md](w2-summary.md) | ⏳ not run | — |
| FBAR / Form 8938 | [fbar-summary.md](fbar-summary.md) | ⏳ not run | — |
| PFIC / Form 8621 | [pfic-summary.md](pfic-summary.md) | ⏳ not run | — |
| Foreign tax credit | [ftc-summary.md](ftc-summary.md) | ⏳ not run | — |
| Childcare / Form 2441 | [childcare-summary.md](childcare-summary.md) | ⏳ not run | — |
| Rental income / Sched E | [rental-income.md](rental-income.md) | ⏳ not run | — |
| IRS worksheets | [worksheets.md](worksheets.md) | ⏳ not run | — |
| Return validation | [validation-report.md](validation-report.md) | ⏳ not run | — |

## Gap analysis
{Paste any ⚠️ missing document warnings for this year from file-index.md}

## Notes
(Skills append notes here as they run)
```

In REFRESH_MODE: update file counts and gap analysis but preserve existing status rows (keep ✅ complete entries intact).

---

## Step 9 — Write top-level TAXBRO/CLAUDE.md

Write {SOURCE_FOLDER}/TAXBRO/CLAUDE.md:

```markdown
# TaxBro — All Years
Source: {SOURCE_FOLDER}
Last updated: {DATE}

## Tax years
| Year | Source files | Analysis | Folder |
|------|-------------|----------|--------|
| {year} | N files | N/9 complete | [{year}/]({year}/CLAUDE.md) |
(one row per year, most recent first)

## Quick links
- [File index & knowledge graph](file-index.md)
- [Session notes](session-notes.md)
{for each year: - [{year} analysis]({year}/CLAUDE.md)}

## Filer overview
{Brief summary from CLAUDE.md: who this is, what years are covered, key complexity flags}
```

In REFRESH_MODE: update file counts and "N/9 complete" counts (scan each year folder for existing output files) but preserve all other content.

---

## Step 10 — Append to session-notes.md

Append (do not overwrite) to {SOURCE_FOLDER}/TAXBRO/session-notes.md:
- Date/time, mode (init or refresh)
- Original path → resolved path (if different)
- Years detected: {list}
- File counts: total, new, changed, removed
- Finder alias count (if any)
- Year folders created or updated

No financial data, names, or document contents.

---

## Step 11 — Report to user

Full init:
```
TaxBro initialized. {SOURCE_FOLDER}
Tax years: {year list}
Files: {N} total across {K} categories
Year folders created: TAXBRO/2025/, TAXBRO/2024/, ...
{Gap warnings if any}

Run /taxbro-checklist to see full status for {most recent year}.
To analyze a specific year: /taxbro-checklist 2024
```

Refresh:
```
TaxBro refreshed. {SOURCE_FOLDER}
Files: {N} total ({new} new, {changed} changed, {removed} removed)
Year folders: {list with file counts}
{Gap warnings if any}
```

If Finder aliases found: "⚠️ N Finder alias(es) detected — cannot be read. Convert with 'ln -s /real/path link.pdf' or copy files directly."

---

IMPORTANT: Never write PII, financial data, or document contents to ~/claude/taxbro/ or any file within it.
file-index.md and CLAUDE.md files contain structural metadata only.
