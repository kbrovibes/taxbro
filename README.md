# TaxBro

A Claude Code assistant for analyzing and preparing US federal tax returns.

Point it at a folder of tax documents — W-2s, 1099s, foreign bank statements, mutual fund CAS reports, childcare receipts — and TaxBro reads everything once, builds a structured knowledge graph, and lets you interrogate, validate, and produce worksheets for your preparer without ever touching the raw documents again.

---

## Quick start

```
 YOU                                    TAXBRO
 ───                                    ──────

 1.  Clone + open Claude Code

     git clone https://github.com/kbrovibes/taxbro ~/claude/taxbro
     cd ~/claude/taxbro && claude

 2.  Point at your tax folder

     /taxbro-init ~/taxes/2025/

 3.  Let the orchestrator drive

     /taxbro-next                       --> extracts all documents
     /taxbro-next                       --> builds checklist
     /taxbro-next                       --> runs W-2 analysis
     /taxbro-next                       --> runs FBAR check
     /taxbro-next                       --> runs PFIC analysis
     /taxbro-next                       --> runs foreign tax credit
     /taxbro-next                       --> runs childcare check
     /taxbro-next                       --> runs rental income
     /taxbro-next                       --> generates worksheets
       ...keep running until it says "all done"

 4.  Hand worksheets to your preparer

     All outputs are in ~/taxes/2025/TAXBRO/

 5.  When the draft return comes back

     /taxbro-validate-return ~/taxes/2025/draft-return.pdf
```

You can also run any skill directly (e.g. `/taxbro-check-fbar`) if you want to skip ahead or re-run a specific analysis. `/taxbro-next` just picks the next one that hasn't been done yet.

---

## How it works

TaxBro is a set of **Claude Code slash commands** (skills) organized around a two-phase architecture:

```
+---------------------------------------------------------------+
|                       YOUR TAX FOLDER                         |
|                                                               |
|  W-2s  1099s  1098s  Bank Statements  CAMS CAS  AIS/TIS      |
|  Childcare Receipts  Interest Certs  Advance Tax Challans     |
+-----------------------------+---------------------------------+
                              |
                   /taxbro-init (discover)
                              |
                              v
+---------------------------------------------------------------+
|  PHASE 1 -- EXTRACTION              /taxbro-extract           |
|                                                               |
|  Reads every source document exactly once.                    |
|  Extracts only what matters -- facts, not content.            |
|                                                               |
|  Per W-2:   wages, withholding, SS tax, Box 12 codes          |
|  Per bank:  max balance + Dec 31 balance (not transactions)   |
|  Per CAMS:  year-end NAV snapshot + any redemptions           |
|  Per AIS:   income category totals (not individual entries)   |
|  Per 1099:  totals only (not individual trades)               |
|  ...and 13 cross-document checks (excess SS, FBAR threshold, |
|     PFIC de minimis, AIS reconciliation, Form 2441, etc.)    |
|                                                               |
|                              v                                |
|           +--------------------------------------+            |
|           |   US-knowledge-graph.md              |            |
|           |                                      |            |
|           |   Issues & Flags                     |            |
|           |   Identity                           |            |
|           |   Income (W-2 / 1099 / HSA)          |            |
|           |   Foreign Income                     |            |
|           |   Deductions                         |            |
|           |   Foreign Accounts (FBAR)            |            |
|           |   PFIC / Mutual Funds                |            |
|           |   Credits (Form 1116)                |            |
|           |   Payments                           |            |
|           |   Source Document Registry            |            |
|           +--------------------------------------+            |
+---------------------------------------------------------------+
                              |
                              |  (read once, reuse many times)
                              |
+---------------------------------------------------------------+
|  PHASE 2 -- ANALYSIS                                         |
|                                                               |
|  /taxbro-next               -> orchestrator (start here)     |
|  /taxbro-checklist          -> topic review + priority flags  |
|  /taxbro-check-w2s          -> excess SS, DCFSA, withholding |
|  /taxbro-check-fbar         -> FBAR/8938 thresholds + table  |
|  /taxbro-check-pfic         -> Form 8621 per fund            |
|  /taxbro-foreign-tax-credit -> Form 1116 basket breakdown    |
|  /taxbro-check-childcare    -> Form 2441 earned income test  |
|  /taxbro-rental-income      -> Schedule E + INR->USD         |
|  /taxbro-generate-worksheets-> pre-filled IRS form line items|
|  /taxbro-validate-return    -> cross-check draft return PDF  |
|                                                               |
|  Each skill reads US-knowledge-graph.md first.                |
|  No raw documents re-read unless a fact is missing.           |
+---------------------------------------------------------------+
                              |
                              v
+---------------------------------------------------------------+
|  OUTPUTS -- all written to {YOUR_TAX_FOLDER}/TAXBRO/          |
|                                                               |
|  US-knowledge-graph.md       US-checklist.md                  |
|  US-w2-summary.md            US-fbar-summary.md               |
|  US-pfic-summary.md          US-ftc-summary.md                |
|  US-childcare-summary.md     US-rental-income.md              |
|  US-worksheets.md            US-validation-report.md          |
|                                                               |
|  [!] Keep this folder private -- it contains financial data   |
|  [*] Nothing sensitive ever touches the TaxBro repo itself    |
+---------------------------------------------------------------+
```

---

## The Knowledge Graph

`US-knowledge-graph.md` is TaxBro's central data structure. Every analysis skill reads from it rather than re-reading source documents. It has two key design principles:

**Extract facts, not content.** A bank statement might be 40 pages of transactions. The graph stores three numbers: maximum balance during the year, December 31 balance, and total interest earned. Everything else is discarded. This keeps the graph compact (a few hundred lines) while preserving everything needed for FBAR thresholds, Schedule B, and Form 1116.

**Link, don't embed.** When a fact is too complex to summarize, the graph records the source document path and the relevant section. Skills that need more detail can go read it — but most don't need to.

A few things the graph enables that would otherwise require re-reading every document each time:

- **Excess Social Security**: computed once across all W-2s, stored as a single flag. No per-W-2 re-read needed.
- **FBAR threshold check**: all foreign account maximums summed in one place. The threshold question is answered at extraction time.
- **Form 2441 earned income test**: the spouse's earned income (or the student/disability exception) is recorded once and referenced by `/taxbro-check-childcare` directly.
- **AIS reconciliation**: income from Indian AIS statements is cross-checked against what bank statements show as deposits, flagged if inconsistent.
- **PFIC de minimis**: all PFIC values summed and compared against the $25K per-fund and $50K aggregate thresholds at extraction time.

The graph uses confidence markers on every value:

| Marker | Meaning |
|--------|---------|
| `✓` | Confirmed — read directly from a source document |
| `~` | Calculated or converted — derived from confirmed figures |
| `?` | Uncertain or needs verification |
| `✗` | Missing — expected but not found |

---

## Workflow

### Standard workflow

```
/taxbro-init /path/to/your/tax/folder/
/taxbro-next
```

`/taxbro-next` is the status-aware orchestrator. Run it after initialization to see exactly where you are and what to do next. It builds an ASCII checklist and can automatically trigger the next required skill.

After `/taxbro-checklist` shows you what needs attention, work through the topics. The checklist output is prioritized — critical items first, then items needing attention, then clear items.

If you need deeper analysis on a specific area:
```
/taxbro-check-fbar
/taxbro-check-pfic
/taxbro-check-childcare
```

When everything looks good:
```
/taxbro-generate-worksheets
```

Once your preparer sends a draft:
```
/taxbro-validate-return /path/to/draft-return.pdf
```

### Starting fresh

If you want to start over (new documents added, prior run was incomplete):
```
/taxbro-init --reset
/taxbro-extract
```

`--reset` archives your existing TAXBRO/ outputs to a timestamped snapshot before reinitializing. Nothing is ever deleted — snapshots are preserved indefinitely.

### Check status at any time
```
/taxbro --status
```

Prints a compact key:value summary: which folder is loaded, which outputs exist (with dates), and what to do next.

---

## Install

```bash
git clone https://github.com/kbrovibes/taxbro ~/claude/taxbro
cd ~/claude/taxbro
claude
```

Claude Code auto-discovers `.claude/commands/` and registers all skills.

**Prerequisites:** [Claude Code](https://claude.ai/code) (the CLI) + a folder of tax documents.

---

## Personalizing with CLAUDE.md

Add a `CLAUDE.md` to your tax documents folder. Every skill reads it before running. The more context you provide, the more targeted the analysis — especially for missing document detection and checklist prioritization.

```markdown
# Tax Return 2025 — Context

## Filers
- Primary: [Name] — W-2, two employers (job change)
- Spouse: [Name] — not working (Form 2441 student exception applies)
- Filing status: MFJ

## Key situations
- [x] Two W-2s (job change mid-year)
- [x] Two mortgage properties (1098 from each)
- [x] Childcare (1 child, 2 providers)
- [x] HSA (employee + employer contributions)
- [x] Foreign accounts: India (ICICI NRE, SBI NRO), Canada (Scotiabank)
- [x] Indian mutual funds — CAMS CAS on file (PFIC risk)
- [x] Indian rental property (50% ownership)
- [x] Indian bank interest + TDS (Form 1116 passive basket)
- [x] India advance tax payments (Form 1116 general basket)
- [ ] Canadian T4 (confirmed: no employment income in Canada 2025)

## Notes
- Prior year QEF elections: none — all PFICs on default excess distribution method
- Rental depreciation basis established in 2021 return (30-year foreign residential)
```

---

## All commands

| Command | Purpose |
|---------|---------|
| `/taxbro` | Overview: commands, session status, output file reference |
| `/taxbro --status` | Compact key:value status of current session |
| `/taxbro-init [path]` | Load source folder, discover documents |
| `/taxbro-init --reset` | Archive existing outputs to snapshot, start fresh |
| `/taxbro-next` | **Identify and execute the next eligible command (orchestrator)** |
| `/taxbro-extract` | **Read all documents → build US-knowledge-graph.md** (run before checklist) |
| `/taxbro-checklist` | Full status check and topic-by-topic review (reads knowledge graph) |
| `/taxbro-check-w2s` | W-2 analysis: excess SS, DCFSA, withholding |
| `/taxbro-check-fbar` | Foreign accounts: FBAR / Form 8938 thresholds |
| `/taxbro-check-pfic` | PFIC: Form 8621 determination per fund |
| `/taxbro-foreign-tax-credit` | Foreign taxes by basket → Form 1116 |
| `/taxbro-check-childcare` | Childcare: Form 2441 earned income test |
| `/taxbro-rental-income` | Foreign rental income → Schedule E |
| `/taxbro-generate-worksheets` | Pre-filled line items for all applicable IRS forms |
| `/taxbro-validate-return [pdf]` | Cross-check draft/final return PDF against source docs |

---

## Output files

All outputs go to `{SOURCE_FOLDER}/TAXBRO/`. Keep this folder private.

| File | Produced by | Contents |
|------|-------------|---------|
| `session-notes.md` | `/taxbro-init` | Document counts, missing doc flags, initialization log |
| `US-knowledge-graph.md` | `/taxbro-extract` | All extracted facts, cross-checks, flags |
| `US-checklist.md` | `/taxbro-checklist` | Prioritized topic review with status and next actions |
| `US-w2-summary.md` | `/taxbro-check-w2s` | W-2 table, excess SS, DCFSA/HSA flags |
| `US-fbar-summary.md` | `/taxbro-check-fbar` | Account balances, FBAR/8938 determinations |
| `US-pfic-summary.md` | `/taxbro-check-pfic` | Fund table, de minimis check, Form 8621 per fund |
| `US-ftc-summary.md` | `/taxbro-foreign-tax-credit` | Foreign taxes by basket, FTC limitation note |
| `US-childcare-summary.md` | `/taxbro-check-childcare` | Provider table, DCFSA offset, Form 2441 eligibility |
| `US-rental-income.md` | `/taxbro-rental-income` | Monthly rent, USD conversion, Schedule E summary |
| `US-worksheets.md` | `/taxbro-generate-worksheets` | Pre-filled line items for all applicable forms |
| `US-validation-report.md` | `/taxbro-validate-return` | Line-by-line cross-check results |

---

## Supported forms

| Form | Handled by |
|------|-----------|
| Form 1040 (key lines) | `/taxbro-generate-worksheets` |
| W-2 excess SS credit (Schedule 3) | `/taxbro-check-w2s` |
| Schedule B (interest, dividends, Part III) | `/taxbro-extract`, `/taxbro-validate-return` |
| Schedule E (foreign rental) | `/taxbro-rental-income` |
| Schedule D / Form 8949 | `/taxbro-checklist` (presence check) |
| Form 1098 (mortgage interest) | `/taxbro-extract`, `/taxbro-checklist` |
| Form 2441 (childcare credit) | `/taxbro-check-childcare` |
| FinCEN 114 (FBAR) | `/taxbro-check-fbar` |
| Form 8938 (FATCA) | `/taxbro-check-fbar` |
| Form 8621 (PFIC) | `/taxbro-check-pfic` |
| Form 1116 (foreign tax credit) | `/taxbro-foreign-tax-credit` |
| Form 8889 (HSA) | `/taxbro-extract`, `/taxbro-checklist` |

---

## Privacy and data safety

- **This repo contains zero personal data** — only skill prompts and instructions
- **All outputs go to `{SOURCE_FOLDER}/TAXBRO/`** — your folder, your control
- **`.current-session`** contains only a path — it is gitignored
- **Never commit your `TAXBRO/` folder** — it contains financial data
- **Nothing in TaxBro's memory** ever contains dollar amounts, names, or account details — only methodological notes like "NRE interest is US-taxable"

---

## Disclaimer

TaxBro is an analysis aid, not tax advice. Always review outputs with a qualified tax preparer, especially for PFIC/Form 8621, FBAR, foreign tax credit calculations, and excess distribution determinations. IRS rules and thresholds change annually — verify for your tax year.
