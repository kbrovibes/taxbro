# Spec 003: TaxBro Simplification — Strip to What Matters

**Status:** Draft
**Date:** 2026-04-07
**Author:** Claude Opus 4.6 + Karthik

---

## Problem

TaxBro has 15 skills, 12 output files, an agent-log handoff protocol, an orchestrator, staleness detection, and a parallel extraction strategy. For a single filer doing one return per year, this is over-engineered.

The core issue: **a 1-shot "read my folder and compute my taxes" prompt produces a more accurate estimated tax number** than the full TaxBro pipeline, because the 1-shot keeps everything in one context window with no intermediate artifacts to drift, go stale, or be misinterpreted by a different agent.

Meanwhile, TaxBro's real value is in things a 1-shot is bad at:
1. **Compliance checklists** — FBAR, PFIC, Form 8938, Form 2441 have threshold logic and edge cases that benefit from dedicated, tested prompts
2. **Return validation** — cross-checking a CPA's draft against source docs is a distinct task that doesn't exist until after the return is prepared
3. **Visualization** — producing a durable HTML artifact you can reference during filing
4. **Repeatability** — when you get a revised draft, you can re-validate without re-explaining your entire situation

The pipeline skills (extract → checklist → w2 → fbar → pfic → ftc → childcare → rental → worksheets) mostly duplicate what a single well-prompted pass already does. They add process without adding insight.

---

## Goals

1. **Reduce skill count** from 15 to ~6 high-value skills
2. **Eliminate intermediate file dependencies** that cause agent drift
3. **Keep the knowledge graph** as an optional caching layer, not a required pipeline stage
4. **Preserve compliance value** — the FBAR/PFIC/FTC/2441 checklists are genuinely useful
5. **Make every skill independently runnable** — no required ordering, no orchestrator needed
6. **Keep multi-agent interop** working (Claude + Gemini can both run any skill)

## Non-goals

- Building a tax preparation tool (TaxBro is an analysis/validation aid)
- Replacing the CPA or tax software
- Supporting multiple filers or multi-year in this iteration

---

## Proposed Skill Set (6 skills, down from 15)

### Tier 1 — Core (always useful)

#### `/taxbro-init [path]`
**Keep as-is.** Points TaxBro at a source folder. No changes needed.

#### `/taxbro-extract`
**Simplify.** Still builds `US-knowledge-graph.md`, but now includes Computed Totals with estimated tax/refund. This is the "1-shot but saved to a file" — one pass, all documents, full computation.

Key change: the knowledge graph becomes a **complete analysis** in one file, not just raw extracted data that needs 8 downstream skills to interpret. The Computed Totals section should include:
- Full income breakdown with sources
- AGI computation
- Itemized vs standard deduction comparison
- Tax bracket computation (simplified)
- Additional Medicare Tax + NIIT
- All payments and credits
- Estimated refund/owe
- All compliance flags (FBAR, PFIC, FTC thresholds)

This means `/taxbro-extract` subsumes what today requires extract + checklist + w2 + fbar + pfic + ftc + childcare + rental + worksheets. One skill, one output file, one context window.

#### `/taxbro-visualize`
**Keep, enhanced.** Reads the knowledge graph and produces `US-knowledge-graph.html`. The recent overhaul (tax waterfall, per-item impact, threshold bars) is the right direction. This is the "make it visual" step.

### Tier 2 — Validation (use when you have a draft return)

#### `/taxbro-validate-return [pdf]`
**Keep as-is.** This is TaxBro's killer feature — cross-checking a CPA's draft against source documents. No changes needed. Reads knowledge graph + the PDF.

### Tier 3 — Deep Dives (use when you need focused analysis)

#### `/taxbro-compliance`
**NEW — merge of check-fbar + check-pfic + foreign-tax-credit.** One skill that does all international compliance:
- FBAR account table + threshold analysis
- Form 8938 threshold analysis
- PFIC fund table + de minimis check + Form 8621 determination
- FTC basket breakdown + carryforward analysis
- NRE interest (no FTC) callout

Outputs `US-compliance-summary.md`. This is the deep-dive version of what the knowledge graph flags briefly.

#### `/taxbro-worksheets`
**Simplified.** Generates preparer-ready line items from the knowledge graph. But now the knowledge graph already has Computed Totals, so this skill is thinner — it's mostly reformatting into IRS form layout, not computing.

### Removed Skills

| Skill | Why removed | Where it went |
|-------|-------------|---------------|
| `/taxbro-next` | Orchestrator adds process without insight | Not needed — skills are independent |
| `/taxbro-checklist` | Duplicate of knowledge graph Issues section | Subsumed by `/taxbro-extract` |
| `/taxbro-check-w2s` | 1-shot extract handles W-2 analysis | Subsumed by `/taxbro-extract` |
| `/taxbro-check-fbar` | Merged into `/taxbro-compliance` | See compliance skill |
| `/taxbro-check-pfic` | Merged into `/taxbro-compliance` | See compliance skill |
| `/taxbro-foreign-tax-credit` | Merged into `/taxbro-compliance` | See compliance skill |
| `/taxbro-check-childcare` | Simple enough for extract to handle | Subsumed by `/taxbro-extract` |
| `/taxbro-rental-income` | Simple enough for extract to handle | Subsumed by `/taxbro-extract` |
| `/taxbro-generate-worksheets` | Renamed to `/taxbro-worksheets` | Simplified |
| `/taxbro` (overview) | README serves this purpose | README |

### Removed Output Files

| File | Replacement |
|------|-------------|
| `US-checklist.md` | Issues section of knowledge graph |
| `US-w2-summary.md` | Income section of knowledge graph |
| `US-fbar-summary.md` | `US-compliance-summary.md` |
| `US-pfic-summary.md` | `US-compliance-summary.md` |
| `US-ftc-summary.md` | `US-compliance-summary.md` |
| `US-childcare-summary.md` | Deductions section of knowledge graph |
| `US-rental-income.md` | Foreign Income section of knowledge graph |
| `agent-log.md` | Removed — no multi-step pipeline to track |

### Kept Output Files (5, down from 12)

| File | Produced by |
|------|-------------|
| `US-knowledge-graph.md` | `/taxbro-extract` (now includes full analysis + computed totals) |
| `US-knowledge-graph.html` | `/taxbro-visualize` |
| `US-compliance-summary.md` | `/taxbro-compliance` |
| `US-worksheets.md` | `/taxbro-worksheets` |
| `US-validation-report.md` | `/taxbro-validate-return` |
| `session-notes.md` | `/taxbro-init` |

---

## Workflow (simplified)

### Standard flow
```
/taxbro-init /path/to/taxes/
/taxbro-extract                    ← does everything in one pass
/taxbro-visualize                  ← open the HTML dashboard
```

That's it. Three commands to go from raw documents to a complete analysis with an interactive dashboard.

### When you need more
```
/taxbro-compliance                 ← deep dive on FBAR/PFIC/FTC if flagged
/taxbro-worksheets                 ← preparer-ready form line items
```

### When your CPA sends a draft
```
/taxbro-validate-return draft.pdf  ← cross-check everything
```

---

## Impact on Multi-Agent Interop

### What changes
- `agent-log.md` protocol removed — no pipeline to track
- `.agents/commands/GEMINI.md` simplified (fewer skills to list)
- Skill recommendation table simplified: Gemini for extract (large context), Claude for visualize + validate

### What stays
- Skills still live in `.agents/commands/`
- Claude reads via `.claude/commands/` symlink
- Gemini reads from `.agents/commands/` directly
- Both can run any skill

---

## Migration Plan

### Phase 1: Enhance `/taxbro-extract`
- Add the full analysis (currently in 8 separate skills) to the extract skill
- Knowledge graph schema grows to include: childcare analysis, rental income computation, W-2 excess SS detail, FBAR/PFIC threshold flags with amounts, and the Computed Totals section (already added)
- Test: run extract, verify all the analysis that used to require 8 skills is now in one file

### Phase 2: Create `/taxbro-compliance`
- Write new skill that merges FBAR + PFIC + FTC into one comprehensive output
- Test: run compliance, verify it covers everything the 3 separate skills did

### Phase 3: Simplify remaining skills
- Thin out `/taxbro-worksheets` (renamed from generate-worksheets)
- Remove `/taxbro-next`, `/taxbro-checklist`, and the individual check-* skills
- Update README, CLAUDE.md, GEMINI.md

### Phase 4: Clean up
- Archive removed skill files to a `/.agents/commands/archive/` folder (never delete)
- Update specs index

---

## Open Questions

1. **Should `/taxbro-compliance` be kept separate or folded into extract?**
   Argument for separate: it's a deep-dive with account-level detail that bloats the knowledge graph. Argument against: another intermediate file.

2. **Should we keep `agent-log.md`?**
   It's useful for debugging ("which agent last ran what") even without the pipeline. Could keep it as optional metadata.

3. **Is `/taxbro-worksheets` worth keeping?**
   If the knowledge graph's Computed Totals section has all the line items, the worksheets skill is just reformatting. A CPA might prefer the knowledge graph directly.

4. **Should `/taxbro --status` survive?**
   It's handy but could be a one-liner in the README: "run `ls -lt TAXBRO/` to see what's there."

---

## Decision Criteria

Implement this if:
- The 1-shot approach + the enhanced extract produces numbers as good or better than the current pipeline
- The reduced skill count makes the system feel lighter to use
- Compliance deep-dives still catch the edge cases that matter

Don't implement if:
- The knowledge graph becomes too large for a single context window when combined with source docs
- Losing the individual skills means losing nuance in specific areas (e.g., PFIC de minimis logic)
