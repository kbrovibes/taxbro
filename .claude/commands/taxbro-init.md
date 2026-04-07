Initialize TaxBro with a source tax documents folder.

**Scope: US Federal Tax Returns only (Form 1040 + international schedules).**
Foreign income, accounts, and taxes from any country are included only insofar as they are
reportable on a US return (Schedule E, Form 1116, FBAR, Form 8938, Form 8621, etc.).
All output files are prefixed with "US-" to make this scope explicit.

Usage:
  /taxbro-init /path/to/tax/folder/           — full initialization
  /taxbro-init /path/to/tax/folder/ --reset   — snapshot existing outputs, then fresh init
  /taxbro-init --reset                        — snapshot + reset using current session path

Flags:
  --reset   (also accepted: --remove)
            Move all current TAXBRO/ output files into a timestamped snapshot directory
            at {SOURCE_FOLDER}/TAXBRO/.snapshots/YYYY-MM-DD-HHMMSS/ before reinitializing.
            Nothing is ever deleted. Snapshots are preserved indefinitely.
            Skills never read from .snapshots/ — it is treated as archived output only.

⚠️ ABSOLUTE RULE: TaxBro NEVER deletes any file, EVER. Not source documents, not outputs,
not snapshots, not anything. The only file operations permitted are: READ and WRITE (create
or overwrite a specific output file). If you find yourself about to run rm, unlink, rmdir,
or any deletion command — STOP. Use --reset to archive outputs instead.

Steps:
1. Parse $ARGUMENTS:
   - RESET_MODE = true if "--reset" or "--remove" is present in $ARGUMENTS
   - SOURCE_PATH = first token that does not start with "--"
   - If no SOURCE_PATH and RESET_MODE is false: ask user for path and stop.
   - If no SOURCE_PATH and RESET_MODE is true: read .current-session for the path.
   - If .current-session is missing and no path provided: tell user to provide a path and stop.

2. Verify the folder exists. If it doesn't, tell the user and stop.

2a. If RESET_MODE is true — snapshot existing outputs BEFORE doing anything else:
   - SNAPSHOT_TS = current timestamp formatted as YYYY-MM-DD-HHMMSS
   - SNAPSHOT_DIR = {SOURCE_FOLDER}/TAXBRO/.snapshots/{SNAPSHOT_TS}/
   - If {SOURCE_FOLDER}/TAXBRO/ exists and has any top-level contents (files or dirs other
     than .snapshots/ itself):
     - Create SNAPSHOT_DIR (mkdir -p)
     - Move every item directly inside TAXBRO/ (except .snapshots/) into SNAPSHOT_DIR:
       Run: find "{SOURCE_FOLDER}/TAXBRO" -maxdepth 1 -mindepth 1 ! -name ".snapshots" \
            -exec mv {} "{SNAPSHOT_DIR}/" \;
     - Write a one-line marker file {SNAPSHOT_DIR}/.taxbro-snapshot containing:
       "snapshot:{SNAPSHOT_TS}" (so tools can identify this as an archived snapshot)
     - Tell user: "Existing outputs archived → TAXBRO/.snapshots/{SNAPSHOT_TS}/"
   - If TAXBRO/ does not exist or is already empty: note "No existing outputs to snapshot."
   - DO NOT delete anything. DO NOT touch .snapshots/ itself. Only move top-level items.

3. Resolve the real path:
   - Run `realpath "$ARGUMENTS"` (or `python3 -c "import os,sys; print(os.path.realpath(sys.argv[1]))" "$ARGUMENTS"`) to
     resolve any symlinks in the folder path itself.
   - If the folder is a symlink, note the resolved target to the user: "Note: folder is a symlink → resolves to [real path]"
   - Write the RESOLVED absolute path to .current-session in the taxbro project root (single line, no trailing newline).
     Subsequent skills will use this resolved path, ensuring consistent behavior regardless of how the folder was referenced.
     Using a project-relative path means each git checkout of taxbro maintains its own independent session pointer.

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

ABSOLUTE RULE — NO DELETIONS EVER:
TaxBro must NEVER delete any file under any circumstances. This includes source documents,
TAXBRO output files, snapshots, and any other file on disk. The only permitted file system
write operations are: creating new files, writing/overwriting specific output files, and
moving files (mv) into .snapshots/ during --reset. Never run rm, rmdir, unlink, shutil.rmtree,
os.remove, or any equivalent. If you feel the urge to delete something, archive it instead.

SNAPSHOT VISIBILITY RULE:
Skills and analysis commands must NEVER read from or write into TAXBRO/.snapshots/.
That directory is archived output only. Treat it as if it does not exist for all analysis purposes.
When discovering files with find, always exclude .snapshots/: use -not -path "*/TAXBRO/.snapshots/*"
