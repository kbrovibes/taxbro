# TaxBro Commands — Gemini CLI Configuration

## How to use skills
Skills are defined in Markdown files within this directory. Each file should describe the logic for a specific tax analysis task.

### Skill Template for Gemini:
1. **Research Phase:**
   - Use `read_file` to get the `SOURCE_FOLDER` path from `.current-session`.
   - Use `glob` or `list_directory` on the `SOURCE_FOLDER` to find relevant tax documents (PDFs, CSVs).
   - Read the filer's context from `{SOURCE_FOLDER}/CLAUDE.md` or `{SOURCE_FOLDER}/GEMINI.md`.

2. **Analysis Phase:**
   - Use `read_file` or relevant MCP tools to extract data from the documents.
   - Cross-reference with IRS rules (e.g., FBAR thresholds: $10,000 across all accounts).

3. **Execution Phase:**
   - Format the findings according to the TaxBro schema.
   - Write the final report to `{SOURCE_FOLDER}/TAXBRO/{skill-name}-summary.md`.

## Data Safety Rules
- NEVER echo document contents back to the user or into this directory.
- All extracted data MUST stay within the context of generating the output file in the `SOURCE_FOLDER`.
