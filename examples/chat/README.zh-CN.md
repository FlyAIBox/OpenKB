# Chat TUI

`openkb chat` 是基于 Wiki 的交互式 REPL。与 `query`（一次性）不同，chat 会话在轮次间保留上下文，可通过斜杠命令编辑知识库，并会保存以便稍后恢复。

```bash
openkb chat
```

```text
OpenKB Chat
~/research-kb · anthropic/claude-sonnet-4-6 · session 20260625-143022-a1x
Type /help for commands, Ctrl-D to exit, Ctrl-C to abort the current response.

>>> How do the two papers differ on their use of attention?
Both rely on scaled dot-product attention, but…
  · read_wiki_file(path="concepts/self-attention.md")
  · read_wiki_file(path="summaries/deepseek-r1.md")

>>> /save attention-comparison
Saved to wiki/explorations/attention-comparison-20260625.md
```

回答基于你的 Wiki：智能体会读取 `concepts/`、`summaries/`、`entities/` 和源文件，并展示它调用的工具。响应在终端中以富 Markdown（标题、表格、代码）渲染。

---

## 持久化会话

每次对话以 JSON 形式存储在 `<kb>/.openkb/chats/`。管理命令：

```bash
openkb chat --list              # table of sessions: id · turns · updated · title
openkb chat --resume            # resume the most recent session
openkb chat --resume 20260625   # resume by id or unique prefix
openkb chat --delete 20260625   # delete a session
```

恢复时会回放最近几轮，以便你获得上下文：

```text
$ openkb chat --resume
Resumed session · 4 turn(s)
[3] >>> How do the two papers differ on their use of attention?
[3]     Both rely on scaled dot-product attention, but…
[4] >>> /save attention-comparison
```

---

## 斜杠命令

在 REPL 中，以 `/` 开头的行是命令而非问题。运行 `/help` 查看当前可用命令；内置命令如下：

| 命令 | 作用 |
| --- | --- |
| `/help` | 列出可用命令 |
| `/exit`、`/quit` | 离开 REPL（Ctrl-D 也可） |
| `/clear` | 开始新会话（上一会话已保存） |
| `/save [name]` | 将对话导出到 `wiki/explorations/<name>-<date>.md` |
| `/status` | 在不离开 chat 的情况下显示 KB 状态 |
| `/list` | 列出 KB 中的文档 |
| `/lint` | 运行完整性 + 知识 lint |
| `/add <path>` | 导入文件或目录（Tab 补全路径） |
| `/skill new <name> "<intent>"` | 从 Wiki 编译技能 —— 见 [`skills/`](../skills/) |
| `/deck new [--critique] [--skill <name>] <name> "<intent>"` | 生成 HTML 幻灯片 —— 见 [`slides/`](../slides/) |
| `/critique <path>` | 对已有 deck/页面运行 HTML critic |

斜杠命令内联执行 —— 出错会报告且对话继续；Ctrl-C 中止当前命令而不结束会话。

### 斜杠命令为何重要

它们把 chat 变成工作台：提问、发现缺源文件、`/add` 导入、继续对话 —— 全在一个会话里完成。由于 chat 智能体可写入 `wiki/explorations/**` 和 `output/**`，让它「写成笔记」或「做成 skill」会产出你保留的真实文件。

---

## 纯文本输出（管道或日志）

```bash
openkb chat --no-color     # disable colored output entirely
openkb chat --raw          # show raw Markdown source, keep prompt/tool colors
```

`--no-color` 也尊重 `NO_COLOR` 环境变量。
