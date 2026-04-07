Analyze all W-2 forms in the source folder.

Steps:
1. Get source folder from $ARGUMENTS or .current-session.

2. Find all W-2 PDF files (filenames containing "W2", "W-2", "w2", or described as W-2 in CLAUDE.md).

3. Read each W-2 and extract:
   - Employer name
   - Box 1: Wages, tips, other compensation
   - Box 2: Federal income tax withheld
   - Box 4: Social Security tax withheld
   - Box 6: Medicare tax withheld
   - Box 10: Dependent care benefits (DCFSA)
   - Box 12 codes (especially: D=401k, W=HSA employer contrib, DD=health insurance)
   - Box 16/17: State wages and state tax withheld

4. Calculations:
   a. Sum all Box 4 values across all W-2s.
      - 2025 SS wage base: $176,100; max employee SS = $10,918.20
      - If sum > $10,918.20 → flag excess SS withheld; calculate refund amount = sum - $10,918.20
      - This excess is a refundable credit on Form 1040 Line 11
   b. Sum Box 10 amounts → report for Form 2441 coordination
   c. Sum Box 12 code W (HSA employer contributions) → needed for HSA limit check
   d. Check if CLAUDE.md notes spouse has no earned income → flag Form 2441 risk

5. Write output to {SOURCE_FOLDER}/TAXBRO/w2-summary.md:
   - Table: Employer | Box1 Wages | Box2 Fed Withheld | Box4 SS | Box10 DCFSA
   - Excess SS calculation (if applicable)
   - Form 2441 earned-income flag (if applicable)
   - Any other notable Box 12 items

IMPORTANT: This file IS written to the source TAXBRO folder (financial data belongs there, not here).
Never write financial data to ~/claude/taxbro/.
