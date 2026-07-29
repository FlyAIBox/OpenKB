# OpenKB 示例

按用例组织的动手材料。每个文件夹是一个案例——简短走查，外加 OpenKB
为该案例产出的**真实产物**（由系统生成，非手写）。

> **测试输入**在 [`docs/`](docs/) — 各案例会摄入的示例 PDF。
> 故意不进 git（体积大 / 第三方文件）；自行放入文档即可跟着做。

## 案例

| 文件夹 | 展示内容 | 内含真实产物 |
| --- | --- | --- |
| [`configuration/`](configuration/) | `init`、`config.yaml`、API 密钥、LiteLLM 调优（Ollama、Copilot） | — |
| [`commands/`](commands/) | 日常循环：`add` · `query` · `remove` · `recompile` · `lint` · `list` · `status` | 已编译的 [`sample-wiki/`](commands/sample-wiki/) |
| [`pageindex-cloud/`](pageindex-cloud/) | 长文档：本地 vs 云端索引，以及导入已云端索引的文档 | — |
| [`chat/`](chat/) | 交互式 REPL：持久会话 + 斜杠命令 | — |
| [`skills/`](skills/) | 从 wiki 蒸馏可分发的 agent skill | [`transformer-attention/SKILL.md`](skills/transformer-attention/SKILL.md) |
| [`slides/`](slides/) | 生成单文件 HTML 幻灯片 | [`attention-intro.html`](slides/attention-intro.html) |
| [`visualize/`](visualize/) | 将 wiki 渲染为交互式知识图谱 | [`graph.html`](visualize/graph.html) |

## 全部来自同一篇论文

编译后的 `sample-wiki/`、skill、幻灯片和图谱，**都**是对同一份文档跑
OpenKB 得到的 —
[`docs/attention-is-all-you-need.pdf`](docs/attention-is-all-you-need.pdf) —
使用 `gpt-5.4`：

```bash
mkdir my-kb && cd my-kb
openkb init --model gpt-5.4 --language en

openkb add /path/to/examples/docs/attention-is-all-you-need.pdf   # → commands/sample-wiki/
openkb query "How does attention replace recurrence?" --save      # → commands/sample-wiki/explorations/
openkb skill new transformer-attention "Reason about the Transformer…"  # → skills/
openkb deck new attention-intro "A short intro deck on the Transformer…" # → slides/
openkb visualize                                                  # → visualize/
```

每次运行生成的文本会不同（LLM 输出非确定性）——稳定的是*结构*。

刚接触 OpenKB？先读 [`configuration/`](configuration/)，再读
[`commands/`](commands/)。
