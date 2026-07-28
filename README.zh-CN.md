<div align="center">

<a href="https://openkb.ai">
  <img src="https://docs.pageindex.ai/images/openkb.png" alt="OpenKB（由 PageIndex 提供支持）" />
</a>

<br />
<br />

<p align="center">
<a href="https://trendshift.io/repositories/26145" target="_blank"><img src="https://trendshift.io/api/badge/repositories/26145" alt="VectifyAI%2FOpenKB | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
</p>

# OpenKB：开放式 LLM 知识库

<p align="center"><i>支持长文档扩展  •  基于推理的检索  •  原生多模态  •  无需向量数据库</i></p>

</div>

<details open>
<summary><h2>📢 近期更新</h2></summary>

- *Google 开放知识格式（OKF）*：Wiki 页面遵循 [Google OKF](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing) 规范，便于知识共享。
- *实体页面*：将人物、组织、地点和产品自动提取为独立 Wiki 页面，并持续保持同步。

</details>

---

# 📑 什么是 OpenKB

**OpenKB（开放知识库）**是一个开源 CLI 系统，使用 LLM 将原始文档编译为结构化、相互链接的 Wiki 式知识库。它由 [**PageIndex**](https://github.com/VectifyAI/PageIndex) 提供支持，针对长文档采用无需向量的、基于推理的检索方式。

这一理念源于 Andrej Karpathy 描述的一个[构想](https://x.com/karpathy/status/2039805659525644595)：由 LLM 生成摘要、概念页面和交叉引用，并自动维护所有内容。知识会随时间不断积累，而不必在每次查询时重新推导。

### 为什么不用传统 RAG？

传统 RAG 每次查询都从头重新发现知识，无法形成积累。OpenKB 只需将知识编译一次，保存为持久化 Wiki，并持续更新。交叉引用已经建立，矛盾会被标记，综合分析也会纳入所有已读取的内容。

OpenKB 分为两层：负责汇编和维护知识的 **Wiki 基础层**，以及将知识转化为实用输出的**生成器**（查询 / 对话 / 技能工厂）。完整命令列表请参阅[使用方法](#️-使用方法)。

### 功能特性

- **广泛的格式支持：** PDF、Word、Markdown、PowerPoint、HTML、Excel、CSV、纯文本、URL 等。
- **可扩展至长文档：** 通过 [PageIndex](https://github.com/VectifyAI/PageIndex) 树状索引处理长篇复杂文档，实现准确、无需向量且感知上下文的检索。
- **原生多模态：** 不仅能检索和理解文本，也能处理图表、表格和图像。
- **编译式 Wiki：** LLM 将文档编译为摘要、概念页面、实体页面和交叉链接，并保持同步。
- **查询与对话：** 支持一次性提问，也支持基于 Wiki 的多轮对话；会话可持久化并恢复。
- **技能工厂：** 从 Wiki 中提炼可分发的智能体技能。
- **兼容 OKF：** Wiki 页面遵循 [Google OKF](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing) 知识共享规范。
- **兼容 Obsidian：** Wiki 由包含交叉链接的纯 `.md` 文件组成，可在 Obsidian 中打开并使用图谱视图。
- **知识工作台（Web UI）：** 内置 Web UI，通过 `/` 提供服务，可在浏览器中浏览知识库、上传并编译文档，以及以流式方式进行查询和对话。

# 🚀 快速上手

### 安装

```bash
pip install openkb
```

<details>
<summary><b><i>其他安装方式：</i></b></summary>

- **从 GitHub 安装最新版：**

  ```bash
  pip install git+https://github.com/VectifyAI/OpenKB.git
  ```

- **从源码安装**（开发用，可编辑模式）：

  ```bash
  git clone https://github.com/VectifyAI/OpenKB.git
  cd OpenKB
  pip install -e .
  ```

</details>

### 快速开始

```bash
# 1. 创建知识库目录
mkdir my-kb && cd my-kb

# 2. 初始化知识库
openkb init

# 3. 添加文档
openkb add paper.pdf
openkb add ~/papers/                            # 添加整个目录
openkb add https://arxiv.org/pdf/2509.11420     # 或从 URL 获取

# 4. 提问
openkb query "主要研究发现是什么？"

# 5. 或进行交互式对话
openkb chat

# （可选）将 Wiki 转换为其他形式的输出
openkb skill new my-expert "像 <你的主题> 专家一样推理"   # 可移植的智能体技能
openkb visualize                                      # 交互式知识图谱
openkb deck new my-deck "<你的主题> 入门介绍"           # 幻灯片——单文件 HTML 演示文稿
```

### 设置 LLM

OpenKB 通过 [LiteLLM](https://github.com/BerriAI/litellm)（已固定到一个[安全版本](https://docs.litellm.ai/blog/security-update-march-2026)）支持[多种 LLM 提供商](https://docs.litellm.ai/docs/providers)，包括 OpenAI、Claude、Gemini 等。

在执行 `openkb init` 时，或在 [`.openkb/config.yaml`](#配置) 中，使用 LiteLLM 的 `provider/model` 格式设置模型（例如 `anthropic/claude-sonnet-4-6`）。OpenAI 模型可以省略提供商前缀（例如 `gpt-5.4`）。

创建 `.env` 文件并填入 LLM API 密钥：

```bash
LLM_API_KEY=your_llm_api_key
```

使用 OAuth 设备流程认证的订阅制提供商（例如 `chatgpt/*`、`github_copilot/*`）不需要 API 密钥；OpenKB 不会对它们发出缺少密钥的警告。

### 知识工作台（Web UI）

OpenKB 内置 Web UI，由 REST API 在 `/` 路径提供服务。安装 API 扩展并启动服务器即可，无需配置：

```bash
pip install "openkb[web]"
openkb-web                       # 在 http://127.0.0.1:7566/ 提供 API 和工作台
```

在浏览器中打开 `http://127.0.0.1:7566/` 即可使用工作台。默认关闭身份验证（本地优先）；将服务器暴露到外部前，请设置 `OPENKB_API_TOKEN` 以要求使用 Bearer Token。参阅[完整 Web UI 指南](examples/rest-api/README.md#knowledge-workbench-web-ui)。

> 要开发 UI 本身？请运行 Vite 开发服务器：`cd frontend && npm install && npm run dev`（它会将 `/api` 代理到正在运行的 `openkb-web`）；也可运行 `npm run build` 重新生成内置的 `openkb/web/`。

# 🧩 OpenKB 的工作原理

### 架构

<div align="center">
  <img src="assets/openkb-architecture.webp" alt="OpenKB 架构：原始文档经 markitdown / PageIndex 处理，由 LLM 编译为 Wiki 基础层，再为查询、对话、技能工厂和未来的生成器提供支持" width="900" />
</div>

### 短文档与长文档的处理方式

|              | 短文档                    | 长文档（PDF ≥ 20 页）          |
| ------------ | ------------------------- | ------------------------------ |
| **转换**     | markitdown → Markdown     | PageIndex → 树状索引 + 摘要    |
| **图像**     | 内联提取（pymupdf）       | 由 PageIndex 提取              |
| **LLM 读取** | 完整文本                  | 文档树                         |
| **结果**     | 摘要 + 概念               | 摘要 + 概念                    |

LLM 会完整读取短文档。长 PDF 则由 [PageIndex](https://github.com/VectifyAI/PageIndex) 处理为分层树状索引。LLM 读取文档树而非完整文本，从而对长文档进行准确、可扩展的检索。

### 知识编译

添加文档时，LLM 会：

1. 生成**摘要**页面
2. 读取已有的**概念**页面和**实体**页面
3. 通过跨文档综合分析创建或更新概念
4. 创建或更新**实体**页面（人物、组织、地点、产品）
5. 更新**索引**和**日志**

单个来源可能涉及 10—15 个 Wiki 页面。知识会持续积累：每份文档都会丰富现有 Wiki，而不是孤立存放。

# ⚙️ 使用方法

OpenKB 命令分为两层：**Wiki 基础层**（编译和管理知识）与**生成器**（将 Wiki 转化为实用输出）。每项功能都链接到具体演练，其中包含 OpenKB 基于同一篇示例论文生成的真实产物（可在 [`examples/`](examples/) 中浏览全部示例）。

## 第一层：🧱 Wiki 基础层——编译和维护

| 命令                                                         | 说明                                                                    |
| ------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `openkb init`                                                | 初始化新知识库（交互式）                                                |
| <code>openkb&nbsp;add&nbsp;&lt;file_or_dir_or_URL&gt;</code> | 添加文件、目录或 URL 并编译为 Wiki（自动检测 URL 内容类型）              |
| `openkb list`                                                | 列出已索引的文档和概念                                                  |
| `openkb status`                                              | 显示知识库统计信息                                                      |
| `openkb watch`                                               | 监视 `raw/` 并自动编译新文件                                            |
| `openkb lint`                                                | 执行结构和知识健康检查                                                  |

<details>
<summary><i>更多 Wiki 命令：</i></summary>
<br>

| 命令                                                               | 说明                                                                                                                                                                                                |
| ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <code>openkb&nbsp;remove&nbsp;&lt;doc&gt;</code>                   | 删除文档，并清理其 Wiki 页面、图像、注册表和 PageIndex 状态（使用 `--dry-run` 预览，使用 `--keep-raw` / `--keep-empty` 保留产物）                                                                    |
| <code>openkb&nbsp;recompile&nbsp;[&lt;doc&gt;]&nbsp;[--all]</code> | 对已索引文档重新运行编译流水线，无需重新索引。它会重新生成摘要并改写概念页面；手工编辑将被覆盖（使用 `--dry-run` 预览，使用 `--refresh-schema` 同时更新 `wiki/AGENTS.md`）                              |
| <code>openkb&nbsp;feedback&nbsp;["msg"]</code>                     | 打开预填内容的 GitHub Issue 以提交反馈（使用 `--type bug/feature/question` 添加类型标签）                                                                                                           |

</details>

→ **示例：** 完整演示日常使用流程——[`examples/commands/`](examples/commands/)。

## 第二层：💡 生成器——将 Wiki 转化为输出

“生成器”读取已编译的 Wiki 并生成可用内容，例如答案、对话或技能目录。Wiki 是底层基质，生成器则是面向用户的使用界面。

| 命令                                                                                  | 输出                                                                                                               | 示例                               |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| <code>openkb&nbsp;query&nbsp;"question"</code>                                        | 基于知识库且带引用的答案（使用 `--save` 保存至 `wiki/explorations/`）                                               | [查询并保存](examples/commands/)   |
| <code>openkb&nbsp;chat</code>                                                         | 基于 Wiki 的交互式多轮会话（使用 `--resume`、`--list`、`--delete` 管理会话）                                        | [对话](examples/chat/)             |
| <code>openkb&nbsp;visualize</code>                                                    | 位于 `output/visualize/graph.html` 的独立交互式知识图谱，提供 3D、思维导图和径向视图                                 | [可视化](examples/visualize/)      |
| <code>openkb&nbsp;skill&nbsp;new&nbsp;&lt;skill-name&gt;&nbsp;"&lt;intent&gt;"</code> | 从 Wiki 中提炼可分发的智能体技能（参阅下方[技能工厂](#skill-factory)）                                               | [技能](examples/skills/)           |
| <code>openkb&nbsp;deck&nbsp;new&nbsp;&lt;name&gt;&nbsp;"&lt;intent&gt;"</code>        | 生成单文件 HTML 演示文稿（`--skill` 选择主题，`--critique` 执行质量审查）                                            | [幻灯片](examples/slides/)         |

### (i) 💬 查询与对话——*向 Wiki 提问*

`openkb query "..."` 根据 Wiki 回答单个问题，并提供有依据、带引用的答案。`openkb chat` 则用于基于同一 Wiki 开展持续的交互式多轮会话（使用 `--resume`、`--list`、`--delete` 管理会话）。→ **[`examples/commands/`](examples/commands/)**（查询）和 **[`examples/chat/`](examples/chat/)**（对话）以真实保存的输出完整演示了这些功能。

在对话中输入 `/` 可使用斜杠命令（按 Tab 补全）。

<details>
<summary><i>更多斜杠命令：</i></summary>
<br>

- `/help`：列出可用命令
- `/status`：显示知识库状态
- `/list`：列出所有文档
- `/add <path>`：不退出对话，直接添加文档或目录
- `/skill new <skill-name> "<intent>"`：根据当前对话编译技能（见下文）
- `/deck new <name> "<intent>"`：根据 Wiki 生成 HTML 演示文稿
- `/critique <path>`：对现有演示文稿运行 HTML 评审器
- `/save [name]`：将对话记录导出到 `wiki/explorations/`
- `/clear`：开始新会话（当前会话仍保存在磁盘上）
- `/lint`：运行知识库检查
- `/exit`：退出（也可按 Ctrl-D）

</details>

<a id="skill-factory"></a>

### (ii) 🛠 技能工厂——*放入一本书，得到一位数字专家。*

`openkb skill new` 可从 Wiki 中提炼可移植的[智能体技能](https://docs.claude.com/en/docs/build-with-claude/skills)，供 Claude Code、Codex 和 Gemini 原生安装与加载。投入足以成书的论文，就能得到其他智能体可以调用的专业助手。→ **[`examples/skills/`](examples/skills/)** 完整演示了真实生成的技能，以及安装、分享、`eval` 和回滚流程。

<details>
<summary><i>更多技能命令：</i></summary>
<br>

| 命令                                                                                                                           | 输出                                              |
| ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------- |
| <code>openkb&nbsp;skill&nbsp;validate&nbsp;[name]</code>                                                                       | 验证已编译的技能（在 `skill new` 后自动运行）      |
| <code>openkb&nbsp;skill&nbsp;eval&nbsp;&lt;name&gt;</code>                                                                     | 检查技能能否由正确的提示词触发                    |
| <code>openkb&nbsp;skill&nbsp;history&nbsp;&lt;name&gt;</code> / <code>openkb&nbsp;skill&nbsp;rollback&nbsp;&lt;name&gt;</code> | 查看技能版本历史及执行回滚                        |

</details>

# 🔧 配置

### 设置

OpenKB 设置由 `openkb init` 初始化，并存储在 `.openkb/config.yaml` 中：

```yaml
model: gpt-5.4                   # LLM 模型（任意 LiteLLM 支持的提供商）
language: en                     # Wiki 输出语言
pageindex_threshold: 20          # 启用 PageIndex 的 PDF 页数阈值
```

完整设置参考，包括 `entity_types`、OAuth 提供商（`chatgpt/*`、`github_copilot/*`），以及 LiteLLM 调优选项（Ollama / LM Studio 等较慢本地运行时的超时设置、`drop_params`、GitHub Copilot 请求头和安装说明），请参阅 **[`examples/configuration/`](examples/configuration/)**。

### PageIndex 设置

长文档检索是 LLM 面临的一个[已知挑战](https://x.com/karpathy/status/2039823314982744522)。[PageIndex](https://github.com/VectifyAI/PageIndex) 通过无需向量、基于推理的检索方式解决这一问题：它构建分层树状索引，让 LLM 能在索引上进行推理，实现感知上下文的检索。

PageIndex 默认使用[开源版本](https://github.com/VectifyAI/PageIndex)在本地运行，无需任何外部依赖。

***云服务支持***（可选）：

对于大型或复杂 PDF，可使用 [PageIndex Cloud](https://docs.pageindex.ai/) 获得更多能力，包括：

- 对扫描版 PDF 的 OCR 支持（通过托管 VLM 模型）
- 更快的结构生成
- 面向大型文档的可扩展索引

在 `.env` 中设置 `PAGEINDEX_API_KEY` 以启用云端功能：

```
PAGEINDEX_API_KEY=your_pageindex_api_key
```

→ **示例：** 本地索引与云端索引对比，以及导入已在云端建立索引的文档——[`examples/pageindex-cloud/`](examples/pageindex-cloud/)。

### AGENTS.md

`wiki/AGENTS.md` 文件定义 Wiki 的结构和约定，是 LLM 维护 Wiki 时使用的操作手册。你可以自定义该文件来改变 Wiki 的组织方式。

LLM 会在运行时从磁盘读取 `AGENTS.md`，因此编辑会立即生效。

# 🔌 集成

### 与 Obsidian 配合使用

Wiki 是一个由 Markdown 文件组成的目录，其中包含 `[[Wiki 链接]]`。Obsidian 可以原生渲染它。

1. 将 `wiki/` 作为 Obsidian 仓库打开
2. 浏览摘要、概念和探索记录
3. 使用图谱视图查看知识之间的联系
4. 使用 Obsidian Web Clipper 将网页文章添加到 `raw/`

### 与 Claude Code / Codex / Gemini CLI 配合使用

OpenKB 自带 `SKILL.md`，任何智能体都可借此读取已编译的 Wiki。无需额外运行时，也无需配置 MCP，只需安装一次技能。

<details>
<summary><i>Claude Code：</i></summary>
<br>

```
/plugin marketplace add VectifyAI/OpenKB
/plugin install openkb@vectify
```

</details>

<details>
<summary><i>OpenAI Codex CLI：</i></summary>
<br>

*（目前没有市场安装命令，请手动创建符号链接）*

```bash
git clone https://github.com/VectifyAI/OpenKB.git ~/openkb-src
mkdir -p ~/.agents/skills
ln -s ~/openkb-src/skills/openkb ~/.agents/skills/openkb
```

</details>

<details>
<summary><i>Gemini CLI：</i></summary>
<br>

```bash
gemini skills install https://github.com/VectifyAI/OpenKB.git --path skills/openkb --consent
```

</details>

该技能是只读的。除非你明确要求，否则它不会运行 `openkb add`、`remove` 或 `lint --fix`。完整指令集请参阅 [`skills/openkb/SKILL.md`](skills/openkb/SKILL.md)。

# REST API

OpenKB 为 HTTP 客户端提供 FastAPI 服务。使用 `pip install -e ".[web]"` 安装，然后通过 `python -m openkb.api` 启动。交互式 API 参考文档位于 [`/docs`](http://127.0.0.1:7566/docs)，也可导入 Postman。

有关端点、身份验证和 SSE 流式传输，请参阅[完整 REST API 参考](examples/rest-api/README.md#rest-api)。

# 🧭 进一步了解

### 与 Karpathy 的方案对比

|          | Karpathy 的工作流          | OpenKB                                            |
| -------- | -------------------------- | ------------------------------------------------- |
| 短文档   | LLM 直接读取               | markitdown → LLM 读取                             |
| 长文档   | 上下文限制、上下文退化     | PageIndex 树状索引                                |
| 输入来源 | Web Clipper → .md           | PDF、Word、PPT、Excel、HTML、文本、CSV、.md、URL |
| Wiki 编译 | LLM 智能体                 | LLM 智能体（相同）                                |
| 实体提取 | 手动                       | 自动（人物、组织、地点、产品）                    |
| 问答     | 在 Wiki 上查询             | Wiki + PageIndex 检索                             |
| 输出     | 仅 Wiki                    | Wiki + 技能工厂 + 智能体 CLI 集成                 |

### 技术栈

- [PageIndex](https://github.com/VectifyAI/PageIndex)——无需向量、基于推理的文档索引和检索
- [markitdown](https://github.com/microsoft/markitdown)——通用文件转 Markdown 工具
- [OpenAI Agents SDK](https://github.com/openai/openai-agents-python)——智能体框架（通过 LiteLLM 支持非 OpenAI 模型）
- [LiteLLM](https://github.com/BerriAI/litellm)——支持多提供商的 LLM 网关
- [Click](https://click.palletsprojects.com/)——CLI 框架
- [watchdog](https://github.com/gorakhargosh/watchdog)——文件系统监控

### 路线图

- [ ] 将长文档处理扩展到非 PDF 格式
- [ ] 通过嵌套目录支持扩展至大型文档集合
- [ ] 为超大型知识库提供分层概念（主题）索引
- [ ] 基于数据库的存储引擎
- [x] 用于浏览和管理 Wiki 的 Web UI（知识工作台，通过 `/` 提供服务）

### 参与贡献

欢迎贡献！你可以提交 Pull Request，或通过 [Issue](https://github.com/VectifyAI/OpenKB/issues) 报告错误和提出功能请求。对于较大的改动，建议先创建 Issue 讨论实现方案。

### 许可证

采用 Apache 2.0 许可证。详见 [LICENSE](LICENSE)。

### 🌐 开源生态

PageIndex 生态中的其他[开源项目](https://docs.pageindex.ai/open-source)：

- [PageIndex](https://github.com/VectifyAI/PageIndex)：面向长文档、无需向量且基于推理的 RAG 框架
- [ChatIndex](https://github.com/VectifyAI/ChatIndex)：面向长对话历史和记忆的树状索引与检索
- [ConDB](https://github.com/VectifyAI/ConDB)：面向大规模树状检索、原生支持 KV 缓存的上下文数据库
- [PageIndex MCP](https://github.com/VectifyAI/pageindex-mcp)：PageIndex 的 MCP 服务器

### 支持我们

如果 OpenKB 对你有帮助，请为我们点亮一颗星 🌟，也欢迎关注 [**PageIndex**](https://github.com/VectifyAI/PageIndex)！  

<div>

[![Twitter](https://img.shields.io/badge/Twitter-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/PageIndexAI)&ensp;
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/company/vectify-ai/)&ensp;
[![联系我们](https://img.shields.io/badge/Contact_Us-3B82F6?style=for-the-badge&logo=envelope&logoColor=white)](https://ii2abc2jejf.typeform.com/to/tK3AXl8T)

</div>
