Initialize TaxBro with a source tax documents folder.

Usage: /taxbro-init /path/to/tax/folder/

Steps:
1. The source folder path is: $ARGUMENTS
   If no argument was provided, ask the user: "Please provide the path to your tax documents folder."

2. Verify the folder exists. If it doesn't, tell the user and stop.

3. Write the resolved absolute path to ~/claude/taxbro/.current-session (single line, no trailing newline).
   This file is gitignored and contains only the path — no PII.

4. Read {SOURCE_FOLDER}/CLAUDE.md if it exists. Summarize the filer's situation in a brief paragraph (no PII details, just structural context like "MFJ filer, dual W-2, India/Canada cross-border").

5. List all files and subdirectories found in the source folder. Group them by likely category:
   - W-2s
   - 1099s (B, DIV, INT, SA, etc.)
   - 1098s (mortgage interest)
   - Childcare documents
   - Foreign account statements
   - Interest certificates
   - Mutual fund statements
   - AIS/TIS (India)
   - Other
   Flag any expected document types that appear to be missing based on the CLAUDE.md context.

6. Create {SOURCE_FOLDER}/TAXBRO/ directory if it doesn't exist.

7. Write a brief session log entry to {SOURCE_FOLDER}/TAXBRO/session-notes.md:
   - Date initialized
   - Document count discovered
   - Any missing document flags
   Do NOT write any personal data, names, account numbers, or financial figures to this file — structural notes only.

8. Tell the user: "TaxBro initialized. Source folder loaded with N documents. Run /tax-checklist to see full status."

IMPORTANT: Never write any PII, financial data, or document contents to ~/claude/taxbro/ or any file within it.
