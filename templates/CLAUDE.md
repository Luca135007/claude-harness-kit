# Global Workspace Rules (Index)

This file contains core rules and routing indexes. Long-form context is stored in the `rules` directory. **When facing specific scenarios, ALWAYS use the Read tool to fetch the mapped file before acting.**
For trivial tasks (single file, <30 lines changed, no research needed), use judgment to simplify the process. Bias toward caution and stability over speed.

## 0. Rule Routing (Read On-Demand, Do NOT Preload)

| Scenario | Read this file first (Absolute Path) |
|---|---|
| Dispatching subagents, selecting model, verifying output | `~/.claude/rules/model-dispatch.md` |
| Writing delegated prompts (Search/Implement/Refactor/Research/Review) | `~/.claude/rules/prompt-templates.md` |
| Uncertainties: Upgrade model? Is it done? Ask user? Pivot? | `~/.claude/rules/judgment-rubrics.md` |
| Editing CLAUDE.md, rule files, settings, or logging a lesson | `~/.claude/rules/maintenance.md` |
| Past pitfalls (Scan this before starting non-trivial tasks) | `~/.claude/rules/lessons.md` |

## 1. Think Before Coding
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. If the user's approach introduces significant workload or risk, propose an alternative once, then follow the user's final decision.
- If something is unclear, stop. Name what's confusing and ask.

## 2. Simplicity First
Write the minimum code required. No speculative design:
- No unrequested features, abstractions, "flexibility", or "configurability".
- No error handling for impossible scenarios.
- Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes (Chesterton's Fence)
- Touch only what you must. Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken. Match existing style.
- If you notice unrelated dead code, mention it - don't delete it.
- Remove orphans (imports/variables/functions) created by YOUR changes. Leave pre-existing dead code alone.
- **Validation:** Every changed line must trace directly to the user's request.

## 4. Goal-Driven Execution & Memory Loop
Transform tasks into verifiable goals before executing:
- "Add validation" → "Write tests for invalid inputs, then make them pass."
- For multi-step tasks, state a brief plan: `1. [Step] → verify: [check]`
- **Continuous Iteration:** After fixing a complex bug, append the root cause and rule to `lessons.md` (format: maintenance.md §2), and mention it in your report.

## 5. Output Control & Terminal Safeguards
- For high-output commands, you MUST limit the output:
  - Bash: Append `| head -50`
  - PowerShell: Append `| Select-Object -First 50`
- When running tests, capture only the failure summary.
- Use the Read tool's offset/limit parameters to read specific sections of large files.
- **Security Boundary:** Always ask for explicit confirmation before executing destructive commands (e.g., deleting databases, dropping tables, or bulk overwriting critical files).

## 6. Delegation Principles (Save Context)
- Keep the main conversation clean. Provide only conclusions in the main thread.
- If a task requires reading **>3 files or >400 lines**, dispatch an `Explore` subagent. Return only "Conclusion + File:LineNumber".
- For mechanical batch edits, verifications, or web searches → Dispatch a subagent (Use `sonnet` or `haiku` model).
- Always read `~/.claude/rules/model-dispatch.md` before delegating.
- **No Self-Verification:** Dispatch a fresh-context agent to verify completed implementations.
- DO NOT dump raw outputs (full files, long error stacks, `pip list`) in the main chat. Delegate or limit lines.

## 7. Platform Notes
- PowerShell 5.1: **No `&&` or `||`**. Use `;` or `if ($?) { ... }`.
- PowerShell writes files in UTF-16 by default. Append `-Encoding utf8` when generating files for other tools.
- Before running Python, set `$env:PYTHONUTF8 = "1"` (or `export PYTHONUTF8=1` on POSIX shells) to prevent encoding errors with non-ASCII outputs.

## 8. Project Index
<!-- Add your projects here: path + one-line description + link to project CLAUDE.md -->
