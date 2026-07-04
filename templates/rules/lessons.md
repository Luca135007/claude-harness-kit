# Lessons/Pitfalls (append-only)

> Add new entries at the bottom, format in maintenance.md section 2. Before starting a non-trivial task, a quick scan of the headings is enough.

<!-- example entry, replace with your own -->
## 2026-07-04 CLAUDE.md referenced a CLI tool that wasn't installed
- Context: the old global CLAUDE.md mandated wrapping commands with a specific CLI tool, but the tool was never installed on this machine
- Error: every session repeatedly tried the tool, failed, and fell back, wasting tool calls
- Rule: before writing any tool into a rule file, confirm it exists first — run `command -v <tool>` via the Bash tool, or `Get-Command <tool> -ErrorAction SilentlyContinue` via PowerShell (PowerShell has no command -v); paths and commands referenced in rule files must all be verified by actually testing them
