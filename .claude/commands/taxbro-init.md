Initialize TaxBro with a source tax documents folder, building a full file index and knowledge graph.
Supports --refresh to reindex without full re-initialization.

Usage:
  /taxbro-init /path/to/tax/folder/           — full initialization
  /taxbro-init /path/to/tax/folder/ --refresh — reindex files, update knowledge graph, preserve session

---

## Parse arguments

Split $ARGUMENTS on whitespace. Extract:
- SOURCE_PATH: the path argument (first non-flag token)
- REFRESH_MODE: true if "--refresh" is present, false otherwise

If no SOURCE_PATH and no .current-session exists: ask the user for the folder path and stop.
If no SOURCE_PATH but .current-session exists and REFRESH_MODE is true: read path from .current-session and proceed.

---

## Step 1 — Verify and resolve the folder path

1. Verify the folder exists. If it doesn't, tell the user and stop.
2. Resolve symlinks: run `realpath "$SOURCE_PATH"` (or the python3 equivalent) to get the canonical path.
   - If the folder itself is a symlink, note: "Note: folder is a symlink → resolves to [real path]"
3. Write the RESOLVED absolute path to ~/claude/taxbro-2/.current-session (single line, no trailing newline).
4. Set SOURCE_FOLDER = resolved path. All subsequent steps use this.

---

## Step 2 — Read filer context

Read {SOURCE_FOLDER}/CLAUDE.md if it exists.
Extract (without recording PII):
- Filing status (MFJ/Single/etc.)
- Expected document types (which W-2s, which foreign accounts, which forms)
- Known edge cases or flags
- Tax year (if stated; otherwise infer from folder name or document names)

This context guides categorization and gap detection below.

---

## Step 3 — Build the transitive file listing

Run a full recursive, symlink-aware file discovery:

```
find {SOURCE_FOLDER} -follow -type f -not -path "*/TAXBRO/*"
```

For each file, collect:
- Relative path (from SOURCE_FOLDER)
- File size in bytes: `stat -f%z` (macOS) or `stat -c%s` (Linux)
- Last modified timestamp: `stat -f%Sm -t "%Y-%m-%d %H:%M"` (macOS) or `stat -c%y` (Linux)
- File extension

Handle special cases:
- Files ending in `.alias` or matching macOS Finder alias patterns: collect separately, warn the user
- Symlink files: use the target filename/extension for categorization, note if symlink name differs

Do NOT traverse into {SOURCE_FOLDER}/TAXBRO/ — that is TaxBro's output directory, not source documents.

---

## Step 4 — Categorize and build the knowledge graph

For each discovered file, infer as much as possible from the filename, path, and extension:

**Category assignment** (use filename patterns, not content):
| Category | Filename signals |
|----------|-----------------|
| W-2 | w2, w-2, wage |
| 1099-B | 1099b, 1099-b, brokerage, proceeds |
| 1099-DIV | 1099div, 1099-div, dividend |
| 1099-INT | 1099int, 1099-int, interest |
| 1099-SA/5498-SA | 1099sa, 5498sa, hsa |
| 1098 | 1098, mortgage |
| Childcare | childcare, daycare, receipt |
| Foreign bank statement | hdfc, icici, sbi, axis, kotak, nre, nro, savings, fd, fixed.deposit |
| Interest certificate | interest.cert, tds.cert, form16a, 16a |
| Mutual fund / PFIC | cams, karvy, cas, mutual.fund, mf, elss, sip |
| AIS / TIS (India) | ais, tis, annual.information |
| Tax payment | challan, advance.tax, tds |
| Return / draft | return, 1040, draft, final |
| Other | (anything not matched above) |

**Year inference**: extract 4-digit years (2023, 2024, 2025, etc.) from filename or path segments.

**Knowledge graph entry per file**:
- Relative path
- Category
- Inferred document type (e.g., "W-2, employer: Acme Corp" if discernible from filename)
- Inferred tax year (if detectable)
- File size
- Last modified

---

## Step 5 — Diff against existing index (both modes)

Check if {SOURCE_FOLDER}/TAXBRO/file-index.md exists.

**If no existing index** (first run or index was deleted):
- All files are "new". Proceed to Step 6 to write a fresh index.

**If existing index exists**:
- Parse the existing index to extract the previously recorded files (path + size + modified).
- Compare against the current file listing:
  - **New files**: present in current listing, absent from index
  - **Removed files**: present in index, absent from current listing
  - **Changed files**: present in both, but size or modified timestamp differs
  - **Unchanged files**: present in both, same size and modified timestamp

In REFRESH_MODE: report the diff to the user prominently.
In full init mode: note any diff silently in the index (handles re-initialization on an existing folder).

---

## Step 6 — Write the file index

Create/update {SOURCE_FOLDER}/TAXBRO/file-index.md:

```
# TaxBro File Index
Last updated: {DATE}
Mode: {full init | refresh}
Source folder: {SOURCE_FOLDER}
Tax year: {inferred year or "unknown"}

## Summary
- Total files discovered: N
- By category: W-2 (N), 1099s (N), Foreign accounts (N), Mutual funds (N), ...
- New since last index: N  (0 if first run)
- Removed since last index: N
- Changed since last index: N
- Unreadable Finder aliases: N (if any)

## Filer context (structural only — no PII)
{One paragraph from CLAUDE.md analysis: filing status, expected forms, edge cases, tax year}

## Knowledge Graph

### {Category}
For each file in category:
| File | Inferred type | Year | Size | Modified | Status |
|------|--------------|------|------|----------|--------|
| relative/path/file.pdf | W-2, Employer: (from filename if readable) | 2025 | 245 KB | 2026-01-15 | new / unchanged / changed |

... repeat for each category ...

### Unresolvable Finder Aliases
| File | Note |
(if any)

## Gap analysis
List expected document types from CLAUDE.md that have NO matching file in the index.
Format: "⚠️ Missing: [document type] — expected based on CLAUDE.md"

## Change log
| Date | Mode | Files added | Files removed | Files changed |
|------|------|-------------|---------------|---------------|
| {DATE} | {mode} | N | N | N |
(append a new row each time — do not overwrite prior rows)
```

IMPORTANT: The index must contain ONLY structural metadata (paths, sizes, dates, inferred types from filenames).
Never write account numbers, balances, names, SSNs, or any content extracted from the documents themselves.

---

## Step 7 — Create TAXBRO/ and update session notes

Create {SOURCE_FOLDER}/TAXBRO/ if it doesn't exist.

Append to {SOURCE_FOLDER}/TAXBRO/session-notes.md (do not overwrite):
- Date and time
- Mode: full init or refresh
- Original path and resolved path (if different)
- File counts: total, new, changed, removed
- Finder alias count (if any)
- Whether gap analysis found missing documents

Do NOT write any financial data, document contents, names, or account numbers here.

---

## Step 8 — Report to user

In REFRESH_MODE:
```
TaxBro refreshed. {SOURCE_FOLDER}
  Files: {total} total ({new} new, {changed} changed, {removed} removed)
  {Gap warnings if any}
  Index updated: {SOURCE_FOLDER}/TAXBRO/file-index.md
```

In full init mode:
```
TaxBro initialized. {SOURCE_FOLDER}
  Files discovered: {N} across {K} categories
  {Gap warnings if any}
  Run /taxbro-checklist to see full status.
  Index: {SOURCE_FOLDER}/TAXBRO/file-index.md
```

If Finder aliases were found, include:
"⚠️ Found N Finder alias file(s) — TaxBro cannot read these. Convert with 'ln -s /real/path ~/taxes/link.pdf' or copy the files directly."

---

IMPORTANT: Never write any PII, financial data, or document contents to ~/claude/taxbro-2/ or any file within it.
The file-index.md lives in {SOURCE_FOLDER}/TAXBRO/ and contains only structural metadata — never document contents.
