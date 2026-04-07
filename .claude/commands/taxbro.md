Show a full overview of the TaxBro project — what it is, all available commands, and current session status.

## What to display

### 1. Project description
TaxBro is a Claude Code assistant for analyzing US tax returns using source documents. It reads PDFs and statements from a source folder, runs analysis against IRS rules, and writes summaries to {SOURCE_FOLDER}/TAXBRO/. No personal data ever touches the taxbro repo directory.

### 2. Current session
Read ~/claude/taxbro/.current-session.
- If the file exists: show the loaded source folder path and note the user can run /tax-checklist to see status.
- If the file does not exist: note that no source folder is loaded and prompt the user to run /taxbro-init.

### 3. Available commands
Display all commands in a table with command name and purpose:

| Command | Purpose |
|---------|---------|
| `/taxbro-init [path]` | Load source folder, read CLAUDE.md, discover documents |
| `/tax-checklist` | Full status check across all tax issues |
| `/check-w2s` | W-2 analysis: excess SS, DCFSA, withholding check |
| `/check-fbar` | Foreign accounts: max/year-end balances, FBAR/8938 thresholds |
| `/check-pfic` | PFIC analysis: fund list, distributions, Form 8621 determination |
| `/foreign-tax-credit` | Consolidate foreign taxes paid → Form 1116 |
| `/check-childcare` | Childcare expenses + Form 2441 earned-income eligibility |
| `/rental-income` | Foreign rental income → Schedule E |
| `/generate-worksheets` | Pre-filled IRS form worksheets ready for preparer entry |
| `/validate-return [pdf]` | Cross-check a completed/draft return PDF against all source documents |

### 4. Output files
All outputs are written to {SOURCE_FOLDER}/TAXBRO/. List the files that skill produces:

| File | Produced by |
|------|------------|
| `checklist.md` | /tax-checklist |
| `w2-summary.md` | /check-w2s |
| `fbar-summary.md` | /check-fbar |
| `pfic-summary.md` | /check-pfic |
| `ftc-summary.md` | /foreign-tax-credit |
| `childcare-summary.md` | /check-childcare |
| `rental-income.md` | /rental-income |
| `worksheets.md` | /generate-worksheets |
| `validation-report.md` | /validate-return |
| `session-notes.md` | /taxbro-init |

### 5. Privacy reminder
All analysis outputs go to {SOURCE_FOLDER}/TAXBRO/ — never inside ~/claude/taxbro/. The .current-session file contains only a path.
