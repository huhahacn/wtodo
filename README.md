# wtodo

At the end of a session, write the work down as a todo, warn you that context is full, and in the next session read it back and continue.

An [Agent Skill](https://code.claude.com/docs/en/skills) (`SKILL.md` format — works with DSH, Claude Code, and Codex).

## The problem

A long session runs out of context: compaction starts, details get lost, and the AI can no longer recall what you said at the beginning. You wrap up, and the next session starts like amnesia — everything has to be re-explained.

wtodo closes the loop in three steps:

1. **Wrap up** — when a session is ending, decide whether context is over 70%
2. **Write it down** — if it is, write or update `todo_<yymmdd>.md` in the cwd (existing todo format, old content folded into `<details>` so nothing is lost)
3. **Restore** — the first thing a new session does is read the newest todo, restate in 3–5 lines "where we left off / what is next / the pitfalls", and continue from the unchecked items

## When it triggers

| Situation | What the user says |
|---|---|
| Wrapping up | wrap up / done / that's it / goodbye / 收工 / 结束 / 先这样 / 再见 / done |
| Switching sessions | new session / restart session / refresh / start over / 新会话 / 重启 session / 换个对话 |
| Starting a new session | continue / pick up where we left off / where did we stop / 继续 / 接着上次 / 上次做到哪了 |
| Context is tight | a compaction happened, a tool output was truncated, or the AI can no longer recall the opening |

## An honest note about "over 70%"

**The model cannot see DSH's exact context percentage** — there is no tool or API for it. So wtodo never pretends to know the number. It walks an evidence chain and states which evidence it used:

| Priority | Evidence | Conclusion |
|---|---|---|
| 1 | The user gives a number ("it's at 78%", pastes the status bar) | Use their number |
| 2 | A compaction happened, or a tool output was truncated | Treat as **>70%** (compaction triggers at 60% by default) |
| 3 | More than 25 user turns, or the first 3 turns cannot be recalled | Treat as **estimated >70%**, say it is an estimate |
| 4 | None of the above | Report "under 70%", update the todo, do not push a new session |

If unsure, ask one question: "What does the context percentage in the top bar show?"

## What's inside

| Section | Content |
|---|---|
| Iron law | no work left unwritten when a session ends |
| Evidence table | the 4-level chain above |
| Writing the todo | `todo_261002.md` lands in the session cwd; name = `todo_` + two-digit year/month/day |
| Todo format | `## Topic` grouping, `- [x]` / `- [ ]`, "Key decisions" and "Pitfalls" sections are mandatory |
| Reminder template | ≤4 lines: percentage + evidence + file written + recommend a new session |
| New-session restore | glob the largest `todo_*.md` filename → restate progress → continue, without re-asking the requirements |
| Don't overdo it | warn about switching sessions at most once per session; never delete old todos; if the user says "don't write files", don't |

## Install

```powershell
# Global: available in every project
git clone https://github.com/huhahacn/wtodo "$env:USERPROFILE\.agents\skills\wtodo"

# Current project only
git clone https://github.com/huhahacn/wtodo ".\.dsh\skills\wtodo"
```

| Tool | Skill roots |
|---|---|
| DSH | `~/.dsh/skills/`, `~/.agents/skills/`, `<project>/.dsh/skills/`, `<project>/.agents/skills/` |
| Claude Code | `~/.claude/skills/` |
| Codex | `~/.agents/skills/` |

Or copy `SKILL.md` manually to `<skill root>\wtodo\SKILL.md`.

## Usage

It fires automatically once installed. To call it by hand: type `/wtodo` in DSH, or just say "write a todo, I'm switching sessions".

The output looks like this, in the session working directory:

```markdown
# todo_261002.md

> auto-generated 2026-10-02 · wtodo

## Mounting a workspace plugin
- [x] Fixed chip insertion for the attach button (lib/client/chip.js)
- [ ] Add edge cases to route.test.js

## Key decisions
- Used a junction instead of a symlink, no admin rights needed on Windows

## Pitfalls
- pwsh exits with 0xC0000142 under the workspace-write sandbox; commands need escalation
```

## License

MIT — see [LICENSE](LICENSE). Use it, change it, ship it.
