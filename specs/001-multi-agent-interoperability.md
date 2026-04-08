# Spec 001 — Multi-Agent Interoperability (Claude + Gemini)

**Status:** Draft
**Date:** 2026-04-07
**Author:** Karthik (with Claude Sonnet 4.6)

---

## Problem

TaxBro currently runs exclusively inside Claude Code. Skills live in `.claude/commands/` and are
invoked via `/skill-name` — a mechanism specific to Claude Code. Gemini CLI has no equivalent
command expansion system.

In practice, different agents have different strengths for TaxBro tasks:

- **Gemini 2.5 Pro** has a 1M+ token context window, making it better suited for
  `taxbro-extract` (reads 30–40 PDFs simultaneously) and tasks that require web lookups
  (e.g. pulling AMFI NAVs for PFIC valuations).
- **Claude** (Sonnet/Opus) produces higher-quality structured output for complex analysis,
  HTML generation (`taxbro-visualize`), and nuanced cross-checking (`taxbro-validate-return`).

The goal is to be able to run an intensive extraction step in Gemini, then pick up in Claude
to run visualization and validation — with no re-extraction and no loss of context.

### Current gaps

1. **Skill invocation**: Gemini has no `/command-name` expansion. The user has to manually
   instruct Gemini to find and follow a skill file.
2. **Handoff state**: When Gemini completes a skill, Claude picking up afterward has no
   structured record of what was run, what was produced, or what remains uncertain.
3. **Workspace divergence**: Two local clones of the same GitHub repo (`taxbro/` and
   `taxbro-2/`) can drift out of sync. Skills pushed from one won't be available in
   the other until `git pull` is run manually.
4. **Skill naming**: `.claude/commands/` signals Claude-only ownership. Skills are actually
   LLM-agnostic instruction files and should live in a neutral location.

---

## Goals

- Both Claude Code and Gemini CLI can execute any TaxBro skill against the same source folder.
- An extraction run by Gemini can be seamlessly continued by Claude (and vice versa) with no
  re-processing of source documents.
- A human reading the output folder can tell which agent ran which skill, when, and what it found.
- Adding a new skill in the future requires changes in exactly one place.
- The solution works with the existing two-workspace model (`taxbro/` and `taxbro-2/`).

## Non-goals

- Automatic skill invocation or pipelines between agents (the user explicitly directs each step).
- Abstracting away agent-specific capabilities (e.g. Gemini's web search, Claude's extended thinking).
- Supporting agents other than Claude Code and Gemini CLI for now.
- Modifying how Claude Code discovers and invokes skills — the symlink is transparent to it.

---

## Decision

### 1. Move skills to `.agents/commands/` — Claude reads via symlink

Skills move from `.claude/commands/` to `.agents/commands/` as the canonical location.
`.claude/commands/` becomes a relative symlink pointing there.

Claude Code follows the symlink transparently — `/taxbro-extract` still works identically.
Gemini CLI is told (via GEMINI.md) to look in `.agents/commands/` for skill definitions.

Rationale for `.agents/` over keeping `.claude/commands/`:
- Signals that skills belong to the project, not to a specific agent.
- If Gemini CLI ever ships a `.gemini/commands/` feature (Claude Code precedent makes this
  plausible), adding support is one additional symlink, no content moves.
- A future third agent (e.g. GPT-4o via `openai` CLI) can be added the same way.

### 2. Teach Gemini the skill convention via GEMINI.md

GEMINI.md at the project root is rewritten to clearly instruct Gemini:
- When the user types `/taxbro-extract` (or any `/command-name`), read
  `.agents/commands/command-name.md` in full and execute its steps.
- Always read `.current-session` first to establish SOURCE_FOLDER.
- Always read `{SOURCE_FOLDER}/TAXBRO/agent-log.md` at session start if it exists.

This relies on Gemini's instruction-following, which is robust for this use case.
No wrappers, no shell scripts, no external tooling.

### 3. Handoff via `agent-log.md` in the TAXBRO output folder

Each skill appends a structured entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` upon completion.
Each skill reads this log at the start of execution to understand prior work.

The log lives in the source folder (alongside other TAXBRO outputs) — never in the repo.
It is structured markdown so both agents can read and write it without parsing code.

### 4. Two-workspace model stays; worktree migration is deferred

`taxbro/` (2026 returns) and `taxbro-2/` (2025 returns) remain separate local clones of the
same GitHub repo. Each has its own `.current-session` pointing to its respective tax year.

The `.agents/commands/` restructure is committed to the GitHub repo, so `git pull` in either
workspace gives both the same canonical skills.

A future migration to `git worktree` (shared git objects, independent working trees) is noted
as cleaner but deferred — the two-clone model works correctly today.

---

## Technical Design

### Repository layout after this spec is implemented

```
taxbro-2/                         (or taxbro/ — same layout)
├── .agents/
│   └── commands/                 ← canonical skill location
│       ├── taxbro.md
│       ├── taxbro-init.md
│       ├── taxbro-extract.md
│       ├── taxbro-checklist.md
│       ├── taxbro-check-w2s.md
│       ├── taxbro-check-fbar.md
│       ├── taxbro-check-pfic.md
│       ├── taxbro-foreign-tax-credit.md
│       ├── taxbro-check-childcare.md
│       ├── taxbro-rental-income.md
│       ├── taxbro-generate-worksheets.md
│       ├── taxbro-validate-return.md
│       ├── taxbro-visualize.md
│       └── GEMINI.md             ← Gemini convention doc (moved here)
├── .claude/
│   ├── commands -> ../.agents/commands    ← relative symlink
│   └── settings.json
├── specs/
│   ├── README.md
│   └── 001-multi-agent-interoperability.md   ← this file
├── CLAUDE.md                     ← unchanged, Claude-specific context
├── GEMINI.md                     ← rewritten (see §GEMINI.md design)
├── CHANGELOG.md
├── README.md
├── .current-session              ← gitignored, per-workspace
└── .gitignore
```

### Symlink

```bash
# From taxbro-2/
mv .claude/commands .agents/commands
ln -s ../.agents/commands .claude/commands
git add .agents/ .claude/commands
```

The symlink target is relative (`../.agents/commands`), not absolute, so it works in any clone.

Git tracks symlinks as first-class objects. `git clone` recreates the symlink on checkout.
macOS, Linux: fully supported. Windows: requires Developer Mode or git `core.symlinks=true`
(noted as a known limitation; TaxBro is currently macOS-only anyway).

### `agent-log.md` protocol

**Location:** `{SOURCE_FOLDER}/TAXBRO/agent-log.md`

**Format:**

```markdown
# TaxBro Agent Log
<!-- Append-only. Each agent appends one entry per skill run. Never edit prior entries. -->

## {YYYY-MM-DD HH:MM} — {agent-id} — /{skill-name}
- **Status:** complete | partial | failed
- **Artifacts written:** {comma-separated filenames}
- **Docs processed:** N of M
- **Key findings:** {1–3 bullet points of non-obvious findings}
- **Uncertain / needs follow-up:** {items that need verification}
- **Suggested next:** {skill names}
```

**Agent identifiers** (use exactly these strings):
- `claude-sonnet-4-6`
- `claude-opus-4-6`
- `gemini-2.5-pro`
- `gemini-2.0-flash`

**Rules:**
- Skills READ the log at the start and use it as prior-work context (brief scan only).
- Skills APPEND to the log at the end — never overwrite prior entries.
- If the log does not exist, create it with the header and first entry.
- Log entries go to the source folder, never to the taxbro repo.

**Skill integration points** (add to every skill's Step 1 and final step):

```
Step 1 (add): Read `{SOURCE_FOLDER}/TAXBRO/agent-log.md` if it exists.
              Note any prior runs of this skill or related skills to avoid redundancy.

Final step (add): Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md`
                  recording: agent, skill, status, artifacts, key findings, next steps.
```

### GEMINI.md design

The root `GEMINI.md` must accomplish three things that the current version does not:

**1. Skill invocation convention**

```markdown
## Invoking Skills

When the user types `/skill-name` (e.g. `/taxbro-extract`):
1. Read `.agents/commands/skill-name.md` in full.
2. Execute all steps in that file exactly as written.
3. Do not summarize or skip steps — each step is load-bearing.

List of available skills: [list all skill names]
```

**2. Session startup protocol**

```markdown
## At the Start of Every Session
1. Read `.current-session` → SOURCE_FOLDER path.
2. If `{SOURCE_FOLDER}/TAXBRO/agent-log.md` exists, read it.
   Summarize prior work in one sentence before proceeding.
3. Tell the user: "Session loaded. Source: {folder}. Prior work: {summary}."
```

**3. Agent capability guidance**

```markdown
## When to Use Gemini vs Claude

Use Gemini for:
- `/taxbro-extract` — reads 30–40 PDFs; benefits from 1M+ context window
- `/taxbro-check-pfic` — may require web lookups (AMFI NAV data)
- Any skill where you want to process many documents in a single pass

Use Claude for:
- `/taxbro-visualize` — complex HTML generation with embedded logic
- `/taxbro-validate-return` — nuanced line-by-line cross-checking
- Interactive Q&A and follow-up questions on extracted data
```

### Skill compatibility requirements

Skills must be written to run on either agent without modification.

**Required:** Skills must not reference Claude-Code-specific tool names (`Read tool`, `Glob tool`,
`Edit tool`). Instead use agent-neutral language: "read the file", "search for files matching",
"write the file". Both agents understand these instructions.

**Required:** Skills must use `SOURCE_FOLDER` (from `.current-session`) for all file paths —
never hardcoded absolute paths.

**Recommended:** Skills should note at the top which agent is better suited:
```markdown
> Best run with: Gemini (large context) | Claude (structured output) | Either
```

**Allowed:** Skills may include agent-specific notes in a collapsible or bracketed section
for tips (e.g. "If using Gemini with web search enabled, query AMFI for NAVs directly").

---

## Migration Plan

### Phase 1 — Restructure (one-time, ~30 min)

```bash
cd ~/claude/taxbro-2

# Create .agents/commands/ and move skills
mkdir -p .agents
cp -r .claude/commands .agents/commands      # copy first, verify, then replace
rm -rf .claude/commands
ln -s ../.agents/commands .claude/commands   # relative symlink

# Verify Claude Code still sees all skills
ls .claude/commands/   # should list all *.md files via symlink

# Stage and commit
git add .agents/ .claude/commands
git commit -m "Move skills to .agents/commands/, .claude/commands is now a symlink"
git push

# Update the other workspace
cd ~/claude/taxbro && git pull
# Verify symlink was recreated correctly
ls .claude/commands/
```

### Phase 2 — Rewrite GEMINI.md (~1 hour)

Replace the root `GEMINI.md` with the full design described above.
This is the highest-leverage change for Gemini usability.

### Phase 3 — Add agent-log.md to skills (~1 hour)

Add the read-at-start and append-at-end blocks to all 12 skill files.
Start with `taxbro-extract` and `taxbro-validate-return` (highest value for handoffs).

### Phase 4 — End-to-end test

1. `cd ~/claude/taxbro-2 && gemini` (or however Gemini CLI is invoked)
2. Type `/taxbro-extract`
3. Confirm Gemini reads `.agents/commands/taxbro-extract.md` and runs it
4. Confirm `agent-log.md` is written to the source folder
5. `cd ~/claude/taxbro-2 && claude`
6. Run `/taxbro-visualize` — confirm Claude reads the agent log and skips re-extraction
7. Confirm `agent-log.md` now has two entries (Gemini + Claude)

### Phase 5 — Deferred: git worktree migration

Replace the two-clone model with a single repo + two worktrees:

```bash
# From the primary clone
git worktree add ~/claude/taxbro main   # adds taxbro/ as a worktree
# .current-session in each worktree is independent
```

Benefits: `git pull` in one worktree makes history immediately available in both.
Skills and GEMINI.md changes propagate without a separate pull step.
Deferred because it requires careful migration of the existing `taxbro/` directory state.

---

## Open Questions

| # | Question | Impact | Resolution |
|---|----------|--------|------------|
| 1 | Does the Gemini CLI binary (`gemini`) support reading GEMINI.md from the working directory automatically, or does it need to be passed explicitly? | High — affects how startup protocol works | Test during Phase 4 |
| 2 | Does Gemini CLI have a file-size limit for GEMINI.md that would prevent including the full skill convention? | Medium — might need to keep GEMINI.md lean and reference skills | Test during Phase 4 |
| 3 | Should agent-log entries include a content hash of the primary artifact (e.g. knowledge graph) so the receiving agent can detect if the artifact was regenerated? | Low — nice-to-have for detecting stale handoffs | Defer |
| 4 | Windows symlink compatibility | Low — TaxBro is macOS-only today | Note in README, revisit if needed |
| 5 | Should GEMINI.md at `.agents/commands/GEMINI.md` be kept or removed once the root GEMINI.md is comprehensive? | Low | Remove to avoid two sources of truth |

---

## Changelog

| Date | Change |
|------|--------|
| 2026-04-07 | Initial draft |
