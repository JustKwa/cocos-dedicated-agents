---
name: serena_mcp_guidelines
description: Serena MCP workflow for symbolic codebase exploration—use when you explore code yourself (e.g. primary agent) or when briefing Serena usage. The @explorer agent reads this file at session start; you do not need to paste it into explorer prompts.
---

## 1. Purpose

You have access to Serena MCP, a toolset that provides **symbolic understanding** of codebases via Language Server Protocol (LSP). Use this guide to determine when and how to use Serena versus default file tools (`read_file`, `write_file`, patch tools).

## 2. Decision Logic

Before performing any file operation, evaluate the task against these thresholds:

| Criteria       | Use Default File Tools              | Use Serena MCP                              |
| :------------- | :---------------------------------- | :------------------------------------------ |
| **File Count** | 1–2 known files                     | 3+ files or unknown file locations          |
| **File Size**  | < 300 lines                         | > 300 lines                                 |
| **Task Type**  | Creating new files, isolated fixes  | Refactoring, impact analysis, navigation    |
| **Context**    | Single-file scripts, config files   | Complex OOP, cross-file dependencies        |

If the file is already fully loaded in your context window and is small, do not call Serena. Use what you already have.

## 3. Mandatory Workflow

### A. Project initialization

You **must** complete project activation and onboarding **before** relying on symbol tools:

- Activate the Serena project for this repository (host tool name is usually `activate_project` / `serena_mcp_activate_project` or equivalent).
- If onboarding is required, run the onboarding/check tools your host exposes (e.g. `check_onboarding_performed`, then `onboarding` if needed).

Skipping this step can make symbol lookups incomplete or unreliable. Do not attempt symbol lookups before indexing is complete.

### B. Navigation & discovery

1. Run `find_symbol` with a relevant query. **Do not** use `grep` or list directory contents as a first resort.
2. If multiple results are returned, run `get_symbols_overview` on the most relevant files to understand the class hierarchy and narrow down the target.

### C. Reading code

- For files **under 100 lines**: use `read_file`.
- For files **over 300 lines**: use `find_symbol` or symbol body reads to load only the relevant symbol. Do not load the entire file into context.
- For files **between 100–300 lines**: use your judgment. If you need the full picture, `read_file` is acceptable. If you only need one function, use Serena.

### D. Editing code

- For **small files** (< 300 lines): use `write_file` or standard patch tools.
- For **large files** (> 300 lines): use `replace_symbol_body` to edit only the target symbol. Do not rewrite the entire file.
- For **adding new code** to an existing file (e.g., a new method or import): use `insert_after_symbol` to place it in the correct semantic location.

### E. Refactoring & impact analysis

Before renaming, deleting, or modifying a function signature:

1. Run `find_referencing_symbols` on the target symbol to identify all call sites and dependents.
2. Evaluate the impact across the codebase.
3. Apply changes systematically using `replace_symbol_body` at each affected site.

Do not skip step 1. Blind edits to shared symbols will cause regressions.

## 4. Fallback rules

- If `find_symbol` returns no results, fall back to `grep` or `read_directory` to verify the file and symbol exist.
- If the LSP server returns an error or times out, fall back to default file tools (`read_file`, `write_file`) and inform the user that Serena indexing may need to be restarted.
- If the project language is not supported by Serena's LSP (e.g., shell scripts, plain text, YAML), use default file tools immediately. Do not attempt Serena calls.

## 5. Token efficiency rules

- **Never** read a full large file when you only need one symbol. This wastes context and leads to "context rot."
- **Never** call Serena tools if the answer is already in your context window.
- **Prefer** `replace_symbol_body` over full file rewrites. This can save up to 70% of output tokens on large edits.
- **Batch** your Serena lookups. If you need to check 3 symbols, plan all 3 calls before executing, rather than doing them one at a time reactively.

## 6. Error handling

| Error                        | Action                                                                 |
| :--------------------------- | :--------------------------------------------------------------------- |
| Symbol not found             | Fall back to `grep` or `read_directory`                                |
| LSP timeout                  | Retry once. If it fails again, fall back to default tools              |
| Project not activated        | Run project activation and onboarding before retrying                  |
| Unsupported language         | Use default file tools. Do not retry with Serena                       |
