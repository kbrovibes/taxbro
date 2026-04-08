# TaxBro Backlog

## High Priority
- [ ] Implement `/taxbro-next` (state-aware orchestrator) — *DONE 2026-04-07*
- [ ] Standardize `agent-log.md` protocol across all skills — *DONE 2026-04-07*

## Bridge to Filing (New Ideas)
Detailed in [Spec 002: Bridge to Filing](./specs/002-bridge-to-filing.md)

- [ ] **`/taxbro-fetch-forms`**: Auto-detect and download blank IRS PDFs needed based on Knowledge Graph findings.
- [ ] **`/taxbro-prefill-map`**: Generate a line-by-line mapping of Knowledge Graph facts to specific IRS form boxes.
- [ ] **`/taxbro-research`**: Automated IRS instruction lookups and reference archiving for edge cases.
- [ ] **`/taxbro-clarify`**: Interactive Socratic interview to resolve `⚠️ Action Required` items.

## Feature Enhancements
- [ ] **Multi-year Support**: Support comparing prior year Knowledge Graph vs current year for trend analysis.
- [ ] **Exchange Rate Automation**: Pull Treasury Reporting Rates of Exchange via API instead of manual Pub 54 lookup.
- [ ] **Visualization 2.0**: Create a React-based dashboard for the Knowledge Graph (standalone HTML today).

## Maintenance
- [ ] Formalize `CLAUDE.md` schema for better missing document detection.
- [ ] Add unit tests for INR/CAD to USD conversion logic.
