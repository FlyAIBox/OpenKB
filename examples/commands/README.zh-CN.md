# 命令参考

日常循环。每个命令以相同方式解析当前活跃的知识库（见 [`configuration/`](../configuration/#4-where-is-the-kb)），因此知识库初始化后，你可以从任意目录运行它们。

> **本文件夹中的真实产物：** [`sample-wiki/`](sample-wiki/) 是由 `openkb add` 从一篇论文编译出的完整 Wiki ——
> [1 篇摘要](sample-wiki/summaries/attention-is-all-you-need.md)、
> [3 个概念](sample-wiki/concepts/)、[9 个实体](sample-wiki/entities/)，
> 以及一条已保存的 [`query --save`](sample-wiki/explorations/) 回答。注意每个页面如何通过 `[[wikilinks]]` 串联在一起 —— 这种交叉链接*就是*你在 [`visualize/`](../visualize/) 中看到的知识图谱。

| 命令 | 用途 | 关键标志 |
| --- | --- | --- |
| `add <path\|dir\|URL>` | 导入文档 | `--from-pageindex-cloud` |
| `query <question>` | 一次性提问 | `--save`、`--raw` |
| `remove <id>` | 删除文档 | `--keep-raw`、`--keep-empty`、`--dry-run`、`--yes` |
| `recompile [doc]` | 重新运行编译流水线 | `--all`、`--dry-run`、`--yes`、`--refresh-schema` |
| `lint` | 检查 Wiki 完整性 | `--fix` |
| `list` | 显示已索引文档与页面 | – |
| `status` | 知识库统计 + 根路径 | – |
| `watch` | 自动导入拖入 `raw/` 的文件 | – |
| `feedback [msg]` | 提交预填 GitHub Issue | `--type` |

---

## `add` — 导入文档

```bash
openkb add ../docs/attention-is-all-you-need.pdf   # a single file
openkb add ~/papers/                               # a directory (recursive)
openkb add https://arxiv.org/pdf/2509.11420        # a URL
openkb add --from-pageindex-cloud <DOC_ID>         # an already-indexed cloud doc
```

- **支持的格式：** `.pdf .md .markdown .docx .pptx .xlsx .xls .html .htm
  .txt .csv`（以及 URL）。
- **URL** 按内容类型识别：PDF 会下载并索引；HTML 经主内容提取器（trafilatura）处理后以 Markdown 导入。
- **长/短 PDF** 由 `pageindex_threshold` 划分 —— 见 [`pageindex-cloud/`](../pageindex-cloud/)。
- **幂等：** 文档仅在编译成功后按内容哈希注册，因此重复添加同一文件会被跳过，失败的添加可以重试。

编译 [`../docs/attention-is-all-you-need.pdf`](../docs/attention-is-all-you-need.pdf)
生成了本文件夹中的 [`sample-wiki/`](sample-wiki/)。

---

## `query` — 一次性提问

```bash
openkb query "What are the main contributions of this paper?"
openkb query "Compare the two training objectives" --save
openkb query "How does this KB work?" --raw | less
```

- `--save` 将回答写入 `wiki/explorations/<slug>.md`，并在 frontmatter 中写入 `query:` 字段，因此优质回答会成为 Wiki 的一部分。
- `--raw` 输出原始 Markdown（无富文本渲染）—— 便于管道传输。
- 输出在终端中流式显示；管道或重定向时切换为纯文本最终答案，因此在脚本中使用是安全的。

> **示例：** 真实的 `--save` 结果见
> [`sample-wiki/explorations/`](sample-wiki/explorations/)。

---

## `remove` — 删除文档

按文件名、slug 或唯一子串识别文档：

```bash
openkb remove attention-is-all-you-need.pdf     # exact filename
openkb remove attention                         # unique substring
openkb remove attention --dry-run               # preview, change nothing
openkb remove attention --keep-empty            # keep concept & entity pages it solely sourced
openkb remove attention --keep-raw --yes        # leave raw/ file, no prompt
```

`remove` 会删除摘要、sources 和提取的图片；从每个概念/实体页面的 `sources:` 中移除该文档（若该文档是**唯一**来源则删除对应页面，除非使用 `--keep-empty`）；修剪 `index.md` 和哈希注册表；并运行范围限定的 `lint --fix` 以清理悬空的 `[[wikilinks]]`。对于本地长 PDF，还会清除 PageIndex 状态。不确定时请先使用 `--dry-run`。

---

## `recompile` — 重新生成 Wiki 页面

对已导入内容重新运行编译步骤（不重新转换、不重新索引）：

```bash
openkb recompile attention                      # one document
openkb recompile --all --dry-run                # preview the full set
openkb recompile --all --yes                    # rebuild everything
openkb recompile --all --refresh-schema         # also refresh wiki/AGENTS.md
```

> ⚠️ 重新编译会**覆盖**生成的摘要和概念页面 —— 对手动编辑的内容会丢失。`--refresh-schema` 在替换前会将旧的 `AGENTS.md` 备份为 `AGENTS.md.bak`。

---

## `lint` — 检查（并修复）Wiki 完整性

```bash
openkb lint            # report only
openkb lint --fix      # repair broken wikilinks first, then report
```

检查悬空的 `[[wikilinks]]`、孤立页面、无 Wiki 条目的 raw 文件、`index.md` 漂移、无效 frontmatter，以及 LLM 驱动的知识检查。报告写入 `wiki/reports/lint_<timestamp>.md`。`--fix` 会对损坏链接进行模糊匹配，或在无匹配时将其剥离为纯文本。

---

## `list` 与 `status`

```bash
openkb status
```

```text
Knowledge base: /Users/you/my-kb

Knowledge Base Status:
  Directory            Files
  -------------------- ----------
  sources              1
  summaries            1
  concepts             3
  entities             9
  reports              0
  raw                  1

  Total indexed: 1 document(s)
  Last compile:  2026-06-25 14:30:22
```

`openkb list` 打印文档表（名称 · 类型 · 页面数 —— 长 PDF 与 cloud 导入均显示为 `pageindex`），随后列出已编译的摘要、概念、实体和报告。（上述计数与 [`sample-wiki/`](sample-wiki/) 一致。）

---

## `watch` — 拖放式导入

```bash
openkb watch
# in another terminal:  cp new-paper.pdf raw/   → auto-compiles
```

监视 `raw/`，对每个新的受支持文件运行 `add`，直到你按 Ctrl-C。

---

## `feedback` — 报告问题

```bash
openkb feedback "add support for EPUB" --type feature
openkb feedback                      # interactive
```

在浏览器中打开**预填**的 GitHub Issue（标题、正文、OpenKB/Python 版本和平台等非敏感诊断信息）—— 由你用自己的账号提交。`--type` 可选 `bug`、`feature`、`question`、`other`。在非交互式 shell 中安全（不会在类型提示处挂起）。
