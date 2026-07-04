# Maintenance Protocol

## 1. Edit permission tiers

| File | Can a weaker model edit it on its own? | Condition |
|---|---|---|
| `~/.claude/rules/lessons.md` | Yes, append anytime | Append only, may not rewrite or delete existing entries (exception for pruning, see section 3) |
| Project CLAUDE.md (e.g. `<project>/CLAUDE.md`) | Yes | Limited to "updating stale facts" (commands, paths, versions); ask the user first before adding new sections |
| Memory files (`~/.claude/projects/*/memory/`) | Yes | Follow the memory conventions in the system prompt |
| `~/.claude/CLAUDE.md` and the body of `~/.claude/rules/*.md` | Ask the user first | Exception: correcting a confirmed factual error (model renamed, path no longer valid) can be done directly, but the report must explicitly state what was changed |
| `settings.json` / `settings.local.json` | Ask the user first | Permissions are handled via the system dialog; don't edit manually |
| Any file you designate as historical (e.g. audit reports, hand-off notes) | Do not edit | Read-only records; list them here as you add them |

**Before editing any file under `~/.claude/rules/`**: if your rules dir is version-controlled, commit the current state first before editing (a clean status means already backed up), then commit again after editing with message format `rules: <one sentence>`. Otherwise, copy the file to a `.bak` before editing.

## 2. Where lessons/pitfalls get written and in what format

- **Lessons general across projects** → append to `~/.claude/rules/lessons.md`
- **Lessons specific to a single project** → that project's CLAUDE.md "Known Pitfalls" section, or the project's memory file
- Triggers: when switching direction after hitting a dead end, when the same mistake happens a second time, when the user corrects you.

Format (one entry, 3-5 lines, copy this structure):

```markdown
## 2026-07-04 Synchronous ollama call blocked the async bot
- Context: called the synchronous ollama client directly inside a discord.py async handler
- Error: the entire bot event loop got blocked, heartbeat timed out and disconnected
- Rule: always wrap synchronous blocking calls made from an async context in asyncio.to_thread()
```

## 3. When accumulated length triggers pruning

- `lessons.md` exceeding **150 lines** → prune: merge similar entries, delete confirmed-stale ones, move anything promotable to a general rule into the corresponding rules file. Commit to git as a backup before pruning (or copy to a `.bak` if not version-controlled); pruning counts as a "rewrite" and requires asking the user first.
- Project CLAUDE.md exceeding **200 lines** → extract long content into `<project>\.claude\docs\*.md`, leaving an index in the body.
- `~/.claude/CLAUDE.md` hard cap of **150 lines**: when wanting to add new content, first ask "can this go into an existing rules file" instead of adding lines.

## 4. Quarterly (or on-demand) health checklist

- [ ] Do the models in `~/.claude/rules/model-dispatch.md` section 0 still exist? (dispatch a haiku agent as a trial to find out)
- [ ] Do all the paths in CLAUDE.md's routing table actually exist? (confirm one by one with `ls`)
- [ ] Are the paths and commands referenced by each project's CLAUDE.md still valid? (at minimum confirm each project's CLAUDE.md exists and the venv path is correct)
- [ ] Check whether tools exist: in Bash use `command -v <tool>`; in PowerShell use `Get-Command <tool> -ErrorAction SilentlyContinue` (PowerShell has no command -v)
- [ ] Are there any entries in lessons.md that no longer apply?
- [ ] If your rules dir is version-controlled, does it have any uncommitted changes?
