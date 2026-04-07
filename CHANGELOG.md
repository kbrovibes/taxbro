# Changelog

All notable changes to TaxBro are documented here.
Format: `## [date] — description`, with details on what changed and why.

---

## [2026-04-07] — Initial release + project restructuring

### Added
- **`/taxbro-init`** (renamed from `/tax-init`) — initializes a session by loading a source folder, reading CLAUDE.md, discovering and categorizing documents, and writing a session log
- **`/taxbro`** — new top-level overview command: shows project description, all commands, current session status, and output file reference
- **`/tax-checklist`** — full status sweep across all applicable tax issues; checks document presence, analysis completion, and flags risks
- **`/check-w2s`** — W-2 analysis: wages, federal/SS/Medicare withholding, excess SS calculation, DCFSA (Box 10), HSA employer contributions (Box 12 Code W)
- **`/check-fbar`** — foreign account analysis: max and year-end balances in USD, FBAR (FinCEN 114) threshold check, Form 8938 (FATCA) threshold check, NRE vs NRO classification
- **`/check-pfic`** — PFIC analysis for Indian (and other foreign) mutual funds: per-fund Form 8621 determination, de minimis check, redemption flags, election method tracking
- **`/foreign-tax-credit`** — consolidates all foreign taxes paid into Form 1116 passive and general baskets; handles NRO TDS, advance tax, rental TDS, Canadian withholding
- **`/check-childcare`** — Form 2441 analysis: provider EINs, qualifying expense cap, DCFSA offset, earned-income test with spouse flag
- **`/rental-income`** — foreign rental income: INR→USD conversion (IRS Pub 54), TDS extraction, Schedule E line items, 30-year depreciation calculation
- **`/generate-worksheets`** — pre-filled worksheets for FinCEN 114, Form 8938, Form 1116 (both baskets), Form 8621 (per fund), Schedule E, Form 2441, Form 1040 summary
- **`/validate-return`** — cross-checks a completed or draft return PDF against all source documents and TaxBro analysis outputs; line-by-line pass/flag/mismatch results
- **CLAUDE.md** — project instructions, PII safety rules, source folder contract, skill reference
- **GEMINI.md** — Gemini CLI equivalent configuration for dual-CLI support
- **README.md** — full project documentation with install, quick-start walkthrough, per-command details, personalization guide, supported forms reference

### Design decisions
- All outputs isolated to `{SOURCE_FOLDER}/TAXBRO/` — zero personal data in the repo
- CLAUDE.md in source folder takes priority over TaxBro defaults for all skills
- `.current-session` stores only a path; gitignored
- Skills support both `$ARGUMENTS` (direct path) and `.current-session` (session-based) invocation
