# TaxBro — Gemini CLI Configuration

## At the Start of Every Session

1. Read `.current-session` → `SOURCE_FOLDER` path.
2. If `{SOURCE_FOLDER}/TAXBRO/agent-log.md` exists, read it and summarize prior work in one sentence.
3. Tell the user: "Session loaded. Source: {folder}. Prior work: {summary or 'none'}."

---

## Invoking Skills

When the user types `/skill-name` (e.g. `/taxbro-extract`):

1. Read `.agents/commands/skill-name.md` in full.
2. Execute all steps in that file exactly as written. Do not summarize or skip steps.
3. At the end of each skill, append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` (format below).

**Available skills:**

| Skill | File | Best agent |
|-------|------|------------|
| `/taxbro` | `.agents/commands/taxbro.md` | Either |
| `/taxbro-init` | `.agents/commands/taxbro-init.md` | Either |
| `/taxbro-extract` | `.agents/commands/taxbro-extract.md` | **Gemini** (large context, 30–40 PDFs) |
| `/taxbro-checklist` | `.agents/commands/taxbro-checklist.md` | Either |
| `/taxbro-check-w2s` | `.agents/commands/taxbro-check-w2s.md` | Either |
| `/taxbro-check-fbar` | `.agents/commands/taxbro-check-fbar.md` | Either |
| `/taxbro-check-pfic` | `.agents/commands/taxbro-check-pfic.md` | **Gemini** (web search for AMFI NAVs) |
| `/taxbro-foreign-tax-credit` | `.agents/commands/taxbro-foreign-tax-credit.md` | Either |
| `/taxbro-check-childcare` | `.agents/commands/taxbro-check-childcare.md` | Either |
| `/taxbro-rental-income` | `.agents/commands/taxbro-rental-income.md` | Either |
| `/taxbro-generate-worksheets` | `.agents/commands/taxbro-generate-worksheets.md` | Either |
| `/taxbro-validate-return` | `.agents/commands/taxbro-validate-return.md` | **Claude** (nuanced cross-checking) |
| `/taxbro-visualize` | `.agents/commands/taxbro-visualize.md` | **Claude** (complex HTML generation) |

---

## agent-log.md Protocol

**Location:** `{SOURCE_FOLDER}/TAXBRO/agent-log.md`

After completing any skill, append an entry in this format:

```markdown
## {YYYY-MM-DD HH:MM} — {agent-id} — /{skill-name}
- **Status:** complete | partial | failed
- **Artifacts written:** {comma-separated filenames}
- **Docs processed:** N of M
- **Key findings:** {1–3 bullet points of non-obvious findings}
- **Uncertain / needs follow-up:** {items that need verification}
- **Suggested next:** {skill names}
```

**Agent identifiers** (use exactly these strings):
- `gemini-2.5-pro`
- `gemini-2.0-flash`
- `claude-sonnet-4-6`
- `claude-opus-4-6`

At the start of every skill, read the log (if it exists) to avoid redundant work.
Never overwrite prior entries — always append.

---

## When to Use Gemini vs Claude

**Use Gemini for:**
- `/taxbro-extract` — reads 30–40 PDFs simultaneously; benefits from 1M+ token context window
- `/taxbro-check-pfic` — can query AMFI NAVs directly with web search enabled
- Any skill where you want to process many documents in a single pass

**Use Claude for:**
- `/taxbro-visualize` — complex self-contained HTML generation with embedded logic
- `/taxbro-validate-return` — nuanced line-by-line cross-checking of a completed return
- Interactive Q&A and follow-up questions on extracted data

Both agents can run any skill. The recommendations above are performance optimizations, not hard limits.

---

## Core Data Safety Rules (Non-Negotiable)

1. **NO PII IN APP DIRECTORY:** Never write personal data into `~/claude/taxbro*/` — no names, SSNs, account numbers, balances, income figures, or addresses.
2. **ISOLATED OUTPUT:** All analysis outputs go to `{SOURCE_FOLDER}/TAXBRO/` only.
3. **SOURCE FOLDER CONTRACT:** Always read `.current-session` to establish `SOURCE_FOLDER`. Never hardcode paths.
4. **NO SENSITIVE COMMITS:** Never stage or commit files from `{SOURCE_FOLDER}/TAXBRO/`.
5. **NO DELETIONS EVER:** Never run `rm`, `rmdir`, or any equivalent. Only permitted operations: read, write, move to `.snapshots/`.

---

## Source Folder Contract

When pointed at a source folder, TaxBro expects:
- A `CLAUDE.md` describing the filer's situation (optional but recommended)
- Tax documents (PDFs, statements, etc.) — any naming convention works
- TaxBro creates a `TAXBRO/` subdirectory for all outputs

**Output files** (all in `{SOURCE_FOLDER}/TAXBRO/`):

| File | Produced by |
|------|-------------|
| `US-knowledge-graph.md` | `/taxbro-extract` |
| `US-checklist.md` | `/taxbro-checklist` |
| `US-w2-summary.md` | `/taxbro-check-w2s` |
| `US-fbar-summary.md` | `/taxbro-check-fbar` |
| `US-pfic-summary.md` | `/taxbro-check-pfic` |
| `US-ftc-summary.md` | `/taxbro-foreign-tax-credit` |
| `US-childcare-summary.md` | `/taxbro-check-childcare` |
| `US-rental-income.md` | `/taxbro-rental-income` |
| `US-worksheets.md` | `/taxbro-generate-worksheets` |
| `US-validation-report.md` | `/taxbro-validate-return` |
| `US-knowledge-graph.html` | `/taxbro-visualize` |
| `agent-log.md` | All skills (append) |
| `session-notes.md` | `/taxbro-init` |

---

## Tool Usage Notes

- Read files by path — do not use tool-specific names like "Read tool" or "Glob tool"; just say "read the file"
- For file discovery, use shell: `find "{SOURCE_FOLDER}" -follow -type f -not -path "*/TAXBRO/*" -not -name ".DS_Store"`
- Skip `.snapshots/` in all operations: `-not -path "*/TAXBRO/.snapshots/*"`
- IRS exchange rates: use IRS Publication 54 average annual rates for recurring foreign income
- All monetary values in knowledge graph: note the conversion rate and mark as `~` (estimated) when converted

---

*Skills live in `.agents/commands/`. Claude Code accesses them via `.claude/commands/` (symlink). Gemini reads from `.agents/commands/` directly.*
