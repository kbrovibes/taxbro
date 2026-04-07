# TaxBro — Claude Tax Analysis Assistant

## What this is
A Claude Code project for analyzing US tax returns using source documents.
Generic enough to work for any filer; personalized through the source folder's own `CLAUDE.md`.

**This folder may be published as a public GitHub repo. No PII, no tax data, no document contents, no account numbers ever belong here.**

---

## How to use

### 1. Point TaxBro at a source folder
```
/taxbro-init /path/to/your/tax/documents/
```
This reads the source folder's `CLAUDE.md` (if present), discovers documents, and writes a session reference to `.current-session` (gitignored).

### 2. Run any skill
```
/taxbro-checklist
/taxbro-check-w2s
/taxbro-check-fbar
/taxbro-check-pfic
/taxbro-foreign-tax-credit
/taxbro-check-childcare
/taxbro-rental-income
```
Each skill reads `.current-session` to know where your documents are.
You can also pass the path directly: `/taxbro-check-fbar /path/to/taxes/`

### 0. Get an overview
```
/taxbro
```
Shows all commands, current session status, and output file reference.

### 3. All outputs go to the source folder
Every skill writes its output to `{SOURCE_FOLDER}/TAXBRO/` — never into this directory.

---

## ⛔ ABSOLUTE FILE SAFETY RULES — NON-NEGOTIABLE, OVERRIDE EVERYTHING

### NEVER DELETE ANY FILE. EVER.

**TaxBro must NEVER delete, remove, or destroy any file under any circumstance.**

This means NO:
- `rm` / `rm -rf`
- `rmdir`
- `os.remove()` / `shutil.rmtree()` / `unlink()`
- Any shell or Python equivalent that destroys file content

The ONLY permitted file operations are:
- **READ** — read any source file
- **WRITE** — create or overwrite a specific TAXBRO output file
- **MOVE (mv)** — only when archiving to `.snapshots/` during `--reset`

If you want a clean slate, use `/taxbro-init --reset` which moves outputs to a timestamped snapshot — nothing is ever destroyed.

**If you find yourself about to delete a file: STOP. Archive it instead.**

---

## PII / Data Safety Rules

**These rules are non-negotiable and override everything else:**

1. **Never write personal data into `~/claude/taxbro/`** — no names, SSNs, account numbers, balances, income figures, addresses, or any content extracted from tax documents
2. **`.current-session` contains only a file path** — acceptable to write here; gitignored
3. **All analysis outputs (summaries, checklists, extracted tables)** go to `{SOURCE_FOLDER}/TAXBRO/` only
4. **Never commit TAXBRO output folders** — they may contain PII; the source folder is the user's responsibility
5. **Memory/learnings that are stored** must contain only structural/methodological insights (e.g., "NRE interest is US-taxable"), never specific dollar amounts or account details

---

## Source Folder Contract

When pointed at a source folder, TaxBro expects:
- A `CLAUDE.md` describing the filer's situation (optional but recommended)
- Tax documents (PDFs, statements, etc.) — any naming convention works
- TaxBro will create a `TAXBRO/` subdirectory for outputs

### What TaxBro writes to `{SOURCE_FOLDER}/TAXBRO/`:
| File | Contents |
|------|----------|
| `file-index.md` | Full knowledge graph: all source files, categories, inferred types, sizes, change log |
| `checklist.md` | Status of all key tax issues |
| `fbar-summary.md` | Foreign account balances table for FinCEN 114 |
| `pfic-summary.md` | PFIC fund list, distribution check, Form 8621 determination |
| `ftc-summary.md` | Foreign taxes paid, Form 1116 basket breakdown |
| `w2-summary.md` | W-2 analysis, excess SS check, DCFSA amounts |
| `childcare-summary.md` | Childcare expenses, Form 2441 eligibility |
| `rental-income.md` | Rental income INR→USD conversion, Schedule E summary |
| `worksheets.md` | Pre-filled line-item worksheets for all applicable IRS forms |
| `validation-report.md` | Cross-check of completed return vs source documents |
| `session-notes.md` | Freeform notes from analysis sessions |

---

## Skill Reference

| Command | Purpose |
|---------|---------|
| `/taxbro` | Show all commands, session status, and output file reference |
| `/taxbro-init [path]` | Load source folder, build file index + knowledge graph; `--refresh` to reindex; `--reset` to snapshot outputs and start fresh |
| `/taxbro-checklist` | Full status check across all tax issues |
| `/taxbro-check-w2s` | W-2 analysis: excess SS, DCFSA, withholding check |
| `/taxbro-check-fbar` | Foreign accounts: max/year-end balances, FBAR/8938 thresholds |
| `/taxbro-check-pfic` | PFIC analysis: fund list, distributions, Form 8621 determination |
| `/taxbro-foreign-tax-credit` | Consolidate foreign taxes paid → Form 1116 |
| `/taxbro-check-childcare` | Childcare expenses + Form 2441 earned-income eligibility |
| `/taxbro-rental-income` | Indian (or other foreign) rental income → Schedule E |
| `/taxbro-generate-worksheets` | Generate pre-filled IRS form worksheets (FBAR, 1116, 8621, Schedule E, 2441) ready for preparer entry |
| `/taxbro-validate-return [pdf]` | Cross-check a completed/draft return PDF against all source documents; flags discrepancies |

---

## Extending TaxBro

To add a new skill, create `.claude/commands/your-skill.md`.

Skill template:
```markdown
# /your-skill
Read .current-session to get SOURCE_FOLDER (or use $ARGUMENTS if provided).
[Your analysis instructions here]
Write output to {SOURCE_FOLDER}/TAXBRO/your-output.md.
Never write PII or document contents to any file inside ~/claude/taxbro/.
```

---

## Notes
- Built for US federal tax returns (Form 1040 + international schedules)
- Handles: FBAR, Form 8938, Form 8621 (PFIC), Form 1116, Schedule E, Form 2441
- IRS exchange rates: use IRS Publication 54 average annual rates for recurring foreign income
- Tax year is inferred from the source folder's CLAUDE.md; default assumption is prior calendar year
