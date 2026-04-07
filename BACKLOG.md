# Backlog

Tracks completed work and planned improvements.
Items are roughly prioritized within each section — top = higher priority.

---

## Done

- [x] `/taxbro-init` — session initialization: load source folder, read CLAUDE.md, categorize documents, create TAXBRO/ output dir
- [x] `/taxbro` — top-level overview command: commands, session status, output file reference
- [x] `/tax-checklist` — full status sweep with ✅/⚠️/❌ status per issue
- [x] `/check-w2s` — W-2 analysis: wages, excess SS, DCFSA, HSA employer contribs
- [x] `/check-fbar` — FBAR + Form 8938 thresholds, NRE/NRO classification, USD conversion
- [x] `/check-pfic` — PFIC determination per fund, de minimis, election method tracking
- [x] `/foreign-tax-credit` — Form 1116 by basket: passive (NRO TDS) and general (advance tax, rental)
- [x] `/check-childcare` — Form 2441: provider EINs, DCFSA offset, earned-income test
- [x] `/rental-income` — Schedule E: INR→USD, TDS, 30-year depreciation
- [x] `/generate-worksheets` — pre-filled line-item worksheets for all applicable IRS forms
- [x] `/validate-return` — line-by-line cross-check of draft/completed return against source docs
- [x] Dual-CLI support: Claude Code (`.claude/commands/`) and Gemini CLI (`GEMINI.md`)
- [x] PII isolation architecture: all outputs to SOURCE_FOLDER/TAXBRO/, nothing in repo
- [x] Personalization via CLAUDE.md in source folder (takes priority over TaxBro defaults)
- [x] Published to GitHub: https://github.com/kbrovibes/taxbro
- [x] Renamed `/tax-init` → `/taxbro-init` for consistent naming
- [x] Detailed README with install, quick-start walkthrough, per-command docs, supported forms table

---

## Up next

### High priority

- [ ] **`/check-1099s`** — analyze 1099-B (capital gains, wash sales), 1099-DIV (dividends, qualified vs ordinary), 1099-INT (interest, OID). Currently checklist only checks presence; this would extract and summarize values for Schedule D and Schedule B
- [ ] **`/check-hsa`** — HSA analysis: 1099-SA (distributions) vs 5498-SA (contributions), contribution limit check (employee + employer vs IRS limit), qualified vs non-qualified distributions, Form 8889 inputs
- [ ] **`/session-notes`** — freeform note-taking skill that appends timestamped notes to `TAXBRO/session-notes.md` without overwriting; useful for recording preparer conversations, decisions, open questions
- [ ] **IRS exchange rate lookup** — skills currently reference "use IRS Pub 54" but don't embed the rate; add a lookup table or prompt the user for the rate at runtime and persist it to session notes

### Medium priority

- [ ] **Multi-year support** — currently assumes single tax year; skills should detect or ask about the year when ambiguous (especially relevant when documents span years)
- [ ] **Canada-specific handling** — T4 analysis, NR4 withholding, Canada-US treaty provisions, provincial vs federal taxes; currently only partially handled in `/foreign-tax-credit`
- [ ] **`/check-mortgage`** — 1098 analysis: deductible vs non-deductible interest based on $750K acquisition debt limit, points paid, PMI (if deductible), second home rules
- [ ] **Document completeness report** — `/taxbro-init` flags missing docs, but a dedicated `/check-documents` skill could do a thorough gap analysis: expected docs from CLAUDE.md vs what's in the folder, with explicit missing-document list
- [ ] **Estimated tax / safe harbor check** — flag if prior year taxes suggest underpayment exposure; check if quarterly estimated payments were made and documented

### Lower priority / future

- [ ] **`/amend-analysis`** — analyze whether an amendment (Form 1040-X) might be warranted based on prior year return vs newly discovered information
- [ ] **State return support** — most skills are federal-only; add state-specific handling for common states (CA, NY, TX, WA) especially for foreign income treatment
- [ ] **Crypto / digital asset handling** — Form 1099-DA (new for 2025), wallet-level gain/loss analysis, staking income
- [ ] **Stock options / RSU handling** — 83(b) elections, ESPP disqualifying dispositions, AMT exposure from ISOs
- [ ] **Schedule K-1 support** — partnership and S-corp income/loss passthrough
- [ ] **Alternative minimum tax (AMT) flag** — ISO exercise, large itemized deductions, high income — flag AMT exposure so preparer can model it
- [ ] **`/prior-year-compare`** — compare this year's analysis to last year's (if prior TAXBRO outputs exist), flag significant swings
- [ ] **Interactive mode** — skills currently output markdown files silently; an interactive mode that walks the user through each form with questions and confirmations could help less experienced filers
- [ ] **Gemini skill parity** — GEMINI.md describes the architecture but individual Gemini-specific command wrappers don't exist yet; consider a dedicated `.gemini/commands/` equivalent

---

## Ideas / not yet scoped

- Export worksheets as CSV for direct import into tax software (TurboTax, H&R Block)
- Generate a "cover letter" for the preparer summarizing situation, open questions, and what TaxBro found
- Integration with IRS transcript data (if accessible via API) to cross-check withholding and estimated payments
- Automated IRS exchange rate fetching from Treasury FMS data
