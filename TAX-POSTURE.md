# Tax Posture — Reference Guide

A PII-free explanation of the tax situation TaxBro was built around.
This describes the _type_ of filer, the _categories_ of income and reporting,
and the IRS rules and limits that apply — not actual numbers.

---

## Filing Status: Married Filing Jointly (MFJ)

**Why MFJ?** One spouse has high W-2 income (tech, with RSUs). The other spouse
has no US earned income. MFJ almost always wins here because:

- The standard deduction is doubled ($29,200 for 2024, $30,000 for 2025)
- Tax brackets are wider — the 24% bracket extends to $383,900 vs $191,950 (single)
- Several credits and deductions phase out at higher AGI thresholds for MFJ
- FBAR and Form 8938 thresholds are higher for MFJ ($100K/$200K vs $50K/$75K)

The only scenario where MFS (Married Filing Separately) might help is when one
spouse has massive medical expenses or student loan considerations. That does not
apply here. MFJ is correct.

---

## Income Categories

### 1. W-2 Wages (Primary Income)

Single employer, big tech. The W-2 is the anchor of the return. Key boxes:

| Box | What it is | Why it matters |
|-----|-----------|----------------|
| Box 1 | Total compensation (salary + RSU vesting + bonuses) | This is AGI starting point |
| Box 2 | Federal tax withheld | Credited against total tax on Line 24 |
| Box 4 | Social Security tax | Check against SS wage base cap |
| Box 12-W | Employer HSA contribution (cafeteria plan) | Reduces Box 1; reported on Form 8889 |
| Box 12-D | 401(k) elective deferral | Pre-tax; reduces Box 1 |
| Box 12-DD | Health insurance cost (informational) | No tax impact |
| Box 14 | RSU income (informational) | Already included in Box 1 — don't double count |

**Social Security wage base (2024):** $168,600. Max SS tax = 6.2% x $168,600 = $10,453.20.
If you have multiple W-2s and combined wages exceed the cap, you get an excess SS
credit on Schedule 3. With a single employer, this doesn't apply — the employer stops
withholding at the cap.

**2025 SS wage base:** $176,100. Max SS tax = $10,918.20.

### 2. RSU (Restricted Stock Units)

RSUs are the largest component of compensation. How they work for taxes:

```
Vest date:  shares vest -> FMV on vest date = ordinary income (Box 1)
            employer withholds shares for taxes ("sell to cover")

Sale date:  when you actually sell the shares
            cost basis = FMV on vest date (what was already taxed as income)
            gain/loss = sale price - vest price
            holding period: vest date -> sale date
              < 1 year = short-term capital gain (ordinary rates)
              > 1 year = long-term capital gain (0/15/20% + 3.8% NIIT)
```

**The 1099-B trap:** Brokerages report RSU sales with $0 cost basis ("non-covered").
This makes it look like you have massive capital gains equal to the full sale proceeds.
You MUST use an adjusted basis (= FMV at vest, from the supplemental gain/loss report).
Without adjustment, your return would show hundreds of thousands in phantom gains.

**Strategy note:** Hold vested RSUs >1 year before selling to get LTCG rates (20% + 3.8%
NIIT = 23.8%) instead of ordinary income rates (up to 37%). But this introduces
concentration risk — you're holding a single stock. Tax savings vs. diversification
is a personal risk decision.

### 3. Capital Gains (Brokerage)

The Fidelity 1099-B reports across four boxes:

| Box | Type | Basis reported? |
|-----|------|----------------|
| A | Short-term, covered | Yes — broker reports basis to IRS |
| B | Short-term, non-covered (RSUs) | No — you must provide adjusted basis |
| D | Long-term, covered | Yes |
| E | Long-term, non-covered (RSUs) | No — you must provide adjusted basis |

Form 8949 is required whenever basis needs adjustment. Each RSU lot needs its own line.

**Net Investment Income Tax (NIIT):** 3.8% surtax on investment income when MAGI
exceeds $250,000 (MFJ). This applies to dividends, capital gains, interest, rental
income. At this income level, NIIT always applies.

### 4. Interest & Dividends

**US sources:** Marcus (Goldman Sachs savings), Chase, Fidelity cash sweep. All
reported on 1099-INT. No withholding. Goes on Schedule B.

**Schedule B Part III:** Required when you have foreign accounts. You must check "Yes"
and disclose the countries. This is separate from FBAR — it's on the tax return itself.

### 5. Canadian Employment Income (T4)

Short-duration Canadian assignment (Amazon Canada). Reported on the US return as
foreign earned income. The key interaction:

```
Canadian wages (CAD -> USD at IRS annual rate)
  -> included in US AGI (worldwide income)
  -> Canadian tax withheld -> Form 1116 FTC (General basket)
  -> Canadian T1 filed as non-resident
  -> CRA Notice of Assessment confirms final Canadian tax
```

**Exchange rate:** Use IRS Publication 54 annual average rate, not spot rate.
The rate used on the return should match what the preparer used, and it may differ
slightly from the Bank of Canada or Federal Reserve rate.

**FTC vs. Exclusion:** Could theoretically use the Foreign Earned Income Exclusion
(Form 2555) instead of FTC, but FTC is better here — the Canadian tax rate is high
enough that the credit eliminates double-taxation dollar-for-dollar, and you don't
lose the income from your US bracket calculations.

---

## Retirement Accounts

### 401(k)

**2024 limit:** $23,000 employee elective deferral (under 50). $23,500 for 2025.
The employer match does not count against this limit. Total combined limit
(employee + employer) = $69,000 for 2024, $70,000 for 2025.

**Check:** Is the full $23,000 being contributed? W-2 Box 12-D shows the actual
deferral. If it's less than $23,000, there's room to increase. At high marginal
rates (32-37%), every dollar of pre-tax 401(k) contribution saves $0.32-0.37 in
current-year federal tax.

**Mega Backdoor Roth:** If the employer plan allows after-tax contributions + in-plan
Roth conversion, you can contribute up to the $69,000 total limit minus employee
deferral minus employer match. This gets money into a Roth account without the
income limits that block direct Roth contributions.

```
Example (2024 limits):
  Total 401(k) limit:              $69,000
  - Employee pre-tax deferral:     $23,000
  - Employer match (varies):       ~$X,XXX
  = Room for after-tax + convert:  $46,000 - match

  This is "mega backdoor Roth" — check if your plan supports it.
  Amazon's plan historically has supported this.
```

**2025:** Total limit = $70,000. Employee = $23,500.

### Traditional IRA -> Backdoor Roth Conversion

**Why backdoor?** At this income level, direct Roth IRA contributions are not
allowed (2024 MAGI limit: $240,000 MFJ). Traditional IRA contributions are not
deductible (covered by employer plan + income too high). But the backdoor works:

```
Step 1: Contribute $7,000 to Traditional IRA (non-deductible, 2024 limit)
        -> $7,500 for 2025
Step 2: Convert to Roth IRA (immediately or shortly after)
Step 3: Report on Form 8606

If IRA balance was $0 before contribution, conversion is nearly tax-free
(only gains between contribution and conversion are taxable — usually ~$0-$11).
```

**Form 8606 is critical.** It tracks your non-deductible IRA basis. Without it, the
IRS treats the conversion as fully taxable. The 1099-R shows the gross distribution
with code 2 (early distribution, exception applies). Form 8606 reduces the taxable
amount to near zero.

**Contribution deadline:** You can make 2025 IRA contributions until April 15, 2026.
If you haven't contributed for 2025 yet, there's still time ($7,500 limit).

### HSA (Health Savings Account)

**2024 family limit:** $8,300 (self+family HDHP coverage). $8,550 for 2025.
The W-2 Box 12-W amount includes both employer and employee contributions made
through payroll. You can also contribute directly outside payroll up to the limit.

```
Check: Box 12-W + direct contributions <= $8,300 (2024)?
If under the limit, you can contribute more.

HSA triple tax advantage:
  1. Contributions are pre-tax (reduce AGI)
  2. Growth is tax-free
  3. Withdrawals for qualified medical expenses are tax-free

At 37% marginal rate, every $1,000 of additional HSA contribution saves $370.
```

**Form 8889 is ALWAYS required** when you have an HSA — even if you only took
distributions and all were for qualified medical expenses. The form reports
contributions, distributions, and confirms no excess. Missing this form is a
common preparer error.

**1099-SA:** Reports HSA distributions. Code 1 = normal distribution. If all
distributions were for qualified medical expenses, tax = $0, but the form is still
required.

---

## Foreign Accounts & Reporting

### FBAR (FinCEN 114)

**Who must file:** Any US person with foreign financial accounts whose aggregate
maximum value exceeds $10,000 at any point during the year. "Aggregate" means you
sum the max balance of every foreign account (even if they peak on different days).

**What counts:** Bank accounts (savings, checking, FDs), brokerage accounts,
mutual fund accounts, insurance policies with cash value, pension accounts.

**Deadline:** April 15 (auto-extended to October 15). Filed electronically via
BSA E-Filing, NOT with your tax return. No tax is owed — it's an information report.

**Penalties for non-filing:** Up to $10,000 per non-willful violation, per account,
per year. Willful violations: up to $100,000 or 50% of account balance.

In this posture, FBAR is required every year. The aggregate foreign account max
comfortably exceeds $10,000.

### Form 8938 (FATCA — Statement of Specified Foreign Financial Assets)

**MFJ US resident thresholds:**
- Year-end aggregate > $100,000, OR
- Any-point aggregate > $150,000

**What counts:** Everything on FBAR, plus foreign stocks/securities held directly
(not through a US broker), interests in foreign entities, foreign-issued life
insurance. Does NOT include accounts at US financial institutions.

**Filed with the return** (not separate like FBAR). Duplicates much of FBAR but
has different thresholds and goes to the IRS (FBAR goes to FinCEN/Treasury).

### NRE vs NRO Accounts (India-specific)

| Feature | NRE (Non-Resident External) | NRO (Non-Resident Ordinary) |
|---------|---------------------------|---------------------------|
| India tax on interest | Exempt | Taxable (TDS at ~31.2%) |
| US tax on interest | Fully taxable | Fully taxable |
| FTC available? | No (no India tax paid) | Yes (TDS paid = FTC) |
| Repatriation | Freely repatriable | Up to $1M/year with CA cert |

**Key trap:** NRE interest is exempt in India, which is great. But US residents owe
US tax on worldwide income — NRE interest is fully taxable on the US return. And
because India charges no tax on it, there's no FTC offset. This is "tax nowhere vs.
taxed once" turning into "taxed once with no credit."

NRO interest has TDS withheld in India, which creates an FTC credit in the passive
basket on Form 1116.

---

## PFIC (Passive Foreign Investment Company)

### What is a PFIC?

Any Indian (or other foreign) mutual fund is a PFIC for US tax purposes. This
includes equity funds, balanced funds, debt funds, liquid funds, gold funds, and
fund-of-funds. There is no exception for actively-managed funds.

### Why it matters

PFICs have the worst tax treatment in the US tax code:

- **Default (excess distribution) method:** Any gain on sale or "excess distribution"
  is allocated across the holding period, taxed at the highest marginal rate for each
  year, plus an interest charge. This can result in effective tax rates of 50%+.
- **QEF election:** Requires the fund to provide an "annual information statement"
  with ordinary earnings and net capital gains. Indian mutual funds don't provide this.
  Practically unavailable.
- **Mark-to-Market (MTM) election:** Recognize annual unrealized gain/loss as ordinary
  income. Possible but must be elected timely (by the return due date for the first
  year you hold the PFIC). Once elected, can't undo.

### Annual reporting requirement (Section 1298(f))

Even if there are NO distributions and NO sales, you must file Form 8621 for
each PFIC if:

- Any single fund value > $25,000, OR
- Aggregate PFIC value > $25,000

**In this posture:** ~10 Indian mutual funds with estimated aggregate value ~$35K.
Form 8621 is required for each fund every year.

### De minimis thresholds (2024)

| Test | Threshold |
|------|-----------|
| Individual fund year-end value | $25,000 |
| Aggregate PFIC year-end value | $25,000 |

If both are below threshold, annual reporting is waived. But you still need to
file Form 8621 in any year you have a disposition (sale/redemption) or excess
distribution regardless of value.

### What you need for each Form 8621

- Fund name, ISIN, country
- December 31 NAV per unit (from AMFI website for Indian funds)
- Number of units held on December 31
- Year-end value in USD (units x NAV / IRS exchange rate)
- Any distributions received during the year
- Any dispositions (sales/redemptions)
- Election status (default/QEF/MTM)

### Strategy considerations

**Should you sell everything?** Holding PFICs with no election means the excess
distribution regime applies on eventual sale. If the funds have minimal unrealized
gains, selling now (taking the PFIC hit on small gains) and reinvesting through a
US-based international fund eliminates future PFIC headaches. The longer you hold,
the larger the potential excess distribution penalty.

**Can you elect MTM going forward?** Yes, for funds that are "marketable" (traded
on a qualifying exchange — Indian mutual funds listed on exchanges qualify). The
election treats unrealized gain as ordinary income each year but prevents the
excess distribution regime. Must be made by the due date (with extensions) of the
return for the first year you want it to apply.

---

## Foreign Tax Credit (Form 1116)

### How it works

When you pay tax to a foreign country on income that's also taxed by the US, you
get a credit (dollar-for-dollar reduction in US tax) to prevent double taxation.
The credit is limited to the US tax attributable to the foreign income.

### Baskets

Income must be sorted into "baskets" — each has its own Form 1116 and its own
limitation calculation:

| Basket | Income types in this posture |
|--------|------------------------------|
| **General** | Canadian employment income (T4 wages) |
| **Passive** | India NRO interest, India rental income |

**NRE interest goes in passive basket but has NO FTC** (India doesn't tax it).
It increases the passive income base, which actually helps the FTC limitation
calculation (more room to use passive FTC), but there's no credit to use.

### Carryover rules

- Unused FTC carries forward 10 years (back 1 year, if elected)
- Each basket carries separately
- Large general basket carryforwards can accumulate if foreign tax rates exceed
  the effective US rate on that income (common with Canadian wages at ~45% combined
  rate vs US effective rate on marginal income)

### Common issues

- **Timing mismatch:** India uses April-March fiscal year, US uses calendar year.
  TDS paid in January-March of the following year may need to be accrued to the
  prior calendar year (accrual vs. cash basis election on Form 1116).
- **Exchange rate:** Use the IRS rate for the tax payment date (cash basis) or the
  average annual rate (accrual basis). Be consistent.

---

## Foreign Rental Property (Schedule E)

### How it's reported

Foreign rental property is reported on Schedule E, just like US rental property.
The main differences are currency conversion and depreciation rules.

### Depreciation

| Property type | Recovery period | Convention |
|--------------|----------------|------------|
| Residential rental (foreign) | 30 years (not 27.5) | Mid-month |
| Residential rental (US) | 27.5 years | Mid-month |

**Yes, foreign residential rental uses 30 years, not 27.5.** This is a common
error. ADS (Alternative Depreciation System) applies to foreign property,
which prescribes 30 years for residential.

**Wait — check this:** The 2024 return uses 27.5 years. The IRS requires ADS
(30 years) for property used predominantly outside the US. Verify with preparer
whether 27.5 or 30 is correct. The difference matters over the life of the asset.

### Cost basis conversion

Convert the INR purchase price to USD at the exchange rate on the date of
acquisition (deed registration date or possession date, depending on convention).
This becomes the permanent USD basis — you do NOT reconvert each year.

### Land vs. improvement split

Only the improvement (building) portion is depreciable. Land is not.
You need documentation supporting the split — a builder's cost sheet showing
land vs. construction allocation works.

### Passive Activity Loss Rules (Form 8582)

Rental income is generally passive. If total rental produces a loss, it may be
suspended under passive activity rules unless:

- AGI < $100,000 AND you actively participate → up to $25,000 loss allowed
- AGI > $150,000 → the $25,000 allowance is fully phased out

At this income level, rental losses are always suspended and carried forward
until the property is sold or you have passive income to offset.

---

## Estimated Tax Payments (Form 1040-ES)

### The problem

With high W-2 income plus large capital gains from RSU sales, W-2 withholding
alone often falls short of the total tax liability. The IRS expects taxes to be
paid throughout the year — not as a lump sum at filing. When you underpay, they
charge a penalty (Form 2210) that functions like interest on the shortfall.

### What happened (2024 tax year — lessons learned)

The 2024 return had a balance due at filing. No quarterly estimated payments
were made during 2024. The entire shortfall — driven by capital gains, foreign
income, and NIIT — was paid at filing plus a Form 2210 underpayment penalty.

Late in 2025, two large estimated tax payments were made ($25K + $50K in December
2025 via IRS Direct Pay), but these are for the **2025 tax year**, not 2024. They
don't retroactively fix the 2024 underpayment — and paying them all in Q4 2025
means Q1-Q3 2025 are still underpaid (the penalty accrues per-quarter).

### How the penalty works (Form 2210)

The penalty is essentially interest on what you should have paid each quarter
but didn't. The IRS computes it quarter by quarter:

```
For each quarter:
  Required payment = 25% of required annual payment
  Required annual payment = lesser of:
    (a) 90% of current-year tax, or
    (b) 110% of prior-year tax (110% because AGI > $150K)

  If you paid less than required by the quarter's due date:
    Penalty = shortfall x IRS underpayment rate x days late

  IRS underpayment rate (2024): ~8% annually (~2% per quarter)
  This rate changes quarterly — tied to federal short-term rate + 3%.
```

The penalty is NOT deductible. It's pure waste.

### Safe harbors — how to avoid the penalty entirely

You owe zero penalty if you meet **either** safe harbor:

| Safe harbor | Rule | When it's easier |
|-------------|------|-----------------|
| **90% of current year** | Total payments (withholding + estimated) >= 90% of 2025 tax | When income is predictable and you can estimate liability |
| **110% of prior year** | Total payments >= 110% of 2024 total tax (Line 24 of 2024 Form 1040) | When prior year tax is known and income may vary |

**The 110% rule is the easy path.** You know exactly what the 2024 tax was.
Multiply by 1.10. Divide by 4. Pay that amount each quarter. Done — no penalty
regardless of what 2025 income turns out to be.

```
Example calculation:
  2024 total tax (Form 1040 Line 24):   $X
  110% safe harbor:                      $X x 1.10 = $Y
  Per-quarter required payment:          $Y / 4 = $Z
  Minus W-2 withholding per quarter:     ~(Box 2 / pay periods) x periods-in-quarter
  Estimated payment needed per quarter:  $Z - withholding credit for that quarter

  If W-2 withholding alone exceeds $Y, no estimated payments needed.
```

### Annualized income installment method (Schedule AI)

If income is lumpy (big RSU vest in Q4, large capital gain in one quarter),
the annualized method can reduce or eliminate the penalty for earlier quarters.
Instead of requiring 25% of annual tax per quarter, it calculates required
payment based on income actually received through each period:

| Period | Annualization factor | Meaning |
|--------|---------------------|---------|
| Q1 (Jan-Mar) | x4 | Annualizes 3 months to 12 |
| Q2 (Jan-May) | x2.4 | Annualizes 5 months to 12 |
| Q3 (Jan-Aug) | x1.5 | Annualizes 8 months to 12 |
| Q4 (Jan-Dec) | x1 | Full year, no annualization |

This is helpful when most RSU vests or sales happen late in the year — it
means you legitimately owe less for Q1-Q2 and more for Q3-Q4.

**However:** The preparer must calculate this. It's not automatic. And it
only helps with the penalty calculation — you still owe the same total tax.

### Payment schedule and due dates

| Quarter | Period covered | Due date | Covers income from |
|---------|---------------|----------|--------------------|
| Q1 | Jan 1 - Mar 31 | April 15 | Salary, RSU vests, cap gains in Q1 |
| Q2 | Apr 1 - May 31 | June 15 | Note: only 2 months, not 3 |
| Q3 | Jun 1 - Aug 31 | September 15 | Peak RSU vest months often here |
| Q4 | Sep 1 - Dec 31 | January 15 (next yr) | Year-end RSU vests, bonuses |

**Important:** Q2 covers only 2 months but requires the same 25% payment as the
other quarters (under the regular method). This is an IRS quirk.

### 2025 action plan — avoiding a repeat

```
Step 1: Calculate 110% safe harbor
  - Pull 2024 total tax from Line 24 of the filed return
  - Multiply by 1.10
  - This is the TOTAL you need paid by Jan 15, 2026

Step 2: Estimate W-2 withholding for 2025
  - Check a recent pay stub for YTD federal withholding
  - Extrapolate to full year
  - If W-2 withholding alone exceeds the 110% number: done, no action needed

Step 3: If shortfall exists, calculate quarterly payments
  - Shortfall = 110% number - projected W-2 withholding
  - Divide by 4 (or by remaining quarters if some have passed)
  - Pay via IRS Direct Pay (irs.gov/directpay) before each due date
  - Keep confirmation numbers

Step 4: Consider increasing W-2 withholding
  - File a new W-4 with your employer requesting additional withholding
  - W-4 Line 4(c): "Extra withholding" — add a flat amount per pay period
  - Advantage: W-2 withholding is treated as paid evenly throughout the year,
    even if you increase it in Q4. Estimated payments are dated to when paid.
    This means a W-4 adjustment in September "covers" Q1-Q2 retroactively.
  - This is the BEST strategy for avoiding underpayment penalties.
```

**The W-4 trick is key:** If you realize in September that you'll be short,
increasing your W-4 withholding for the remaining pay periods is better than
making estimated payments, because the IRS treats withholding as if it were
paid evenly across all four quarters. A $20K extra withholding in Q4 via W-4
is treated as $5K per quarter. A $20K estimated payment in Q4 only counts for Q4.

### How to pay estimated taxes

| Method | How |
|--------|-----|
| IRS Direct Pay | irs.gov/directpay — free, from checking account |
| EFTPS | eftps.gov — requires enrollment, allows scheduling |
| IRS2Go app | Mobile payments |
| Credit/debit card | Via approved processors — fees apply |

Use payment type "1040-ES" and select the correct tax year (2025).
Keep all confirmation numbers for your records.

---

## Optimization Checklist — Things to Verify Each Year

### Retirement accounts — are you maxing out?

```
+----------------------------------+----------+----------+
| Account                          |   2024   |   2025   |
+----------------------------------+----------+----------+
| 401(k) employee deferral         | $23,000  | $23,500  |
| 401(k) total (emp + employer)    | $69,000  | $70,000  |
| Backdoor Roth IRA                |  $7,000  |  $7,500  |
| HSA (family)                     |  $8,300  |  $8,550  |
+----------------------------------+----------+----------+
| Maximum tax-advantaged savings    | $84,300  | $86,050  |
| (including mega backdoor if      |          |          |
|  plan allows)                    |          |          |
+----------------------------------+----------+----------+
```

**Action items:**
- [ ] Verify 401(k) deferral hits $23,500 for 2025 (check last pay stub)
- [ ] Check if mega backdoor Roth is available in plan — if so, calculate room
- [ ] Contribute $7,500 to Traditional IRA for 2025 and convert to Roth (before April 15, 2026)
- [ ] Verify HSA contributions for 2025 hit $8,550 (W-2 Box 12-W + direct)
- [ ] File Form 8889 even in years with zero distributions

### Tax-loss harvesting

At high marginal rates, realizing losses to offset gains saves real money.
Long-term losses offset LTCG (taxed at 20% + 3.8% NIIT = 23.8%).
Short-term losses offset STCG (taxed at ordinary rates, up to 37% + 3.8% = 40.8%).
Net losses up to $3,000/year offset ordinary income.

Watch the wash sale rule: no buying "substantially identical" securities within
30 days before or after the sale. Applies across all accounts (brokerage, IRA, HSA).

### Charitable giving

At high income, consider:
- **Donor-Advised Fund (DAF):** Bunch multiple years of giving into one year to
  exceed the standard deduction and itemize. Contribute appreciated stock directly
  to avoid capital gains.
- **Qualified Charitable Distribution (QCD):** Not applicable yet (must be 70.5+).

### State tax

Washington state has no income tax. However:
- **WA capital gains tax:** 7% on long-term capital gains exceeding $270,000 (2024).
  This is a significant additional tax on large RSU sales. The threshold is per return,
  not per person. Only LTCG counts — STCG is exempt.
  2025 threshold: $280,950.
- Standard deduction of $270,000 means gains below this are exempt.

### PFIC strategy

Every year you hold Indian mutual funds, you accumulate reporting burden (10x Form 8621)
and potential excess distribution exposure. Consider:

- [ ] Evaluate cost of selling all PFIC holdings and reinvesting in US-domiciled
      international funds (e.g., Vanguard FTSE Emerging Markets ETF)
- [ ] If keeping, file Form 8621 for every fund every year
- [ ] Consider MTM election if funds qualify and you want to avoid excess distribution
      regime on eventual sale

### Estimated tax payments — don't repeat 2024

The 2024 return incurred an underpayment penalty because no quarterly estimated
payments were made. This is avoidable:

- [ ] Calculate 110% of 2024 total tax (Line 24) — this is the 2025 safe harbor
- [ ] Check: does projected 2025 W-2 withholding alone exceed that number?
- [ ] If not, calculate per-quarter shortfall and set up IRS Direct Pay
- [ ] **Better option:** Adjust W-4 to increase per-paycheck withholding
      (withholding is treated as paid evenly across all quarters — retroactive fix)
- [ ] Set calendar reminders: Apr 15, Jun 15, Sep 15, Jan 15

### FBAR / Form 8938 maintenance

These are required every year as long as foreign accounts exist:

- [ ] Collect max balance and Dec 31 balance for every foreign account
- [ ] Include spouse's accounts (MFJ = both spouses' accounts aggregate)
- [ ] FBAR deadline: April 15 (auto-extended to Oct 15)
- [ ] Form 8938 filed with return

---

## Forms Checklist

Every form that applies to this tax posture, every year:

| Form | Why | Frequency |
|------|-----|-----------|
| 1040 | Federal return | Annual |
| Schedule B | Interest + dividends + foreign account disclosure | Annual |
| Schedule D + 8949 | Capital gains from RSU sales | Annual (while selling) |
| Schedule E | Foreign rental property | Annual (while renting) |
| Form 8606 | Backdoor Roth IRA conversion | Annual (while converting) |
| Form 8889 | HSA contributions and distributions | Annual (while HSA exists) |
| Form 1116 (General) | Canadian tax credit | Annual (while Canadian income) |
| Form 1116 (Passive) | India tax credit (NRO TDS, rental) | Annual (while India income) |
| Form 8621 (x10) | PFIC annual reporting per fund | Annual (while holding PFICs) |
| Form 8582 | Passive activity loss limitation | Annual (while rental loss suspended) |
| FinCEN 114 (FBAR) | Foreign bank account report | Annual (while accounts exist) |
| Form 8938 | FATCA foreign asset statement | Annual (if over threshold) |
| Form 2210 | Underpayment penalty calculation | Annual (verify safe harbor) |
| WA excise tax | Washington capital gains tax | Annual (if LTCG > $270K) |
| Canada T1 | Canadian non-resident return | Annual (while Canadian income) |
| India ITR | India non-resident return | Annual (while India income) |

---

## Key IRS Limits Reference (2024 / 2025)

| Item | 2024 | 2025 |
|------|------|------|
| Standard deduction (MFJ) | $29,200 | $30,000 |
| SS wage base | $168,600 | $176,100 |
| 401(k) employee deferral | $23,000 | $23,500 |
| 401(k) total limit | $69,000 | $70,000 |
| IRA contribution | $7,000 | $7,500 |
| Roth IRA MAGI limit (MFJ) | $240,000 | $246,000 |
| HSA family limit | $8,300 | $8,550 |
| NIIT threshold (MFJ) | $250,000 | $250,000 |
| FBAR threshold | $10,000 | $10,000 |
| Form 8938 year-end (MFJ) | $100,000 | $100,000 |
| Form 8938 any-point (MFJ) | $150,000 | $150,000 |
| Passive loss phase-out start | $100,000 | $100,000 |
| Passive loss phase-out complete | $150,000 | $150,000 |
| PFIC de minimis (per fund) | $25,000 | $25,000 |
| PFIC de minimis (aggregate) | $25,000 | $25,000 |
| WA capital gains threshold | $270,000 | $280,950 |
| Child tax credit | $2,000 | $2,000 |
| LTCG rate (high bracket) | 20% | 20% |
| Additional Medicare (>$200K) | 0.9% | 0.9% |

---

## TL;DR

High-income MFJ filer in Washington state. One W-2 with large RSU component.
No state income tax but WA capital gains tax applies on large LTCG. Backdoor
Roth and HSA are active. Foreign complexity from India (NRE/NRO accounts, rental
property, mutual funds = PFICs) and Canada (short assignment, FTC). Around 15
forms required annually. Underpayment penalty hit in 2024 — must set up quarterly
estimated payments or adjust W-4 withholding for 2025. Main optimization levers:
max retirement contributions (401k + mega backdoor + IRA + HSA), fix estimated tax
payments, PFIC exit strategy, tax-loss harvesting on RSU lots, and ensuring the
FTC carryforward doesn't expire unused.
