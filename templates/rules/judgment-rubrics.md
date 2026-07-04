# Judgment Rubrics (executable by weaker models)

> Each rule comes with criteria, a positive example, and a negative example. When unsure, follow the checklist — don't go by feel.

## 1. When to escalate the model

**Criteria (escalate if any one holds; path in model-dispatch.md section 4):**
- [ ] The same subtask has failed enough times per the retry counts in model-dispatch.md section 4 (haiku: 1 failure; sonnet: 2 consecutive failures), and the cause isn't typo-level (typo-level = fixable by changing one character, a path, or a parameter name; needing a logic change or a different method does not count as typo-level)
- [ ] The task requires causal reasoning across 5+ files (not just searching)
- [ ] It involves a final judgment before an irreversible operation (deleting data, changing production config, publishing externally)
- [ ] The requirement itself is ambiguous and requires taste-based judgment to guess what the user actually wants

**Positive example**: sonnet fixes a race condition; after two rounds of test edits it still fails intermittently → escalate to opus with both rounds' diffs and test output attached.
**Negative example**: sonnet runs tests and discovers it forgot to install a package → this is an environment issue, not a capability issue; install it and rerun, no escalation needed.

## 2. When something actually counts as done

**All boxes must be checked to count as done:**
- [ ] Acceptance criteria checked off one by one (not "looks about right")
- [ ] There is mechanical evidence: passing test output, read-back file contents, actual run results
- [ ] Verification was done by a fresh-context agent or an actual run, not by looking at one's own code and saying it looks fine
- [ ] The change left no orphans (unused imports, leftover debug prints, temp files)

**Positive example**: "Fixed the bot reconnect bug" = wrote a repro script, the script passes after the fix, the bot was actually restarted once and confirmed working.
**Negative example**: "I reviewed the code logic, it should be fine" — no execution evidence, doesn't count as done, and also violates the ban on vague hedging language.

## 3. When to stop and ask the user

**Stop if any one holds (otherwise proceed autonomously — don't interrupt frequently):**
- [ ] Two or more reasonable interpretations exist, and the cost of picking wrong exceeds the cost of asking once (e.g., does "clean up old files" mean delete or archive?)
- [ ] About to do something irreversible or externally visible: deleting files, force push, sending messages to an external service, spending money
- [ ] The same thing has already been retried 2 rounds and still fails
- [ ] Discover that the user's premise is wrong (the file/setting they assume exists doesn't, or is the opposite) — report the fact, don't push forward on the wrong premise

**Positive example**: user says "switch the bot's model" without saying which one → list the locally available ollama models for them to choose from.
**Negative example**: mid-implementation, discover you need to choose camelCase vs snake_case for a variable naming style → just follow the surrounding code, no need to ask.

## 4. Signals that the direction is wrong (time to switch approach, not retry again)

**If any of these signals appear, stop the current approach and back up to the last decision point to try a different method:**
- [ ] Every fix produces a new error; the error count isn't converging
- [ ] Starting to add special cases/hacks just to make the approach work (the appearance of the 2nd special case is the signal)
- [ ] The scope of required changes keeps expanding, far beyond the original estimate (estimated 1 file, now 5 files)
- [ ] Fighting against the framework's or library's default behavior instead of going with it

**Positive example**: to keep a synchronous library from blocking an async bot, already wrapped it in two layers of thread + queue and there's now a deadlock → stop, find a native async library instead or ask the user.
**Negative example**: 3 tests failing, after a fix only 1 remains → the count is converging, this is normal debugging, keep going.

**Mandatory when switching approach**: write the dead end down in 1-3 lines in `lessons.md` (format in maintenance.md), to avoid repeating it in the next session.

## 5. How to verify the quality baseline (the last gate before delivery)

**Before any delivery, run the following based on artifact type:**

| Artifact | Minimum verification |
|---|---|
| Python code | `$env:PYTHONUTF8="1"; python -m py_compile <file>` passes + run tests if any exist |
| Config files (json/yaml/toml) | Load once with the corresponding parser. Run via the Bash tool (to avoid PowerShell quote nesting): `python -c 'import json,sys; json.load(open(sys.argv[1], encoding="utf-8"))' <file path>` |
| Docs / rule files | fresh agent read-back: read the file and check it item-by-item against the requirements list |
| Long-running programs (bot, server) | actually start it once, confirm it comes up and the log has no ERROR, then deliver |

**Positive example**: after editing bot.py → py_compile → restart the bot → check the log to confirm it connected to Discord → report "restarted and confirmed online."
**Negative example**: after editing, report "done" directly, when the bot actually can't start due to an import error.

## 6. Honesty clause (the baseline when reporting)

- If a test fails, say so and attach the output; if a step was skipped, say so explicitly.
- For information you couldn't find, write "unconfirmed" — don't fill the gap with a plausible-sounding guess.
- Decomposition and verification can save execution quality; they cannot save you from ambiguous requirements or taste judgment calls — when you hit those, say so explicitly (escalate the model, ask the user to decide, or state plainly that it can't be done); don't pretend you can.
