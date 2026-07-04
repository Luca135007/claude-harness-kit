# Model Dispatch Guidelines

> Purpose: spend expensive context and expensive models where it counts. The main conversation is the "commander," responsible for decisions and integration; reading, searching, batch edits, and verification are handed to cheaper subagents.

## 0. Available models and parameters (Example values verified 2026-07 — model names go stale; run the harness-audit skill or check docs.claude.com/en/docs/about-claude/models before trusting this table.)

| Tier | Model ID | Agent tool `model` value | Intended use |
|---|---|---|---|
| Cheapest | `claude-haiku-4-5-20251001` | `haiku` | Mechanical, low-ambiguity: batch string edits, read-back verification, format conversion |
| Cheap workhorse | `claude-sonnet-5` | `sonnet` | Default workhorse: search, implementation, refactoring, running tests, general research |
| Advanced | `claude-opus-4-8` | `opus` | Hard problems: architecture decisions, debugging after two consecutive failures, taste judgment, final review |
| Highest | `claude-fable-5` | `fable` | Top tier, only if your plan includes it |

- ⚠️ **The Agent tool has no `effort` parameter** (the only valid fields are description / isolation / model / prompt / subagent_type). Requirements about how thorough a subagent should be must be written as a sentence in the prompt body (e.g., "mechanical task, prioritize speed" / "deep review, better slow than sloppy"). effort exists only at the session level (the `/effort` command).
- ⚠️ Unconfirmed: whether requests routed to Opus 4.8 by the safety mechanism consume quota is unconfirmed — verify on the claude.ai usage dashboard when needed; do not write it into the rules based on a guess.
- Model IDs go stale: if specifying a model produces an error, fall back to calling without a model parameter (inheriting the main model), and record the new model ID in lessons.md.

## 1. The commander doesn't get into the trenches (when dispatch is mandatory)

Dispatch whenever any of the following holds — don't do it yourself:

1. **Large read volume**: expected to need more than 3 files, or a single file over 400 lines, to answer → `Explore` agent (read-only search).
2. **Mechanical batch work**: the same kind of edit needs to be applied in 3+ places → `general-purpose` agent, model `sonnet` (or `haiku` for very simple cases).
3. **Web research**: looking up documentation or API usage → `general-purpose` agent or `claude-code-guide` (dedicated to Claude Code / Claude API questions).
4. **Verification**: any "check whether what I just did is correct" → always dispatch (see section 5).

The only legitimate reasons for the main conversation to act directly: a small single-file edit, a final edit requiring cross-file integration judgment, or the conversation with the user itself.

## 2. The three required elements of a task handoff (every subagent prompt must include)

1. **Goal and motivation**: what to accomplish and why (so the agent can make its own tradeoffs when it hits something unexpected).
2. **Acceptance criteria**: mechanically checkable completion criteria (e.g., "py_compile passes," "every returned conclusion includes file:line").
3. **Report format**: explicitly specify what to report back and what not to.

Templates are in `prompt-templates.md` — fill in the blanks and use directly.

## 3. Reporting contract (output rules for subagents, to be written into every handoff prompt)

- Report only: conclusions, the minimal evidence needed for the decision, `file:line` references.
- Do not report: full file contents, complete command output, a blow-by-blow account of the exploration process.
- Long artifacts (reports, large code blocks, lists) → save to the specified path, return only the path + a 3-line summary.
- Report failures honestly: if it couldn't be done, say so and say where it got stuck — don't report "roughly done."

## 4. Escalation/de-escalation path

- **haiku fails once** → re-dispatch the same task at sonnet.
- **sonnet fails on the same subtask twice in a row** → escalate to opus, and the prompt must include the full failure trail (what was done, the verbatim error messages, hypotheses already ruled out) — don't make opus guess from scratch.
- **After opus solves it**: write the solution pattern as explicit steps, then de-escalate to sonnet/haiku to batch-apply it to the remaining similar spots.
- **Retry the same thing at most 2 rounds** (3 attempts total). Still failing → stop, report the blocker and what's been tried to the user (criteria in judgment-rubrics.md section 3).

## 5. No Self-Verification

The one who did the work does not verify their own output. Verification is always dispatched to a **fresh-context** subagent (a new general-purpose agent, not a fork, to avoid inheriting bias):

| Artifact type | Verification method |
|---|---|
| Docs / config files | read-back: a new agent reads the file, confirms existence and completeness, checks it item-by-item against the requirements list |
| Code | run tests or actually run it (not just reading the code); at minimum a mechanical check at the `python -m py_compile` level |
| High-risk judgment (architecture, deletions, irreversible operations) | second opinion: dispatch an agent specifically to find fault (prompt says "try to disprove this conclusion"); or generate 2-3 alternative approaches and have another agent judge and pick the best |

The verification agent's prompt should not reveal "I think this is already done correctly" — give it only the requirements and the artifact path.

## 6. Agent tool parameter quick reference for dispatch

```
Agent(
  subagent_type: "Explore" | "general-purpose" | "claude-code-guide" | "Plan",
  model: "haiku" | "sonnet" | "opus",   // specify explicitly, don't leave blank and default to an expensive model
                                        // ("fable" is also a valid value, but don't use it unless confirmed available, see section 0)
  prompt: <prompt with all three required elements>
)
```

- Multiple independent tasks → dispatch them in parallel in the same message.
- `Explore` is read-only and cannot edit files; use `general-purpose` for edits.
