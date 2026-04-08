Generate an interactive HTML visualization of the TaxBro Knowledge Graph.

Usage:
  /taxbro-visualize          — generate or refresh the HTML dashboard
  /taxbro-visualize --open   — generate and tell user to open the file

Output: {SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.html
Opens in any browser. Self-contained — no server required.

⚠️ NEVER DELETE ANY FILE. Only write to TAXBRO/US-knowledge-graph.html.

⚠️ COMPUTATION RULE: Read the "Computed Totals" section of US-knowledge-graph.md FIRST.
Use those pre-computed values for ALL dollar amounts shown in the dashboard.
Do NOT independently recompute income, AGI, tax, or refund/owe from raw sections.
If Computed Totals is missing, compute using the formulas in the extract schema and flag as "⚠️ estimated".

---

## Steps

1. Read `.current-session` → SOURCE_FOLDER.
2. Read `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.md` — primary data source.
   **Read the "Computed Totals" section first** — these are the authoritative numbers.
3. Read `{SOURCE_FOLDER}/TAXBRO/file-index.md` if it exists.
4. **Read `{SOURCE_FOLDER}/TAXBRO/US-validation-report.md` if it exists.**
   This file contains return-specific figures (filed values per form/line) and validation
   check results (PASS/FLAG/MISMATCH). Use it to populate the "Return Δ" tab.
5. Extract structured data from ALL sections of the knowledge graph (see Data Extraction below).
6. Build a `TAX_DATA` JSON object and embed it in the HTML file.
7. Write a self-contained HTML file to `{SOURCE_FOLDER}/TAXBRO/US-knowledge-graph.html`.
8. Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md`.

---

## Data Extraction (from Knowledge Graph)

Extract into the `TAX_DATA` JSON object:

```javascript
TAX_DATA = {
  meta: { taxYear, filingStatus, filers: [...], extractedDate, exchangeRate },

  // From "Computed Totals" section (AUTHORITATIVE — use these, don't recompute)
  totals: {
    wages, taxableInterest, ordinaryDividends, qualifiedDividends,
    capitalGain, rentalIncome, totalIncome, agi,
    standardDeduction, itemizedTotal, itemizedBreakdown: { mortgage, salt, charitable },
    deductionUsed, taxableIncome,
    regularTax, additionalMedicareTax, niit, estimatedTotalTax,
    w2Withholding, estimated1099Withholding, estimatedPayments,
    excessSS, ftc, childTaxCredit, totalPayments,
    refundOrOwe, penaltyEstimate, netRefundOrOwe
  },

  // Issue detail
  issues: [{ severity, area, issue, detail, dollarImpact }],

  // Income breakdown for charts
  income: {
    w2s: [{ employer, wages, fedWh, ssTax, medTax, dcfsa, hsa, source }],
    brokerage: [{ institution, ordDiv, qualDiv, interest, netGL, source }],
    usInterest: [{ payer, amount, source }],
    foreignInterest: { nre: { inr, usd }, nro: { inr, usd, tds } },
    rental: { grossRentINR, grossRentUSD, depreciation, palCarryforward, netIncome },
    hsa: { employerContrib, distributions, limit, overContrib }
  },

  // Deductions detail
  deductions: {
    mortgage: [{ lender, interest, principal, propTax, deductiblePortion, source }],
    mortgageCap: { totalPrincipal, limit, ratio, deductibleInterest, lostInterest },
    salt: { propTax, incomeTax, total, cap, lostSalt },
    childcare: { providers: [...], totalPaid, dcfsa, creditAmount, earnedIncomeTest }
  },

  // Foreign accounts
  foreignAccounts: {
    accounts: [{ institution, country, type, owner, last4, maxBalUSD, dec31BalUSD, source }],
    fbar: { aggregateMax, threshold, required },
    fatca: { yearEndTotal, anyPointTotal, thresholds, required }
  },

  // PFIC
  pfic: {
    funds: [{ name, amc, owner, units, nav, valueINR, valueUSD, redemptions, dividends, source }],
    aggregateUSD, threshold, deminimis, form8621Required
  },

  // Credits
  credits: {
    ftc: {
      passive: { income, taxPaid, carryforward },
      general: { income, taxPaid, carryforward }
    },
    excessSS: { total, max, excess },
    childTaxCredit: { amount, children, phaseOutStart, fullyEliminated, agiOverThreshold, reason }
  },

  // Payments
  payments: {
    w2Withholding: [{ employer, amount }],
    estimatedPayments: [{ date, amount }],
    total
  },

  // Prior year references
  priorYear: { agi, totalTax, capitalLossCarryforward, ftcCarryforwards, palCarryforward },

  // Documents
  documents: [{ file, category, extracted, notes }],

  // Validation (if US-validation-report.md exists)
  validation: {
    returnName, preparer, date,
    checks: [{ form, line, description, sourceValue, returnValue, status, note }]
  }
}
```

---

## HTML Design Requirements

The dashboard must be a single self-contained HTML file (no external files needed).

### Overall Design
- **Professional Analytics UI:** Dark-mode (#0f172a background, #1e293b cards, #e2e8f0 text)
- **Goal:** Complete "Truth Source" — explain the tax outcome, show every document's impact, highlight what matters
- Works when opened via file:// protocol (no CORS issues)
- Responsive layout (works on MacBook screen)
- All data rendered from embedded JSON — no hardcoded HTML strings
- Include "Copy data as JSON" button for debugging
- Footer: "Generated by TaxBro on {date} | /taxbro-visualize"

### Layout: Sidebar + Main Panel

**Left sidebar (fixed, ~260px):**
- App title "TaxBro Insight" with tax year badge
- **Bottom Line:** Large, prominent "EST. REFUND" or "YOU OWE" amount with color (green refund / red owe)
- Navigation icons with labels and active state highlighting
- **Quick Stats:** 🔴 Critical count, ⚠️ Warning count, 📑 Docs processed count
- Confidence indicator: "Estimates use simplified bracket math — actual return will differ"

---

### Tab 1 — 📊 Dashboard (The Big Picture)

**Purpose:** Answer "how much do I get back / owe, and why?"

**Section A — Tax Waterfall (MOST IMPORTANT)**
Visual waterfall chart (CSS-based, no external libs) showing the flow:
```
Gross Income ($X)
  → minus Adjustments ($Y)
  = AGI ($Z)
  → minus Deduction ($D) [show which: standard or itemized]
  = Taxable Income ($T)
  → Regular Tax ($R) [show bracket breakdown]
  → + Additional Medicare Tax ($M) [0.9% × (wages − $250K)]
  → + NIIT ($N) [3.8% × min(NII, MAGI − $250K)]
  = Total Tax Liability ($L)
  → minus Total Payments & Credits ($P)
  = REFUND / OWE ($F)
```

Each step should be a horizontal bar with the dollar amount and a brief explanation.
Use green for income/credits, red for taxes/deductions, blue for the final result.

**Section B — Income Composition**
Doughnut or stacked bar showing:
- W-2 Wages (show % of total)
- Capital Gains (adjusted, show % of total)
- Interest (US + foreign)
- Dividends
- Rental Income
Each segment is labeled with dollar amount and percentage.

**Section C — Payments vs Tax**
Side-by-side comparison:
- Left: "What You Owe" (total tax)
- Right: "What You've Paid" (withholding + estimated + credits)
- Gap shown prominently as refund or amount due
Breakdown: W-2 withholding | Estimated payments | Excess SS | FTC | Child tax credit

---

### Tab 2 — 🧠 How Each Item Impacts Your Taxes

**Purpose:** For each major tax item, explain: what it is, what it costs or saves you, and what you need to do.

Render as expandable cards, each with:
- **Title** (e.g., "Mortgage Interest Cap")
- **Dollar Impact** — how much this costs or saves you in actual tax dollars (not just the deduction amount, but the tax effect: deduction × marginal rate)
- **Status** — 🔴/⚠️/✅/ℹ️
- **Explanation** — 2-3 sentences: what happened, why, what the IRS rule is
- **Action Required** — what the filer or preparer needs to do

**Items to include (derive from Issues & Flags + key knowledge graph sections):**

1. **Excess Social Security** — Amount: show excess. Impact: refundable credit. Action: claim on Schedule 3.
2. **RSU Cost Basis Adjustment** — Show: face 1099-B gain vs adjusted gain. Impact: difference in tax. Action: preparer must adjust.
3. **Mortgage Interest Cap (TCJA)** — Show: total interest, deductible portion, lost interest. Impact: lost deduction × marginal rate. Action: preparer computes ratio.
4. **SALT Cap** — Show: total property tax, cap, lost amount. Impact: lost deduction × marginal rate.
5. **FBAR / Form 8938** — Show: aggregate balances, thresholds. Impact: compliance (penalties if not filed). Action: file FinCEN 114 + attach 8938.
6. **PFIC Status** — Show: aggregate value, threshold, de minimis status. Impact: if over $50K, Form 8621 per fund. Action: confirm Dec 31 NAVs.
7. **Foreign Tax Credit** — Show: taxes paid, carryforward available, limitation fraction. Impact: dollar-for-dollar credit. Action: file Form 1116.
8. **NRE Interest (No FTC)** — Show: NRE interest amount. Impact: fully taxable with NO foreign tax credit. Explain why.
9. **Childcare / DCFSA** — Show: total paid, DCFSA exclusion, credit amount. Impact: DCFSA saves ~$X (DCFSA × marginal rate). Credit = $0 and why.
10. **Rental Income (India)** — Show: gross rent, depreciation, net income. Impact: taxable at marginal rate. Action: ADS 30-year depreciation.
11. **Additional Medicare Tax** — Show: wages over $250K threshold. Impact: 0.9% on excess.
12. **NIIT** — Show: NII amount, MAGI over $250K. Impact: 3.8% on lesser of NII or MAGI excess.
13. **Form 2210 Penalty** — Show: quarterly analysis, when payments were made. Impact: estimated penalty. Explain safe harbor.
14. **Qualified Dividend / LTCG Rate Benefit** — Show: amount taxed at preferential rates instead of ordinary. Impact: tax savings vs if all ordinary.
15. **Child Tax Credit (Phase-Out)** — Show: AGI, phase-out threshold ($400K MFJ), fully-eliminated threshold ($440K), computed credit amount (likely $0). Impact: at AGI above $440K, credit is fully phased out. Explain the math: reduces by $50 per $1,000 over $400K. Same applies to $500 ODC. **This is a mandatory callout** — always render even when the credit is $0, because it explains WHY it's $0.

Each card should show the **marginal tax rate context** — e.g., "At your marginal rate of 35%, this $X deduction saves $Y in tax."

**MANDATORY RULE for "not applicable" items:** When a credit or deduction is phased out
due to income level, DO NOT silently omit it. Always render a card explaining:
1. What the credit/deduction is
2. What the phase-out threshold is
3. Where AGI falls relative to the threshold
4. The result (e.g., "$0 credit — fully phased out")
This is essential context — the filer needs to understand what they're NOT getting and why.

---

### Tab 3 — 📑 1040 Worksheet (Line-by-Line)

**Purpose:** Every major line of Form 1040, with source tracking.

Render as a table:
| Line | Description | Amount | Status | Source / Calculation |
|------|-------------|--------|--------|---------------------|

Group into sections with visual separators:
- **Income** (Lines 1-9): Show each income line, how it was computed, which documents feed it
- **Adjustments** (Lines 10-11): HSA, educator, etc.
- **AGI** (Line 11): Prominent — show it's the sum
- **Deductions** (Lines 12-14): Show standard vs itemized comparison, which was used and why
- **Taxable Income** (Line 15): AGI minus deduction
- **Tax** (Line 16): Show bracket computation or note "from Tax Table / Qualified Dividends worksheet"
- **Credits** (Lines 19-21): FTC, child tax credit
- **Other Taxes** (Lines 23-24): Additional Medicare, NIIT, penalty
- **Payments** (Lines 25-33): Withholding, estimated, excess SS, FTC
- **Refund/Owe** (Line 34-37)

For each line:
- Status icon: ✅ (confirmed from document), ⚠️ (estimated/flagged), ❓ (needs preparer input)
- Calculation column: show exactly how the number was derived, e.g., "Amazon $194,192 + Meta $313,384"

---

### Tab 4 — 📋 Action Items & Issues

**Purpose:** Consolidated task list for the filer and preparer.

**Section A — Questions for User** (items marked ⚠️ needing user input):
Render as checklist cards with:
- What's needed
- Why it matters
- What happens if not resolved (dollar impact or compliance risk)

**Section B — Preparer Action Items** (items the CPA/tax software needs to handle):
- RSU basis adjustment
- Mortgage cap ratio computation
- Form 2210 penalty computation
- PFIC de minimis confirmation

**Section C — All Flags** from knowledge graph Issues table:
- Grouped by severity (🔴 first, then ⚠️, then ℹ️)
- Each with the detail text from the knowledge graph

---

### Tab 5 — 🌍 Global Assets

**Purpose:** Complete picture of international tax exposure.

**Section A — Foreign Account Map**
Table of all accounts with:
- Institution, country, type, owner
- Max balance (USD), Dec 31 balance (USD)
- Interest earned, TDS paid
- Status icon (data confirmed vs estimated)

**Section B — FBAR Threshold Progress Bar**
Visual progress bar: aggregate max balance vs $10,000 threshold
- Show green if under, red if over
- Label: "X times over threshold — FBAR REQUIRED"

**Section C — Form 8938 (FATCA) Threshold Progress Bars**
Two bars:
- Year-end total vs $100,000 (MFJ)
- Any-point total vs $150,000 (MFJ)

**Section D — PFIC Holdings**
Table of all funds with:
- Fund name, AMC, units, Dec 31 value (INR + USD)
- Redemptions/dividends status
- Row highlighting for funds over individual $25K threshold

**PFIC De Minimis Progress Bar:**
- Aggregate value vs $50,000 threshold
- Color: green if under (no filing needed), amber if close, red if over

**Section E — FTC Summary**
Table showing:
- Basket (passive/general)
- Foreign income, foreign tax paid, carryforward available
- Net credit this year
- Limitation fraction

---

### Tab 6 — 📄 Documents

**Purpose:** Source document registry with extraction status.

Searchable/filterable table:
- File name (with category badge)
- Extraction status (✅ extracted / ⚠️ partial / ❌ not extracted / 🔒 unreadable)
- Key values extracted (one-line summary)
- Category filter buttons: W-2, 1099, Foreign, Childcare, Mortgage, Prior Return, Other

---

### Tab 7 — 🔄 Return Δ (only if US-validation-report.md exists)

**Purpose:** Compare filed return values against source document values.

Table:
| Form/Line | Description | Source Value | Return Value | Delta | Status |

Status badges:
- ✅ PASS (values match)
- ⚠️ FLAG (values differ but may be explainable)
- 🔴 MISMATCH (values differ significantly)

Highlight rows where delta > $100 or status is MISMATCH.
Show summary: X of Y lines match, N flagged, M mismatched.

---

## Code Quality Requirements

- All data embedded as `TAX_DATA` JSON in `<script>` block
- CSS included inline (Tailwind utility classes via CDN link OR custom CSS — either works)
- JS for tab switching, search, expand/collapse — all inline
- No external file dependencies beyond CDN CSS (optional)
- Works with file:// protocol
- Responsive (min-width: 1024px optimized, scales down gracefully)
- All monetary values formatted with `$` and commas: `$507,576.42`
- All percentages formatted with 1 decimal: `35.0%`
- Charts rendered with CSS (colored divs with proportional widths) — no Chart.js or D3 required
- Tooltips on hover for abbreviated content

---

## Privacy Note
The HTML file contains financial figures from the knowledge graph. Written to SOURCE_FOLDER
(not the taxbro repo). Never commit to git. Treat with same care as tax documents.
