Show a full overview of the TaxBro project — what it is, all available commands, and current session status.

If $ARGUMENTS contains "--status", run STATUS MODE instead of the full overview (see below).

---

## STATUS MODE — /taxbro --status

Print a concise key:value block. Do this in under 15 lines. No tables, no prose, no headers.

Steps:
1. Read .current-session → SOURCE_FOLDER. If missing, print "session: none" and stop.
2. Check which output files exist in {SOURCE_FOLDER}/TAXBRO/ and note their mtime.
3. Scan {SOURCE_FOLDER}/TAXBRO/.snapshots/ — count snapshot directories.
4. Read the first few lines of {SOURCE_FOLDER}/TAXBRO/session-notes.md if it exists (for tax year / filer context).
5. Read {SOURCE_FOLDER}/CLAUDE.md or any subfolder CLAUDE.md to infer tax year and filing status (structural only, no PII).

Print exactly this format (omit any line where the value is unknown/not applicable):

```
session:      <last path component of SOURCE_FOLDER, e.g. "ALL-TAX-DOCS">
folder:       <SOURCE_FOLDER>
tax year:     <inferred year, e.g. "2025 (filed 2026)">
filing:       <e.g. "MFJ, sole earner, dual W-2">
cross-border: <e.g. "India (FBAR, PFIC, FTC), Canada (FBAR)"> or "none"
snapshots:    <N archived — run /taxbro-init --reset to create another>
outputs:
  checklist:      <exists | missing> [last updated: YYYY-MM-DD]
  w2:             <exists | missing> [last updated: YYYY-MM-DD]
  fbar:           <exists | missing> [last updated: YYYY-MM-DD]
  pfic:           <exists | missing> [last updated: YYYY-MM-DD]
  ftc:            <exists | missing> [last updated: YYYY-MM-DD]
  childcare:      <exists | missing> [last updated: YYYY-MM-DD]
  rental:         <exists | missing> [last updated: YYYY-MM-DD]
  worksheets:     <exists | missing> [last updated: YYYY-MM-DD]
  validation:     <exists | missing> [last updated: YYYY-MM-DD]
next:         <one-line suggestion: e.g. "run /taxbro-checklist to start" or "all outputs present — run /taxbro-validate-return">
```

Use `stat -f "%Sm" -t "%Y-%m-%d"` (macOS) to get mtime. If stat fails, omit the date.
Never print balances, names, SSNs, or any content from documents — structural metadata only.

---

## FULL OVERVIEW MODE

## What to display

### 1. Project description
TaxBro is a Claude Code assistant for preparing and validating **US Federal Tax Returns** (Form 1040 + international schedules). It reads PDFs and statements from a source folder, runs analysis against IRS rules, and writes summaries to {SOURCE_FOLDER}/TAXBRO/. No personal data ever touches the taxbro repo directory.

**Current scope: US returns only.** Foreign income, accounts, and taxes from any country are included only as they are reportable on a US return. All analysis output files are prefixed with `US-`.

### 2. Current session
Read .current-session.
- If the file exists: show the loaded source folder path and note the user can run /taxbro-checklist to see status.
- If the file does not exist: note that no source folder is loaded and prompt the user to run /taxbro-init.

### 3. Available commands
Display all commands in a table with command name and purpose:

| Command | Purpose |
|---------|---------|
| `/taxbro-init [path]` | Load source folder, read CLAUDE.md, discover documents |
| `/taxbro-checklist` | Full status check across all tax issues |
| `/taxbro-check-w2s` | W-2 analysis: excess SS, DCFSA, withholding check |
| `/taxbro-check-fbar` | Foreign accounts: max/year-end balances, FBAR/8938 thresholds |
| `/taxbro-check-pfic` | PFIC analysis: fund list, distributions, Form 8621 determination |
| `/taxbro-foreign-tax-credit` | Consolidate foreign taxes paid → Form 1116 |
| `/taxbro-check-childcare` | Childcare expenses + Form 2441 earned-income eligibility |
| `/taxbro-rental-income` | Foreign rental income → Schedule E |
| `/taxbro-generate-worksheets` | Pre-filled IRS form worksheets ready for preparer entry |
| `/taxbro-validate-return [pdf]` | Cross-check a completed/draft return PDF against all source documents |

### 4. Output files
All outputs are written to {SOURCE_FOLDER}/TAXBRO/. List the files that skill produces:

| File | Produced by |
|------|------------|
| `US-checklist.md` | /taxbro-checklist |
| `US-w2-summary.md` | /taxbro-check-w2s |
| `US-fbar-summary.md` | /taxbro-check-fbar |
| `US-pfic-summary.md` | /taxbro-check-pfic |
| `US-ftc-summary.md` | /taxbro-foreign-tax-credit |
| `US-childcare-summary.md` | /taxbro-check-childcare |
| `US-rental-income.md` | /taxbro-rental-income |
| `US-worksheets.md` | /taxbro-generate-worksheets |
| `US-validation-report.md` | /taxbro-validate-return |
| `session-notes.md` | /taxbro-init |

### 5. Privacy reminder
All analysis outputs go to {SOURCE_FOLDER}/TAXBRO/ — never inside ~/claude/taxbro/. The .current-session file contains only a path.
