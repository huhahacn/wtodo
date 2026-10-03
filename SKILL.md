---
name: wtodo
description: Use when a session is ending or context is getting heavy — user says 收工 / 结束 / 先这样 / 再见 / 新会话 / 重启 session / 继续 / 接着上次, or the transcript has already been compacted or a tool output was truncated. Checks whether context is over 70%; if so writes or updates todo_<yymmdd>.md in the session cwd, warns that context is past 70% and recommends a fresh session, and in a new session reads the newest todo to restore context.
---

# wtodo —— 会话收尾：落盘 todo + 提醒换会话 + 新会话恢复

一句话：**工作不落盘，不许结束会话。**

`todo_<yymmdd>.md` 的命名和格式沿用你已有的约定（`dsh-summarize-restart` 的 `/todo_write` 就是这套），别另起一套。

## 铁律

```
会话结束前，工作必须先写进 todo_<yymmdd>.md；不落盘不结束。
```

## 触发时机

| 场景 | 用户会说的话 |
|---|---|
| 会话收尾 | 收工 / 结束 / 先这样 / 就这样吧 / 再见 / done / thanks |
| 换会话 | 新会话 / 重启 session / 刷新 / 换个对话 / 重新开始 |
| 新会话开工 | 继续 / 接着上次 / 上次做到哪了 / 现在干什么 |
| 上下文吃紧 | 你看到过 compaction、工具输出被截断、或已经复述不出会话开头 |

## 第一步：判定上下文是否 >70%

**模型看不到精确百分比**，所以按下面的顺序取证据，并在提醒里写明你用的是哪一条 —— 别假装知道准确数字。

| 优先级 | 证据 | 结论 |
|---|---|---|
| 1 | 用户给的数字（「现在 78%」、贴了状态栏/截图） | 以它为准 |
| 2 | 本会话出现过 compaction，或出现过 `long output is truncated` / spill 文件路径 | 按 **>70%** 处理（本 profile 60% 就触发压缩） |
| 3 | 用户消息 > 25 轮；或你已复述不出会话开头 3 轮的内容 | 按 **估算 >70%** 处理，注明是估算 |
| 4 | 以上都不成立 | 报「估算未达 70%」，只更新 todo，不劝换会话 |

拿不准就直接问用户一句：「顶栏显示的上下文是多少？」——有准确数字就用准确数字。

## 第二步：写/更新 `todo_<yymmdd>.md`

- 位置：会话 cwd（`<cwd>\todo_261002.md` 这种）
- 文件名：`todo_` + 两位年 + 两位月 + 两位日。例：2026-10-02 → `todo_261002.md`
- 今天已有同名文件 → **原地更新**，把更新前的内容折进文件末尾的 `<details>` 里，不要覆盖丢失

格式：

```markdown
# todo_261002.md

> 自动生成于 2026-10-02 · wtodo

## 主题A（例：工作区挂载插件）
- [x] 已完成的事项（带关键文件路径）
- [ ] 未完成的事项

## 关键决定
- 选了什么、为什么（新会话最需要这一段）

## 坑
- 踩过什么、下次别踩
```

规则：

1. 按主题分组（`## 标题`），不要写成一条条流水账
2. 已完成 `- [x]`，未完成 `- [ ]`
3. 只写结论、文件路径、未完成项 —— 不要把对话记录搬进去
4. 「关键决定」和「坑」两节必须有，没有就写「无」

## 第三步：提醒（≤4 行，别啰嗦）

```
上下文约 75%（依据：本会话出现过一次压缩）。已写入 todo_261002.md。
建议开新会话：说「继续」，我读 todo_261002.md 接着干。
```

## 第四步：新会话开工流程

1. 用 glob 在 cwd 找 `todo_*.md`，取日期最大那份（不是最新修改时间，是文件名最大）
2. 读完用 3–5 行复述：**上次做到哪 / 下一步做什么 / 有哪些坑**
3. 再从 todo 里没打勾的条目接着干，别重新问一遍需求
4. 如果那份 todo 是 3 天前的，先问一句「这份 todo 是 X 天前的，期间有别的进展吗」

## 捷径：已装的命令

如果这个 profile 装了 `dsh-summarize-restart`，`/todo_write` 命令做的事和第二步完全一样（自动读最近 3 份 todo + 写今天的），直接建议用户敲它；没装就自己用 write 工具写文件。

## 别做过头

- **上下文没压力时，一次会话最多提醒换会话一次**；用户说「继续用」就闭嘴，别再劝
- 不删旧 todo 文件 —— 历史文件就是新会话的上下文来源
- 用户说「别写文件」→ 不写，改成在回复里给一段可直接复制的 todo 文本
- 别把 wtodo 当日常 todo 工具：它只在会话收尾/开工这两个节点用
