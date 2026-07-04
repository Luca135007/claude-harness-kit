---
name: harness-audit
description: Audit and revise the working-rules system (~/.claude/CLAUDE.md + ~/.claude/rules): verify environment facts, catch stale rules, apply safe revisions, adversarially review, then commit. Use when the user asks to "audit the rules" or "check the harness setup", as quarterly maintenance, after a Claude Code or model-family upgrade, or whenever a rule contradicts observed reality.
---

# Harness audit (rules health-check and revision)

Purpose: let a model of any tier safely maintain the rules system at `~/.claude/CLAUDE.md` and `~/.claude/rules/`.
This skill is the process skeleton; edit permissions and formats are governed by `~/.claude/rules/maintenance.md` — if the two conflict, maintenance.md wins; if you're unsure whether they conflict, ask the user.

## Step 0: Bootstrap check

If `~/.claude/rules/` does not exist, this kit hasn't been installed. Stop and tell the user to install the templates first (see the kit's README) — do not invent rules files from scratch.

## Step 1: Inventory (read-only, change nothing)

1. Backup state: if the rules directory is under git, run `git status -s` there — if dirty, commit the current state first (that is the backup). If not under git, copy each file you plan to touch to `<name>.bak` before any later edit.
2. Read `~/.claude/CLAUDE.md`; check every path in its routing table actually exists (`ls` / `Test-Path`).
3. Line caps: `CLAUDE.md` ≤150 lines, `rules/lessons.md` ≤150, each project CLAUDE.md ≤200.
4. Scan `rules/lessons.md` headings; mark entries that look stale.

## Step 2: Fact verification (test every environmental claim; trust nothing written)

- **Model IDs**: dispatch a minimal subagent task (e.g. "reply ok") with the cheapest model value from `rules/model-dispatch.md` section 0. Success = the dispatch table still works; failure = record the error, fall back to a call without a model parameter, and update the table.
- **Tool availability**: for every CLI tool the rules mention, run `command -v <tool>` (POSIX) or `Get-Command <tool> -ErrorAction SilentlyContinue` (PowerShell).
- **Paths and commands**: spot-check at least one path and one command from each project CLAUDE.md using harmless probes (`ls` / `Test-Path` / `python -m py_compile`).
- Anything you cannot verify gets written up as "unconfirmed" — filling gaps with plausible guesses is forbidden.

## Step 3: Revision (fix only what Steps 1–2 proved wrong; no drive-by "improvements")

1. Check edit permissions against maintenance.md's tier table — typically: lessons.md may be appended freely; CLAUDE.md and rules bodies may only be edited directly to correct a confirmed factual error; everything else needs the user's approval first.
2. Every edit must trace to a specific finding from Steps 1–2.
3. Stale lessons entries: flag and report; rewriting or deleting them needs user approval.
4. Over a line cap: extract long content into an on-demand file and leave an index line, don't trim meaning.

## Step 4: Adversarial review (never self-verify)

Dispatch a fresh-context `general-purpose` subagent (a cheap tier is fine). It has no access to your conversation — the prompt MUST include the raw findings from Steps 1–2 and your edit list verbatim (paths checked, model-test results, diffs), not just check-item names. The prompt must also say:

- "Assume these edits are wrong; find the problems."
- Checklist: rules contradicting each other / paths and tool names that don't exist (require it to actually test them) / model IDs inconsistent with the verification results provided / sentences a weaker model would misread.
- Report format: "file:line — problem — fix", ordered by severity.
- Do not tell it which parts you believe are fine.

Fix what it finds. Re-review only if the fixes were substantial.

## Step 5: Wrap-up

1. If under git: commit with message format `rules: <one sentence>`. Otherwise keep the `.bak` files beside the originals.
2. Report to the user in this fixed shape:
   - What was checked and found healthy (one line)
   - Problems found and fixed (each: problem → fix → file)
   - Items needing the user's decision (permission-gated or taste calls)
   - The "unconfirmed" list

## Forbidden

- Rewriting whole files to "polish wording" — touch only what a verified finding justifies.
- Editing files maintenance.md marks read-only.
- Editing anything before the backup step is done.
