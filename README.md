# TaxBro

A Claude Code assistant for analyzing and preparing US federal tax returns.

Point it at a folder of tax documents — W-2s, 1099s, foreign bank statements, mutual fund CAS reports, childcare receipts — and TaxBro reads everything in one pass, builds a structured knowledge graph with computed totals, and produces an interactive dashboard. Use the compliance and worksheet skills when you need deep dives or preparer-ready output.

---

## Quick start

```
1. Clone + open Claude Code

   git clone https://github.com/kbrovibes/taxbro ~/claude/taxbro
   cd ~/claude/taxbro && claude

2. Point at your tax folder

   /taxbro-init ~/taxes/2025/

3. Extract and analyze everything

   /taxbro-extract

4. See the results

   /taxbro-visualize              --> open the HTML dashboard

5. When you need more

   /taxbro-compliance             --> FBAR/PFIC/FTC deep dive
   /taxbro-worksheets             --> preparer-ready form line items

6. When your CPA sends a draft

   /taxbro-validate-return ~/taxes/2025/draft-return.pdf
```

---

## How it works

TaxBro reads every source document once and builds a structured knowledge graph. The graph includes extracted facts, cross-document checks, and computed totals (income, AGI, estimated tax, refund/owe). All downstream skills read from the graph — no raw documents re-read.

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
|  /taxbro-extract                                              |
|                                                               |
|  Reads every document once. Extracts facts, not content.      |
|  Runs 15 cross-document checks. Computes totals.              |
|                                                               |
|              +--------------------------------------+         |
|              |   US-knowledge-graph.md              |         |
|              |                                      |         |
|              |   Issues & Flags                     |         |
|              |   Income (W-2 / 1099 / HSA)          |         |
|              |   Foreign Income + Rental            |         |
|              |   Deductions + Childcare             |         |
|              |   Foreign Accounts (FBAR)            |         |
|              |   PFIC / Mutual Funds                |         |
|              |   Credits (FTC, Excess SS)            |         |
|              |   Payments + Estimated Tax            |         |
|              |   Computed Totals (AGI -> Refund)     |         |
|              +--------------------------------------+         |
+---------------------------------------------------------------+
                              |
                              v
+---------------------------------------------------------------+
|  DOWNSTREAM SKILLS (all read from knowledge graph)            |
|                                                               |
|  /taxbro-visualize     --> interactive HTML dashboard         |
|  /taxbro-compliance    --> FBAR + PFIC + FTC deep dive        |
|  /taxbro-worksheets    --> preparer-ready IRS form line items |
|  /taxbro-validate-return --> cross-check draft return PDF     |
+---------------------------------------------------------------+
                              |
                              v
+---------------------------------------------------------------+
|  OUTPUTS -- all in {YOUR_TAX_FOLDER}/TAXBRO/                  |
|                                                               |
|  US-knowledge-graph.md      US-knowledge-graph.html           |
|  US-compliance-summary.md   US-worksheets.md                  |
|  US-validation-report.md    session-notes.md                  |
|                                                               |
|  [!] Keep this folder private -- it contains financial data   |
|  [*] Nothing sensitive ever touches the TaxBro repo itself    |
+---------------------------------------------------------------+
```

---

## The Knowledge Graph

`US-knowledge-graph.md` is TaxBro's central data structure. It has three design principles:

**Extract facts, not content.** A bank statement might be 40 pages. The graph stores three numbers: max balance, Dec 31 balance, and interest earned.

**Link, don't embed.** Every value includes a source document citation. Skills that need more detail can go read the file.

**Compute once, use everywhere.** The "Computed Totals" section pre-computes income, AGI, taxable income, estimated tax, and refund/owe. All downstream skills read these totals instead of independently recomputing — this prevents agent drift when different models (Claude vs Gemini) interpret the same data differently.

The graph uses confidence markers:

| Marker | Meaning |
|--------|---------|
| `✓` | Confirmed — read directly from a source document |
| `~` | Calculated or converted — derived from confirmed figures |
| `⚠️` | Uncertain or needs verification |
| `✗` | Missing — expected but not found |

---

## All commands

| Command | Purpose |
|---------|---------|
| `/taxbro-init [path]` | Load source folder, discover documents |
| `/taxbro-init --reset` | Archive existing outputs to snapshot, start fresh |
| `/taxbro-extract` | **Read all documents, build knowledge graph with full analysis** |
| `/taxbro-visualize` | Interactive HTML dashboard — tax waterfall, per-item impact, threshold bars |
| `/taxbro-compliance` | Deep dive: FBAR + Form 8938 + PFIC/Form 8621 + FTC/Form 1116 |
| `/taxbro-worksheets` | Preparer-ready IRS form line items |
| `/taxbro-validate-return [pdf]` | Cross-check a completed/draft return PDF against source docs |

---

## Output files

All outputs go to `{SOURCE_FOLDER}/TAXBRO/`. Keep this folder private.

| File | Produced by | Contents |
|------|-------------|---------|
| `session-notes.md` | `/taxbro-init` | Document inventory, initialization log |
| `US-knowledge-graph.md` | `/taxbro-extract` | All facts, cross-checks, flags, computed totals |
| `US-knowledge-graph.html` | `/taxbro-visualize` | Interactive dashboard |
| `US-compliance-summary.md` | `/taxbro-compliance` | FBAR/PFIC/FTC filing determinations and detail |
| `US-worksheets.md` | `/taxbro-worksheets` | Pre-filled IRS form line items |
| `US-validation-report.md` | `/taxbro-validate-return` | Line-by-line cross-check results |

---

## Multi-agent support

TaxBro works with both Claude Code and Gemini CLI:

| Skill | Recommended agent | Why |
|-------|-------------------|-----|
| `/taxbro-extract` | **Gemini** | 1M+ context window for 30-40 PDFs |
| `/taxbro-visualize` | **Claude** | Complex self-contained HTML generation |
| `/taxbro-validate-return` | **Claude** | Nuanced cross-checking |
| All others | Either | |

Skills live in `.agents/commands/`. Claude reads via `.claude/commands/` (symlink). Gemini reads directly.

---

## Supported forms

| Form | Covered by |
|------|-----------|
| Form 1040 (key lines) | `/taxbro-extract` (Computed Totals), `/taxbro-worksheets` |
| W-2 excess SS credit | `/taxbro-extract` (cross-checks) |
| Schedule B (interest, Part III) | `/taxbro-extract`, `/taxbro-compliance` |
| Schedule D / Form 8949 | `/taxbro-extract` (RSU basis adjustment) |
| Schedule E (foreign rental) | `/taxbro-extract` |
| Form 1098 (mortgage) | `/taxbro-extract` (TCJA cap analysis) |
| Form 2441 (childcare) | `/taxbro-extract` (DCFSA/IRC 129 analysis) |
| Form 2210 (underpayment) | `/taxbro-extract` (quarterly attribution) |
| FinCEN 114 (FBAR) | `/taxbro-compliance` |
| Form 8938 (FATCA) | `/taxbro-compliance` |
| Form 8621 (PFIC) | `/taxbro-compliance` |
| Form 1116 (FTC) | `/taxbro-compliance` |
| Form 8889 (HSA) | `/taxbro-extract` |

---

## Privacy and data safety

- **This repo contains zero personal data** — only skill prompts and instructions
- **All outputs go to `{SOURCE_FOLDER}/TAXBRO/`** — your folder, your control
- **`.current-session`** contains only a path — it is gitignored
- **Never commit your `TAXBRO/` folder** — it contains financial data

---

## Disclaimer

TaxBro is an analysis aid, not tax advice. Always review outputs with a qualified tax preparer, especially for PFIC/Form 8621, FBAR, foreign tax credit calculations, and excess distribution determinations. IRS rules and thresholds change annually — verify for your tax year.
