Initialize TaxBro with a source tax documents folder.

Usage: /taxbro-init /path/to/tax/folder/

Steps:
1. The source folder path is: $ARGUMENTS
   If no argument was provided, ask the user: "Please provide the path to your tax documents folder."

2. Verify the folder exists. If it doesn't, tell the user and stop.

3. Resolve the real path:
   - Run `realpath "$ARGUMENTS"` (or `python3 -c "import os,sys; print(os.path.realpath(sys.argv[1]))" "$ARGUMENTS"`) to
     resolve any symlinks in the folder path itself.
   - If the folder is a symlink, note the resolved target to the user: "Note: folder is a symlink → resolves to [real path]"
   - Write the RESOLVED absolute path to ~/claude/taxbro/.current-session (single line, no trailing newline).
     Subsequent skills will use this resolved path, ensuring consistent behavior regardless of how the folder was referenced.

4. Read {SOURCE_FOLDER}/CLAUDE.md if it exists. Summarize the filer's situation in a brief paragraph (no PII details,
   just structural context like "MFJ filer, dual W-2, India/Canada cross-border").

5. Discover all documents using symlink-aware traversal:
   - Use `find {SOURCE_FOLDER} -follow -type f` (the -follow flag tells find to follow symlinks) to recursively list
     all files, including those reachable through symlinks within the folder.
   - This handles: Unix symlinks (ln -s), symlinked subdirectories, and symlinked files pointing elsewhere on disk.
   - macOS Finder aliases (created via right-click > Make Alias) are NOT Unix symlinks and cannot be followed by
     standard tools. If any files end in `.alias` or appear to be Finder alias files, list them separately and warn:
     "Found N Finder alias file(s) — these cannot be read by TaxBro. Convert to Unix symlinks with 'ln -s' or copy
     the actual files into the folder."
   - For each discovered file, use the symlink target's name and extension for categorization (not the symlink name).
     If a symlink name differs meaningfully from its target filename, note both.

   Group discovered files by likely category:
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
   (Use the resolved real path from step 3.)

7. Write a brief session log entry to {SOURCE_FOLDER}/TAXBRO/session-notes.md:
   - Date initialized
   - Original path provided and resolved path (if different)
   - Document count discovered (including documents reached via symlinks)
   - Count of any unresolvable Finder aliases (if any)
   - Any missing document flags
   Do NOT write any personal data, names, account numbers, or financial figures to this file — structural notes only.

8. Tell the user: "TaxBro initialized. Source folder loaded with N documents. Run /taxbro-checklist to see full status."
   If Finder aliases were found, include the warning about converting them.

IMPORTANT: Never write any PII, financial data, or document contents to ~/claude/taxbro/ or any file within it.
