# Task Handoff Prompt Templates

> Usage: pick the matching template, fill in the `{...}` blanks, and use the whole block as the Agent tool's prompt. `model` and `subagent_type` follow the header's suggestion. All three required elements (goal & motivation / acceptance criteria / report format) are mandatory.
> ⚠️ The Agent tool has **no** `effort` parameter — the thoroughness requirement is already written into each template's body; don't try to pass it as a parameter.
> Before filling in: if you don't know the project's test/start commands, read that project's CLAUDE.md first; if there's no project CLAUDE.md, dispatch an Explore agent first to find out before handing off the task — don't leave it blank and guess.

## Universal reporting contract (already included at the end of every template, do not remove)

> Reporting rule: report only conclusions and the minimum necessary evidence; references must always use "absolute file path:line number." Do not paste full file contents or complete command output. Artifacts over 30 lines should be saved to {output path}, returning the path plus a 3-line summary. State honestly what couldn't be done or is uncertain — don't report "roughly done."

---

## 1. Search / locate (subagent_type: `Explore`, model: `sonnet`)

```
This is a locate task, prioritize speed, don't over-dig.
Goal: find {what to find, e.g. "all locations in bot.py that call the Ollama API"}.
Motivation: {why you need to find it, e.g. "next we're going to unify the timeout parameter to 120 seconds"}.
Scope: {directory or file list}; search breadth: {quick|medium|very thorough}.
Acceptance criteria:
- Every result includes file:line and a one-line note on what that spot does
- Explicitly state "search complete" or "the following locations are uncertain, recommend manual review": {uncertain list}
Report format: bullet list, each item "file:line — description". If not found, say so directly and list the keywords you searched.
(Universal reporting contract applies)
```

## 2. Implementation (subagent_type: `general-purpose`, model: `sonnet`)

```
Goal: {the feature to implement, one sentence}.
Motivation: {the user's context, so you can make your own tradeoffs on details}.
Constraints:
- Only touch these files: {list}; allowed to add new files: {no|yes, new files go in {directory}}
- If you find issues in other files, only report them, don't fix them
- For behavior that can only be observed via an external service (e.g. a real Discord connection): a mock or dry run is sufficient for verification; note in the report "not verified against a real environment"
- Follow ~/.claude/CLAUDE.md sections 2 and 3 (minimal solution, surgical edits)
- Identifiers must always be named in English; comments may be in Chinese
Acceptance criteria (all must be met to count as done):
- {observable feature behavior description}
- `$env:PYTHONUTF8="1"; python -m py_compile {file}` passes
- {test command} passes (if no tests exist: write a minimal test that verifies this feature first)
Report format: which files were changed (file:line range), acceptance criteria checked off one by one (pass/fail), remaining risk in one sentence.
(Universal reporting contract applies)
```

## 3. Refactoring (subagent_type: `general-purpose`, model: `sonnet`)

```
Goal: refactor {target} into {target form}; external behavior must not change.
Motivation: {why this refactor is worthwhile}.
Prerequisite: first run {existing test command}, record the passing status as a baseline. If there are no tests, stop and report "no test coverage, recommend adding tests first" — don't refactor blind.
Constraints: don't change public interfaces/function signatures unless explicitly listed: {allowed changes list}.
Acceptance criteria:
- The same test suite passes fully after the refactor, matching the baseline
- No new dependencies, no leftover dead code (that you introduced)
Report format: before/after test result comparison, summary of changes (file:line), behavioral differences (should be "none").
(Universal reporting contract applies)
```

## 4. Research / lookup (subagent_type: `general-purpose`, model: `sonnet`)

```
Goal: answer {specific question, e.g. "discord.py 2.x's slash command timeout limit and workarounds"}.
Motivation: {what decision this answer will inform}.
Method: prioritize official docs and GitHub source/issues, then community sources; cross-confirm key facts with at least 2 independent sources.
Acceptance criteria:
- Every key conclusion includes a source URL and what was checked
- Clearly distinguish three confidence levels: "stated in docs" vs "community experience" vs "your own inference"
- List sub-questions that couldn't be answered as an "unconfirmed list" — don't fill gaps with speculation
Report format: conclusion first (within 3 lines), followed by each fact with its source; if the report exceeds 30 lines, save it to {output path}.
(Universal reporting contract applies)
```

## 5. Review / verification (subagent_type: `general-purpose`, model: `sonnet`, use `opus` for major judgment calls)

```
You are a fresh-context reviewer; you don't know and don't need to know the prior implementation process. Please review thoroughly — better slow than sloppy.
Goal: check against the requirements list and try to find problems in {artifact path}. Assume it's wrong; your job is to find how.
Requirements list (check each one):
- {condition 1}
- {condition 2}
Method:
- Docs: read back the full text, check item by item; verify paths, commands, and names actually exist (test with ls/command -v)
- Code: actually run tests or start the program, don't judge from reading code alone
Acceptance criteria: mark each requirement Pass / Fail / Unable to verify (with reason).
Report format: issue list (sorted by severity, with file:line and suggested fix); if there are no issues, say plainly "checked item by item, all pass" — don't pad it with courtesy "suggestions."
(Universal reporting contract applies)
```
