Show a full overview of the TaxBro project — what it is, all available commands, and current session status.

## What to display

### 1. Project description
TaxBro is a Claude Code assistant for analyzing US tax returns using source documents. It reads PDFs and statements from a source folder, runs analysis against IRS rules, and writes summaries organized by tax year into {SOURCE_FOLDER}/TAXBRO/{year}/. No personal data ever touches the TaxBro repo directory.

### 2. Current session
Read ~/claude/taxbro/.current-session.
- If the file exists: show the loaded source folder path.
  - Read {SOURCE_FOLDER}/TAXBRO/CLAUDE.md if it exists and show the years table (tax years, file counts, analysis status).
  - Tell user they can run /taxbro-checklist to see full status for the most recent year.
- If not: note no folder is loaded, prompt to run /taxbro-init.

### 3. Available commands
| Command | Purpose |
|---------|---------|
| `/taxbro-init [path]` | Initialize: build file index, create year folders, write CLAUDE.md summaries |
| `/taxbro-init [path] --refresh` | Reindex files, update knowledge graph, preserve existing outputs |
| `/taxbro-checklist [year]` | Full status check for a tax year |
| `/taxbro-check-w2s [year]` | W-2 analysis: excess SS, DCFSA, withholding check |
| `/taxbro-check-fbar [year]` | Foreign accounts: max/year-end balances, FBAR/8938 thresholds |
| `/taxbro-check-pfic [year]` | PFIC analysis: fund list, distributions, Form 8621 determination |
| `/taxbro-foreign-tax-credit [year]` | Consolidate foreign taxes paid → Form 1116 |
| `/taxbro-check-childcare [year]` | Childcare expenses + Form 2441 earned-income eligibility |
| `/taxbro-rental-income [year]` | Foreign rental income → Schedule E |
| `/taxbro-generate-worksheets [year]` | Pre-filled IRS form worksheets ready for preparer entry |
| `/taxbro-validate-return [year] [pdf]` | Cross-check a completed/draft return PDF against source documents |

All year arguments are optional — skills default to the most recent year in the file index.

### 4. Output structure
All outputs are written to {SOURCE_FOLDER}/TAXBRO/{year}/ — never inside the TaxBro repo.

```
{SOURCE_FOLDER}/TAXBRO/
├── CLAUDE.md              ← all-years index with status overview
├── file-index.md          ← full knowledge graph of all source files
├── session-notes.md       ← initialization and refresh log
├── 2025/
│   ├── CLAUDE.md          ← year summary with links to all outputs
│   ├── checklist.md
│   ├── w2-summary.md
│   ├── fbar-summary.md
│   ├── pfic-summary.md
│   ├── ftc-summary.md
│   ├── childcare-summary.md
│   ├── rental-income.md
│   ├── worksheets.md
│   └── validation-report.md
└── 2024/
    ├── CLAUDE.md
    └── ...
```

### 5. Privacy reminder
All analysis outputs go to {SOURCE_FOLDER}/TAXBRO/ — never inside the TaxBro repo directory.
The .current-session file contains only a folder path. CLAUDE.md files contain only structural metadata.
