# TaxBro — Gemini CLI Configuration

## Core Mandates & Data Safety (CRITICAL)

**These rules are non-negotiable and override all other tool behavior:**

1. **NO PII IN APP DIRECTORY:** Never write personal data into `~/claude/taxbro/` (no names, SSNs, account numbers, balances, income figures, or addresses).
2. **ISOLATED OUTPUT:** All analysis outputs (summaries, checklists, extracted tables) MUST go to `{SOURCE_FOLDER}/TAXBRO/` only.
3. **SOURCE FOLDER CONTRACT:** The source folder is established via the `.current-session` file. Always read this file to determine where to find documents and where to write output.
4. **NO SENSITIVE COMMITS:** Never stage or commit any files within a `{SOURCE_FOLDER}/TAXBRO/` directory.

## Workflow Overview

TaxBro is a tool for analyzing US tax returns using source documents. It uses a "Research -> Strategy -> Execution" lifecycle to verify tax data.

### 1. Initialization
The command `/tax-init [path]` sets the active source folder by writing its absolute path to `.current-session`. 

### 2. Document Discovery
When pointed at a source folder, TaxBro expects:
- A `CLAUDE.md` or `GEMINI.md` within the source folder describing the filer's situation.
- Tax documents (PDFs, statements, spreadsheets).

### 3. Output Schema
Every skill writes its output to `{SOURCE_FOLDER}/TAXBRO/` using the following file naming convention:
- `checklist.md`: Overall status
- `fbar-summary.md`: Foreign account balances
- `pfic-summary.md`: Form 8621 determinations
- `ftc-summary.md`: Foreign tax credit data
- `w2-summary.md`: W-2 analysis
- `childcare-summary.md`: Childcare expenses
- `rental-income.md`: Rental income summaries
- `session-notes.md`: Miscellaneous notes

## Skill Implementation for Gemini

To implement a new skill for Gemini CLI, use the following pattern:
1. **Read `.current-session`** to get the `SOURCE_FOLDER`.
2. **Search `SOURCE_FOLDER`** for relevant documents (e.g., PDFs with "W-2" in the name).
3. **Analyze** the content according to IRS guidelines (e.g., Publication 54 for foreign income).
4. **Write results** to `{SOURCE_FOLDER}/TAXBRO/{skill-name}-summary.md`.
