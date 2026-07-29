# Skill Factory

Skill Factory 将你的 Wiki 提炼为**可再分发的 agent skill** —— 一个自包含目录，Claude Code、Codex 和 Gemini CLI 可原生加载。`query`/`chat` 用于回答问题，skill 则把你的知识打包，让*另一个*智能体基于它推理。

> **本文件夹中的真实产物：** [`transformer-attention/`](transformer-attention/)
> 由 `openkb skill new` 从一篇论文生成 —— 实际的
> [`SKILL.md`](transformer-attention/SKILL.md)、其
> [`references/`](transformer-attention/references/)，
> 以及写入的 [`marketplace.json`](marketplace.json)。打开 `SKILL.md`：它是一组*决策规则*和精确的触发描述，而非摘要。

一个 skill 的结构是：

```
output/skills/<name>/
├── SKILL.md          # frontmatter (name + description) + the worldview/decision rules
└── references/       # optional supporting pages (resolved wikilinks)
```

---

## 创建 skill

```bash
openkb skill new <name> "<intent>"
```

- `<name>` — kebab-case slug（也是 skill 名称和输出文件夹名）。
- `<intent>` — 用自然语言描述 skill 应做什么；它既决定生成器从 Wiki 拉取哪些内容，也塑造 skill 的 `description:`（即智能体中*触发*它的依据）。

```bash
openkb skill new transformer-attention \
  "Reason about the Transformer architecture — self-attention, multi-head \
   attention, positional encoding — and why attention replaced recurrence"
```

```text
Compiling skill 'transformer-attention'...
Saved: output/skills/transformer-attention/
Manifest: .claude-plugin/marketplace.json updated

Install locally:   cp -r output/skills/transformer-attention ~/.claude/skills/
Share:             push your KB to GitHub, then  npx skills@latest add <owner>/<repo>
```

该命令在本文件夹中生成了 [`transformer-attention/`](transformer-attention/) 和
[`marketplace.json`](marketplace.json)。对已存在的 skill 重新编译（`--yes` 跳过覆盖提示）会将旧版本备份到 `output/skills/<name>-workspace/iteration-N/`，并附带 `diff.md`。

---

## 迭代、验证、评估

```bash
openkb skill history transformer-attention            # list saved iterations
openkb skill rollback transformer-attention --to 1    # restore iteration-1
openkb skill validate transformer-attention --strict  # structural checks
openkb skill eval transformer-attention --save        # measure quality
```

- **`validate`** 仅做结构检查（无 LLM）：frontmatter 有效、名称为 kebab-case、description 长度、文件大小上限，以及 wikilinks 可解析。`--strict` 将警告转为非零退出码 —— 适合 CI。省略名称则验证 KB 中所有 skill。
- **`eval`** 生成 trigger/no-trigger 提示并评估两项：

```text
$ openkb skill eval transformer-attention --save --count 15
Generating eval set for 'transformer-attention' (count=15 per side)...

Eval set: 30 prompts
Trigger accuracy: 28/30 (93%) — does the description fire on the right questions?
Body coverage:    27/28 (96%) — does SKILL.md actually support what it promises?

Trigger misses (2):
  - [should-trigger → no] "How does attention scale with sequence length?"
  - [should-not   → yes] "What's the capital of France?"

Eval set persisted to .openkb/eval-sets/transformer-attention.json
```

`--save` 让你稍后对*同一* eval set 重跑（`--eval-set <path>`），以便迭代间 apples-to-apples 对比。

---

## 生成 skill 的结构

frontmatter 是智能体匹配的依据 —— `description` 精确说明*何时*触发、*何时不*触发。本文件夹中 [`transformer-attention/SKILL.md`](transformer-attention/SKILL.md) 的示例：

```yaml
---
name: transformer-attention
description: Use when reasoning about Transformer self-attention, multi-head
  attention, positional encoding, masked decoder attention, or why attention
  replaced recurrence/convolutions in sequence models; not for generic NLP or
  unrelated attention topics.
---
```

……随后是诸如*「当 recurrence 成为瓶颈时，优先采用纯 attention 计算」*的决策规则，以及显式的 **Known gaps** 小节。每次编译还会（重新）写入插件清单 `.claude-plugin/marketplace.json`（[本文件夹副本](marketplace.json)），`npx skills add` 安装的就是它。

> OpenKB 自身也附带提炼好的 skill —
> [`../../skills/openkb/SKILL.md`](../../skills/openkb/SKILL.md) —— 教智能体导航任意 OpenKB Wiki 的那一个。

---

## 分发

```bash
# Locally, for your own Claude Code:
cp -r output/skills/transformer-attention ~/.claude/skills/

# To others — push the KB to GitHub, then they run:
npx skills@latest add <owner>/<repo>
```

安装后，当智能体任务匹配其 `description` 时 skill 会自动加载 —— 这正是 `skill eval` 所衡量的。

> **注意：** `marketplace.json` 的 `owner`/`author` 来自本地 git 配置。本文件夹中的副本已 scrub 为 `Your Name` / `you@example.com`。
