---
name: wtodo
description: Use when a session is ending or context is getting heavy — user says wrap up / done / goodbye / new session / restart session / continue / pick up where we left off, or the Chinese equivalents 收工 / 结束 / 先这样 / 再见 / 新会话 / 重启 session / 继续 / 接着上次, or the transcript has already been compacted or a tool output was truncated. Checks whether context is over 70%; if so writes or updates todo_<yymmdd>.md in the session cwd, warns that context is past 70% and recommends a fresh session, and in a new session reads the newest todo to restore context.
---

# wtodo — wrap up a session: write the todo, warn about context, restore in the next one

**No work is left unwritten when a session ends.**

The filename and format follow the convention you already use (`dsh-summarize-restart`'s `/todo_write`). Don't invent a new one.

## Iron law

```
Before a session ends, the work must be in todo_<yymmdd>.md. No file, no end.
```

## When it triggers

| Situation | What the user says |
|---|---|
| Wrapping up | wrap up / done / that's it / goodbye / 收工 / 结束 / 先这样 / 就这样吧 / 再见 / thanks |
| Switching sessions | new session / restart session / refresh / start over / 新会话 / 重启 session / 换个对话 / 重新开始 |
| Starting a new session | continue / pick up where we left off / where did we stop / 继续 / 接着上次 / 上次做到哪了 / 现在干什么 |
| Context is tight | you have seen a compaction, a truncated tool output, or you can no longer recall the opening of the session |

## Step 1: decide whether context is over 70%

**The model cannot see the exact percentage.** Take evidence in this order, and state in the reminder which one you used — never pretend to know a precise number.

| Priority | Evidence | Conclusion |
|---|---|---|
| 1 | The user gives a number ("it's at 78%", pastes the status bar or a screenshot) | Use their number |
| 2 | This session had a compaction, or a `long output is truncated` / spill-file path appeared | Treat as **>70%** (this profile compacts at 60%) |
| 3 | More than 25 user turns, or you can no longer restate the first 3 turns | Treat as **estimated >70%**, say it is an estimate |
| 4 | None of the above | Report "estimated under 70%", update the todo, do not push a new session |

If you are unsure, ask one question: "What does the context percentage in the top bar show?" If there is a real number, use it.

## Step 2: write or update `todo_<yymmdd>.md`

- Location: the session cwd, e.g. `<cwd>\todo_261002.md`
- Filename: `todo_` + two-digit year + month + day. Example: 2026-10-02 → `todo_261002.md`
- A file for today already exists → **update it in place**, fold the previous content into a `<details>` block at the end. Never overwrite and lose it.

Format:

```markdown
# todo_261002.md

> auto-generated 2026-10-02 · wtodo

## Topic A (e.g. mounting a workspace plugin)
- [x] Done items, with the key file paths
- [ ] Not done yet

## Key decisions
- What was chosen and why (the next session needs this most)

## Pitfalls
- What broke, so it does not break again
```

Rules:

1. Group by topic (`## heading`), not a running log of one-line entries.
2. `- [x]` for done, `- [ ]` for not done.
3. Write conclusions, file paths, and open items only — do not paste the conversation in.
4. "Key decisions" and "Pitfalls" must both exist; write "none" if empty.
5. If an existing todo in the cwd uses Chinese headings, keep its headings. Consistency beats translation.

## Step 3: the reminder (4 lines max)

```
Context ~75% (evidence: one compaction happened in this session). Written to todo_261002.md.
Start a new session: say "continue" and I will read todo_261002.md and pick up from there.
```

## Step 4: new-session startup flow

1. Glob `todo_*.md` in the cwd and take the largest filename date — not the newest modification time.
2. Read it and restate in 3–5 lines: **where we left off / what is next / what the pitfalls are**.
3. Resume the unchecked items. Do not re-ask the requirements.
4. If that todo is more than 3 days old, ask one question first: "This todo is from X days ago — anything changed since then?"

## Shortcut: the installed command

If this profile has `dsh-summarize-restart`, `/todo_write` does exactly step 2 (it reads the three most recent todos and writes today's). Suggest it. If it is not installed, write the file yourself with the write tool.

## Don't overdo it

- **When context is not under pressure, warn about switching sessions at most once per session.** If the user says "keep going", stop pushing.
- Never delete old todo files — they are the context source for the next session.
- If the user says "don't write files", do not write. Output a copy-paste todo block in the reply instead.
- wtodo is not a daily todo tool. It runs at exactly two moments: session wrap-up and session start.
