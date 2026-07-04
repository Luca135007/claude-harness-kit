# claude-harness-kit

Rule files that make cheaper Claude models work with senior-engineer discipline.

## The idea

Big models are expensive and rationed. Most sessions run on cheaper, faster models — and the quality gap shows up not in raw intelligence, but in *discipline*: reading whole files into context instead of delegating, self-certifying work as "done", retrying a dead-end approach five times, silently guessing when facts should be verified.

This kit externalizes that discipline into files a weaker model can actually follow:

- **Concrete rules, not vibes.** Every rule has a threshold ("more than 3 files or 400 lines → delegate"), a checklist, or a positive/negative example pair. Abstract advice ("be careful") is worthless to a small model; executable criteria are not.
- **The commander stays out of the trenches.** The main conversation makes decisions; bulk reading, batch edits, research, and verification go to cheap subagents that report back conclusions plus `file:line` references — never raw dumps.
- **No self-verification.** Whoever did the work never checks it. Verification goes to a fresh-context agent that gets the requirements and the artifact, not the author's optimism.
- **Escalate on evidence, de-escalate on success.** One failure at the cheap tier → retry one tier up with the full failure trail attached. Once the expensive model cracks the pattern → hand the pattern back down for batch application.
- **Facts rot; verify before trusting.** Model IDs, tool availability, paths — anything the rules assert about the environment gets re-verified by an audit skill, not trusted from memory.

The kit was distilled in a single session with a top-tier model, deliberately designed to outlive it: the value is in the institution, not the model that wrote it.

## What's inside

```
templates/
  CLAUDE.md                 ← always-loaded index (≤150 lines, routing table only)
  rules/
    model-dispatch.md       ← delegation, model/tier selection, escalation paths, verification contract
    judgment-rubrics.md     ← when to escalate / when it's actually done / when to ask the user / when to pivot
    prompt-templates.md     ← fill-in-the-blank handoff prompts: search, implement, refactor, research, review
    maintenance.md          ← who may edit what, lesson format, pruning thresholds
    lessons.md              ← append-only pitfall log (starts with one example entry)
skills/
  harness-audit/SKILL.md    ← /harness-audit: verify environment facts, catch stale rules, revise safely, commit
```

## Install

1. Copy `templates/CLAUDE.md` to `~/.claude/CLAUDE.md` (merge if you already have one — keep yours ≤150 lines; it should be an index, not an essay).
2. Copy `templates/rules/` to `~/.claude/rules/`.
3. Copy `skills/harness-audit/` to `~/.claude/skills/harness-audit/`.
4. Open `~/.claude/CLAUDE.md` and fill in the project index placeholder; skim each rules file and adjust thresholds to taste.
5. Optional but recommended: put `~/.claude/rules/` under version control so edits are auditable and reversible.

Then run `/harness-audit` once — it verifies that every path, tool, and model ID the rules mention is real on *your* machine, and flags what isn't.

## How it works

`CLAUDE.md` is the only always-loaded file. It holds the core behavioral rules plus a routing table: "in situation X, Read rules file Y first." Everything else loads on demand, so the standing context stays small and the detailed protocols stay maintainable.

Maintenance is part of the design: lessons append to `lessons.md` in a fixed format, pruning triggers at fixed line counts, and `/harness-audit` re-verifies environment facts on a schedule. Institutions rot silently; this one audits itself.

## Honest caveats

- The numeric thresholds (3 files/400 lines, 2 retry rounds, 150-line caps) are working heuristics, not measurements. Tune them.
- Model IDs in `model-dispatch.md` go stale. The audit skill exists precisely because of this.
- Externalized process fixes execution quality. It does not fix taste — ambiguous requirements and judgment calls still need a stronger model or a human. The rubrics tell the weak model to *recognize* those moments and escalate, which is the honest best it can do.

## License

MIT
