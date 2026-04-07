# TaxBro

A Claude Code project for analyzing US tax returns using source documents.
Point it at a folder of tax PDFs, and it reads, extracts, and summarizes everything you need.

## What it does

- Discovers and categorizes your tax documents automatically
- Checks FBAR / Form 8938 thresholds across all foreign accounts
- Analyzes PFIC holdings (Indian mutual funds) for Form 8621 requirements
- Consolidates foreign taxes paid for Form 1116
- Checks W-2s for excess Social Security withholding
- Summarizes childcare expenses and Form 2441 eligibility
- Converts foreign rental income to USD for Schedule E
- Runs a full checklist across all issues

All outputs are written to a `TAXBRO/` folder inside your tax documents directory — **never** inside this repo.

## Setup

1. Install [Claude Code](https://claude.ai/code)
2. Clone this repo:
   ```
   git clone https://github.com/yourusername/taxbro ~/claude/taxbro
   ```
3. Open a Claude Code session from this directory:
   ```
   cd ~/claude/taxbro
   claude
   ```
4. Point it at your tax documents:
   ```
   /taxbro-init /path/to/your/2025/tax/documents/
   ```

## Usage

```
/taxbro                       # Overview: all commands, session status, output files
/taxbro-init /path/to/taxes/  # Load source folder (run once per session)
/tax-checklist                # Full status check — start here
/check-w2s                    # W-2 analysis
/check-fbar                   # Foreign account balances + FBAR/8938 thresholds
/check-pfic                   # Indian mutual funds + Form 8621 determination
/foreign-tax-credit           # Consolidate foreign taxes → Form 1116
/check-childcare              # Childcare expenses + Form 2441 eligibility
/rental-income                # Foreign rental income → Schedule E
/generate-worksheets          # Pre-filled IRS form worksheets ready for preparer entry
/validate-return [draft.pdf]  # Cross-check completed return against source docs
```

Each skill can also accept the path directly:
```
/check-fbar /path/to/taxes/
```

## Source folder setup (recommended)

Add a `CLAUDE.md` to your tax documents folder describing your situation. This lets TaxBro personalize its analysis. See the template below.

<details>
<summary>CLAUDE.md template</summary>

```markdown
# Tax Return [YEAR] — Context

## Filers
- Primary: [Name] — [employment situation]
- Spouse: [Name] — [working / not working]
- Filing status: MFJ / Single / etc.

## Key situations
- [ ] Dual W-2 (job change)
- [ ] Two mortgages
- [ ] Childcare
- [ ] HSA
- [ ] Foreign accounts (list countries)
- [ ] Foreign rental property
- [ ] PFIC / foreign mutual funds
- [ ] Foreign tax credit
- [ ] Canada / other country history

## Foreign accounts
List each: institution, country, account type (NRE/NRO/savings/FD/brokerage)

## Notes
Any other flags or edge cases.
```
</details>

## Privacy

- This repo contains **zero personal data** — only skill prompts and instructions
- All analysis outputs go to `{your-tax-folder}/TAXBRO/` — keep that folder private
- `.current-session` (gitignored) stores only a folder path

## Supported forms

| Form | Skill |
|------|-------|
| Form 1040 | `/tax-checklist` |
| W-2 excess SS credit | `/check-w2s` |
| Schedule E (rental) | `/rental-income` |
| Form 2441 (childcare) | `/check-childcare` |
| FinCEN 114 (FBAR) | `/check-fbar` |
| Form 8938 (FATCA) | `/check-fbar` |
| Form 8621 (PFIC) | `/check-pfic` |
| Form 1116 (FTC) | `/foreign-tax-credit` |
| Schedule B Part III | `/check-fbar` |
| All forms (worksheets) | `/generate-worksheets` |
| Return validation | `/validate-return` |

## Extending

Add a new skill by creating `.claude/commands/your-skill.md`. Start with:
```
Get source folder from $ARGUMENTS or ~/claude/taxbro/.current-session.
[Your instructions]
Write output to {SOURCE_FOLDER}/TAXBRO/your-output.md.
Never write financial data to ~/claude/taxbro/.
```

## Disclaimer

TaxBro is an analysis aid, not tax advice. Always review outputs with a qualified tax preparer,
especially for complex situations (PFIC, FBAR, foreign tax credit, dual-country filings).
