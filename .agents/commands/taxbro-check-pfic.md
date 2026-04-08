Analyze Indian (or other foreign) mutual fund holdings for PFIC compliance.

Background:
- Indian mutual funds are almost certainly PFICs (Passive Foreign Investment Companies)
- Form 8621 must be filed per PFIC fund per year (with exceptions)
- De minimis exception: aggregate value of all PFIC shares ≤ $50,000 at year-end AND no prior QEF/MTM elections
- Three methods: (1) Default excess distribution, (2) QEF election, (3) Mark-to-Market election

Steps:
1. Get source folder from $ARGUMENTS or .current-session.

2. Read CLAUDE.md for context on who holds the mutual funds (primary filer vs spouse) and any prior elections.

3. Find mutual fund statement files (CAMS CAS, KARVY/KFintech statements, or similar).

4. For each fund found, extract:
   - Fund name and fund house (AMC)
   - ISIN or scheme code
   - Units held as of December 31
   - NAV (Net Asset Value) as of December 31
   - Total value in INR as of December 31
   - Any redemptions/withdrawals during the year (distributions = taxable events under default method)
   - Any dividends paid/reinvested during the year

5. Convert total portfolio value to USD (IRS Pub 54 average rate or year-end rate).

6. Determinations:
   a. De minimis check: if total USD value ≤ $50,000 AND no prior QEF/MTM elections → Form 8621 may not be required (flag for confirmation with tax preparer)
   b. If any redemptions occurred → excess distribution calculation needed for each fund with redemptions
   c. If no redemptions AND no distributions AND using default method → no annual income to report, but Form 8621 still required if above de minimis

7. Election analysis:
   - If no elections were made in prior years: default (excess distribution) method applies
   - Switching to QEF requires a "purging election" — significant tax catch-up cost; flag if considering
   - MTM election: simpler annually but requires unrealized gain/loss each year

8. Write output to {SOURCE_FOLDER}/TAXBRO/US-pfic-summary.md:
   - Table: Fund Name | AMC | Units | Dec 31 NAV (INR) | Dec 31 Value (INR) | Dec 31 Value (USD) | Redemptions This Year | Form 8621 Required
   - Total portfolio USD value
   - De minimis determination
   - Recommended action per fund
   - Election status and recommendation

9. Append an entry to `{SOURCE_FOLDER}/TAXBRO/agent-log.md` recording: agent-id (e.g., `gemini-2.0-flash`), skill-name, status (complete/partial/failed), artifacts written, key findings, and suggested next steps.

IMPORTANT: All output goes to {SOURCE_FOLDER}/TAXBRO/US-pfic-summary.md only.
Never write financial data to ~/claude/taxbro/.
