Comprehensive international compliance analysis — FBAR, Form 8938, PFIC/Form 8621, and Foreign Tax Credit.

This is the deep-dive skill for foreign accounts and assets. Run it after `/taxbro-extract` when the
knowledge graph flags international compliance items (FBAR threshold, PFIC holdings, foreign taxes paid).

Usage:
  /taxbro-compliance              — full compliance analysis
  /taxbro-compliance --section X  — one section only (FBAR, PFIC, FTC)

Output: {SOURCE_FOLDER}/TAXBRO/US-compliance-summary.md

⚠️ NEVER DELETE ANY FILE. Only write to TAXBRO/US-compliance-summary.md.
⚠️ COMPUTATION RULE: Read "Computed Totals" from US-knowledge-graph.md first. Use those values.

---

## Steps

1. Read `.current-session` → SOURCE_FOLDER.
2. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` — primary data source.
   If missing, stop: "Run /taxbro-extract first."
3. Read `{SOURCE_FOLDER}/CLAUDE.md` for filer context (who holds accounts, prior elections, etc.).

---

## Section 1 — Foreign Accounts (FBAR + Form 8938)

Read the Foreign Accounts section of the knowledge graph.

### Per-account analysis
For each foreign account:
- Institution name, country, account type (NRE/NRO/savings/FD/checking)
- Owner (primary/spouse/joint)
- Max balance during year (local currency + USD)
- December 31 balance (local currency + USD)
- Interest earned, TDS withheld
- Data confidence (✓ confirmed from statement / ⚠️ estimated / ✗ missing)

If any account has ⚠️ balances, flag: "Statement review needed — balances estimated."

### FBAR determination (FinCEN 114)
- Sum ALL foreign account max balances (USD)
- If aggregate > $10,000 at any point: **FBAR required**
- Due: April 15 (auto-extension to October 15)
- Filed at: bsaefiling.fincen.treas.gov (free)
- Note: closed accounts still reportable if held at any point during the year

### Form 8938 determination (FATCA)
- MFJ US resident thresholds: >$100,000 year-end OR >$150,000 at any point
- Include: bank accounts + PFIC fund values (from PFIC section below)
- Filed with Form 1040

### NRE vs NRO flags
- NRE: interest tax-free in India, **fully taxable in US**, no FTC available
- NRO: interest taxable in India (TDS withheld), taxable in US, FTC available for TDS

### Schedule B Part III
List all accounts requiring "yes" answer to foreign account questions.

---

## Section 2 — PFIC Analysis (Form 8621)

Read the Indian Mutual Funds (PFIC) section of the knowledge graph.

### Per-fund analysis
For each fund:
- Fund name, AMC, owner, folio
- Dec 31 units, NAV (INR), value (INR + USD)
- Redemptions during calendar year? (date, units, proceeds)
- Dividends/distributions during calendar year?
- Prior election status (default/QEF/MTM)

### De minimis check
- Sum all PFIC year-end values (USD)
- MFJ threshold: aggregate ≤ $50,000 AND no individual fund > $25,000
- If under threshold AND no redemptions/distributions: Form 8621 likely not required
- If over threshold OR any transactions: Form 8621 required per fund

### Excess distribution analysis (if any redemptions/dividends)
For each fund with activity:
- Excess distribution = current year distribution − (125% × average of prior 3 years)
- Allocated over holding period, taxed at highest marginal rate per year + interest
- Flag: "Excess distribution calculation required — provide to preparer"

### Election analysis
- Default method (excess distribution): applies if no prior QEF/MTM elections
- QEF: requires fund-level income statement (Indian MFs rarely provide this)
- MTM: simpler but triggers annual unrealized gain/loss recognition
- Recommendation: note whether switching elections is worth the purging election cost

---

## Section 3 — Foreign Tax Credit (Form 1116)

Read the Credits / Foreign Tax Credit section of the knowledge graph.

### Passive basket
| Item | Gross Foreign Income (USD) | Foreign Tax Paid (USD) | Source |
|------|---------------------------|----------------------|--------|
| NRO savings interest | | | Interest certificates |
| NRO FD interest | | | Interest certificates |
| Dividends from foreign sources | | | If any |
| **Passive total** | | | |

### General basket
| Item | Gross Foreign Income (USD) | Foreign Tax Paid (USD) | Source |
|------|---------------------------|----------------------|--------|
| India advance tax payments | | | Challans |
| Rental TDS (if any) | | | TDS certificate |
| Canada wages/tax (if any) | | | T4 |
| **General total** | | | |

### NRE interest callout
- NRE interest amount (USD): fully taxable in US
- No Indian TDS → **no FTC available** for this income
- Common mistake: claiming FTC on NRE interest. Flag prominently.

### FTC limitation
- Limitation = (Foreign income ÷ Worldwide income) × US tax
- With large US income, limitation rarely binding (credit usually allowed in full)
- Note excess FTC carries forward 10 years

### Carryforwards from prior year
| Basket | Regular FTC | AMT FTC | Source |
|--------|------------|---------|--------|
| General | | | Prior return |
| Passive | | | Prior return |

---

## Output Format

Write `{SOURCE_FOLDER}/TAXBRO/US-compliance-summary.md` with:

```markdown
# TaxBro International Compliance Summary — Tax Year {YEAR}
<!-- Generated: {DATE} | Source: US-knowledge-graph.md -->

## Filing Requirements Summary
| Form | Required? | Reason | Due Date |
|------|-----------|--------|----------|
| FinCEN 114 (FBAR) | | | Apr 15 (auto-ext Oct 15) |
| Form 8938 (FATCA) | | | With 1040 |
| Form 8621 (PFIC) | | | With 1040 |
| Form 1116 (FTC) | | | With 1040 |
| Schedule B Part III | | | With 1040 |

## Section 1 — Foreign Accounts
[Account table, FBAR determination, Form 8938 determination, NRE/NRO flags]

## Section 2 — PFIC Holdings
[Fund table, de minimis check, excess distribution analysis, election status]

## Section 3 — Foreign Tax Credit
[Passive basket, general basket, NRE callout, limitation, carryforwards]

## Action Items
[Numbered list of what filer/preparer needs to do]
```

---

## Privacy Note
All output goes to {SOURCE_FOLDER}/TAXBRO/US-compliance-summary.md only.
Never write financial data to ~/claude/taxbro/.
