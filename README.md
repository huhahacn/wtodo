# wtodo

会话收尾时把工作落盘成 todo，提醒你换会话，新会话再读回来接着干。

一个 [Agent Skill](https://code.claude.com/docs/en/skills)（`SKILL.md` 格式，DSH / Claude Code / Codex 通用）。

## 解决什么

长会话干到最后上下文快满了：开始压缩、忘细节、复述不出开头说过什么。这时候收工，下次会话就像失忆，一切从头问。

wtodo 定了个三步闭环：

1. **收尾** —— 会话要结束时判定上下文是否超过 70%
2. **落盘** —— 超了就写/更新 cwd 里的 `todo_<yymmdd>.md`（沿用既有 todo 格式，旧内容折进 `<details>` 不丢）
3. **恢复** —— 新会话第一件事读最新那份 todo，3–5 行复述「上次做到哪 / 下一步 / 坑」，从没打勾的条目接着干

## 什么时候触发

| 场景 | 用户会说的话 |
|---|---|
| 会话收尾 | 收工 / 结束 / 先这样 / 就这样吧 / 再见 / done |
| 换会话 | 新会话 / 重启 session / 刷新 / 换个对话 / 重新开始 |
| 新会话开工 | 继续 / 接着上次 / 上次做到哪了 |
| 上下文吃紧 | 出现过 compaction、工具输出被截断、或已复述不出会话开头 |

## 关于「>70%」的诚实说明

**模型看不到 DSH 的精确上下文百分比**（没有对应的工具/接口）。所以 wtodo 不假装知道数字，而是走证据链，并在提醒里写明依据：

| 优先级 | 证据 | 结论 |
|---|---|---|
| 1 | 用户给的数字（「现在 78%」、贴了状态栏） | 以它为准 |
| 2 | 本会话出现过 compaction，或工具输出被截断 | 按 **>70%** 处理（默认 60% 就触发压缩） |
| 3 | 用户消息 > 25 轮，或复述不出开头 3 轮 | 按 **估算 >70%** 处理，注明是估算 |
| 4 | 以上都不成立 | 报「未达 70%」，只更新 todo，不劝换会话 |

拿不准就问一句：「顶栏显示的上下文是多少？」

## 里面有什么

| 段落 | 内容 |
|---|---|
| 铁律 | 工作不落盘，不许结束会话 |
| 判定表 | 上面那 4 级证据链 |
| 写 todo | `todo_261002.md` 落在会话 cwd；命名 = `todo_` + 两位年月日 |
| todo 格式 | `## 主题` 分组、`- [x]`/`- [ ]`、必须有「关键决定」和「坑」两节 |
| 提醒模板 | ≤4 行：百分比 + 依据 + 已写入的文件 + 建议开新会话 |
| 新会话恢复 | glob 取文件名最大的 `todo_*.md` → 复述进度 → 接着干，不重新问需求 |
| 别做过头 | 一次会话最多劝一次换会话；不删旧 todo；用户说「别写文件」就不写 |

## 安装

```powershell
# 全局：所有项目都能用
git clone https://github.com/huhahacn/wtodo "$env:USERPROFILE\.agents\skills\wtodo"

# 只给当前项目用
git clone https://github.com/huhahacn/wtodo ".\.dsh\skills\wtodo"
```

| 工具 | 技能根 |
|---|---|
| DSH | `~/.dsh/skills/`、`~/.agents/skills/`、`<项目>/.dsh/skills/`、`<项目>/.agents/skills/` |
| Claude Code | `~/.claude/skills/` |
| Codex | `~/.agents/skills/` |

也可以手动把 `SKILL.md` 放到 `<技能根>\wtodo\SKILL.md`。

## 用法

装好即自动触发。想手动叫：DSH 里输入 `/wtodo`，或直接说「写个 todo，我要换会话」。

配套文件长这样（放在会话工作目录）：

```markdown
# todo_261002.md

> 自动生成于 2026-10-02 · wtodo

## 工作区挂载插件
- [x] 修好 attach 按钮的 chip 插入（lib/client/chip.js）
- [ ] 补 route.test.js 的边界用例

## 关键决定
- 用 junction 而不是 symlink，Windows 免管理员

## 坑
- pwsh 在 workspace-write 沙箱下启动即 0xC0000142，命令必须提权
```

## 许可

MIT，见 [LICENSE](LICENSE)。随便用、随便改、随便发。
