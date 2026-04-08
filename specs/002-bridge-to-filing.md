# Spec 002 — Bridge to Filing (Forms & Research)

**Status:** Draft
**Date:** 2026-04-07
**Author:** Gemini 2.0 Flash

---

## Overview
This specification bridges the gap between **TaxBro's** analytical engine (Knowledge Graph) and the mechanical act of filing tax returns. It borrows "last-mile" concepts from the `us-federal-tax-assistant-skill` but adapts them to TaxBro's verifiable, cross-border architecture.

## Proposed Features

### 1. `/taxbro-fetch-forms` (Official Document Sourcing)
Automatically identify and download required IRS PDFs based on the Knowledge Graph.

- **Trigger:** Reads `US-pfic-summary.md`, `US-fbar-summary.md`, and `US-checklist.md`.
- **Logic:**
    - Detect required form numbers (e.g., 8621, 1116, 8938, 1040 Sch B).
    - Use Gemini Web Search to find the direct download URL from `irs.gov` for the **current tax year**.
    - Download PDFs to `{SOURCE_FOLDER}/TAXBRO/forms/`.
- **Value:** Eliminates manual searching; ensures correct form versions.

### 2. `/taxbro-prefill-map` (Exact Field Mapping)
Instead of a general summary, produce a field-by-field map that matches IRS form box numbers.

- **Format:** A Markdown table or CSV.
- **Example:**
  | Form | Line | Description | Value | Source |
  |---|---|---|---|---|
  | 1116 | 1a | Gross Income (Passive) | $5,432 | US-ftc-summary.md |
  | 8621 | 7a | Units held at year end | 5331.9 | US-pfic-summary.md |
- **Value:** Makes manual entry into TurboTax or Fillable PDFs 10x faster.

### 3. `/taxbro-research` (Legal Basis & Citations)
Curate specific IRS instruction text or revenue rulings relevant to the filer's edge cases.

- **Logic:** Search IRS.gov for keywords like "NRE interest taxability" or "PFIC de minimis 1291".
- **Output:** Save citations to `{SOURCE_FOLDER}/TAXBRO/references/`.
- **Value:** Provides the "Why" behind a flag to help the filer or their human CPA.

### 4. `/taxbro-clarify` (Interactive Interview)
A Socratic skill to resolve `⚠️ Action Required` items in the checklist.

- **Logic:** Parse the checklist for flags. Prompt the user for missing info (e.g., "Was this FD closure a rollover?").
- **Output:** Update `US-knowledge-graph.md` with the user's answers marked as `✓ (user-confirmed)`.
- **Value:** Turns "Unknowns" into "Facts" without leaving the CLI.

---

## Implementation Notes
- **Web Search:** Gemini should be the primary agent for `/taxbro-fetch-forms` and `/taxbro-research`.
- **Field Mapping:** Claude is better suited for generating the precise `/taxbro-prefill-map` due to its attention to structured formatting.
- **Safety:** Never write PII into the pre-fill maps if they are ever moved out of the source folder.
