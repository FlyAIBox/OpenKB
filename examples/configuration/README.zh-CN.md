# 配置与安装

控制 OpenKB 如何与 LLM 通信的内容，都在两处：
`.openkb/config.yaml`（模型、语言、调优）和 `.env`（API 密钥）。

---

## 安装

```bash
pip install openkb
```

OpenKB 钉死了其 PageIndex 依赖的一个**预发布**版本
（`pageindex==0.3.0.dev3`），部分安装器默认会跳过。若安装无法解析
`pageindex`，请允许预发布：

```bash
uv tool install openkb --prerelease=allow   # uv
pip install --pre openkb                     # pip
```

若安装成功后仍找不到 `openkb`，说明 console-script 目录不在 `PATH` 上
（例如 `pip --user` 会装到 `~/.local/bin`）——把它加进 `PATH`。

---

## 1. 初始化知识库

```bash
mkdir my-kb && cd my-kb
openkb init
```

在终端中 `init` 是交互式的，会询问三项：

- **Model** — LiteLLM 的 `provider/model` 格式。OpenAI 模型可省略前缀
  （`gpt-5.4`）；其他需要前缀（`anthropic/claude-sonnet-4-6`、
  `gemini/gemini-3-flash-preview`）。
- **LLM API key** — 隐藏输入；若提供会以 `0600` 权限写入 `.env`。按 Enter
  可跳过，稍后设置。
- **Language** — wiki 输出语言。任意语言均可；例如联合国六种官方语言：
  `en`（英语）、`zh`（中文）、`es`（西班牙语）、`fr`（法语）、`ar`（阿拉伯语）、
  `ru`（俄语）。

用标志完全跳过提示——适合脚本：

```bash
openkb init --model anthropic/claude-sonnet-4-6 --language en
openkb init -m gpt-5.4 -l zh
```

> **非交互（管道/CI）：** 提示受 TTY 门控。stdin 不是终端时，`init` 使用默认值
> 而不会挂起，因此 `printf 'gpt-5.4\n\nen\n' | openkb init` 可在脚本中使用。

`init` 会创建：`raw/`、`wiki/{summaries,concepts,entities,sources/images}`、
`wiki/AGENTS.md`、`wiki/index.md`、`wiki/log.md`，以及 `.openkb/config.yaml`。

---

## 2. `.openkb/config.yaml` 参考

`init` 写出的文件很小；其余均为可选。以下是仓库自带的
[`config.yaml.example`](../../config.yaml.example)，原文照录：

```yaml
model: gpt-5.4                   # LLM model (any LiteLLM-supported provider)
language: en                     # Wiki output language
pageindex_threshold: 20          # PDF pages threshold for PageIndex

# Optional: cap concurrent LLM calls during ingest (PageIndex indexing and
# concept/entity compilation — they never overlap, so one setting covers
# both). Lower it if you hit provider rate limits or "too many open files" on
# large PDFs. Omit to let each stage apply its own default.
# concurrency: 5

# Optional: whether the LLM agents (query, chat, lint, skill) may call tools
# in parallel. Leave it UNSET (commented out) to keep OpenKB's per-agent
# defaults. Setting it applies the SAME value to every agent:
#   true    allow parallel tool calls
#   false   force sequential tool calls
#   null    don't send the setting at all (use the provider default) — REQUIRED
#           for Amazon Bedrock Claude, which rejects the request when
#           parallel_tool_calls is sent at all (any value). See #175.
# parallel_tool_calls: null

# Optional: override the entity-type vocabulary used for entity pages.
# Omit this key to use the default 7 types
# (person, organization, place, product, work, event, other).
# entity_types:
#   - person
#   - organization
#   - dataset
#   - model

# Optional: LLM / LiteLLM tuning. Keys are forwarded to LiteLLM; `timeout` and
# `extra_headers` apply per request, the rest are set as litellm.<key>.
# litellm:
#   timeout: 1200          # per-request timeout (s); raise for slow local backends (Ollama)
#   drop_params: true      # let LiteLLM drop params a provider rejects (e.g. Ollama)
#   num_retries: 3
#   extra_headers:         # extra HTTP headers some providers need (e.g. GitHub Copilot)
#     Editor-Version: vscode/1.95.0
#     Copilot-Integration-Id: vscode-chat
```

| 键 | 默认值 | 作用 |
| --- | --- | --- |
| `model` | `gpt-5.4` | 所有编译/查询/对话所用的 LLM。 |
| `language` | `en` | wiki 撰写语言。 |
| `pageindex_threshold` | `20` | 页数达到该值**或更多**的 PDF 走长文档（PageIndex）路径；更短的走短文档路径。见 [`pageindex-cloud/`](../pageindex-cloud/)。 |
| `concurrency` | `null` | 限制摄入期间 OpenKB 并发 LLM 调用数——覆盖 PageIndex 对长文档的索引，以及 OpenKB 自身的概念/实体编译。同一文档两者不会同时跑，因此一个设置即可。遇到提供商速率限制或大 PDF 的 “too many open files” 时可调低。`null` 让各阶段用自己的默认值。 |
| `parallel_tool_calls` | 未设置 | LLM agent（query、chat、lint、skill）是否可并行调用工具。未设置则保留 OpenKB 各 agent 默认；`true`/`false` 对所有 agent 强制允许/串行；`null` 省略该设置（用提供商默认）。**Amazon Bedrock 需要 `null`**（见下）。 |
| `entity_types` | 7 个默认类型 | 实体页的自定义词汇表。`other` 始终保留。 |
| `litellm:` | – | 透传给 LiteLLM 的块。见下。 |

### `litellm:` 块

OpenKB 将该块转发给 LiteLLM，以便调优 LiteLLM 支持的任意项——
你设置，LiteLLM 使用。有两个键较特殊：

- `timeout` 与 `extra_headers` 按**每次请求**应用（每次调用都需要）。
- 其余键（`drop_params`、`num_retries`、`ssl_verify`、…）设为 `litellm`
  模块的进程级全局。

#### 慢速本地运行时（Ollama、LM Studio、llama.cpp）

本地推理可能很慢——在 Mac 上跑 **LM Studio** 时，单次编译调用可能要数分钟，
而**默认请求超时会中止它**（这是本地运行时失败的常见原因）。提高 `timeout`
（秒）。对拒绝仅 OpenAI 参数的后端（如 Ollama）加上 `drop_params`：

```yaml
model: ollama/llama3.1     # or your LM Studio / llama.cpp model id
language: en
litellm:
  drop_params: true
  timeout: 1200            # raise further (e.g. 3600) for large local models
```

#### GitHub Copilot / ChatGPT 订阅类提供商

这些需要额外请求头，并使用 OAuth（无 API 密钥）：

```yaml
model: github_copilot/gpt-4o
language: en
litellm:
  extra_headers:
    Editor-Version: vscode/1.95.0
    Copilot-Integration-Id: vscode-chat
```

#### OpenRouter 响应缓存

当 `model` 为 `openrouter/*` 时，可选择启用 OpenRouter 的
[Response Caching](https://openrouter.ai/docs/guides/features/response-caching)：
相同 payload 的请求约在 80–300 ms 内返回，且**不计 token 费用**。
这对编译重试路径（失败的 `add` 会用相同 prompt 重跑每个 summary/plan/concept
调用）以及反复的 `lint` / 开发迭代直接有利。通过 `extra_headers` 发送缓存头：

```yaml
model: openrouter/anthropic/claude-sonnet-4.5
language: en
extra_headers:                  # top-level, or nested under `litellm:` — both work
  X-OpenRouter-Cache: "true"
  X-OpenRouter-Cache-TTL: "600" # optional, 1–86400s (OpenRouter default 300)
```

设计上为可选：响应会存在 OpenRouter，因此零数据留存 / 受监管内容请保持关闭。
仅 `openrouter/*` 模型会读这些头；其他提供商会忽略。

---

## 3. API 密钥与提供商

设置一个通用密钥，OpenKB 会按 `model` 路由到正确的提供商。仓库自带的
[`.env.example`](../../.env.example)：

```bash
# OpenAI:    LLM_API_KEY=sk-...
# Anthropic: LLM_API_KEY=sk-ant-...
# Gemini:    LLM_API_KEY=AIza...
LLM_API_KEY=your-key-here
```

- **提供商自动检测：** `model: anthropic/claude-sonnet-4-6` → 你的
  `LLM_API_KEY` 会自动导出为 `ANTHROPIC_API_KEY`。
- **OAuth 提供商**（`chatgpt/*`、`github_copilot/*`）**不需要**密钥 — OpenKB
  不会因缺少密钥而警告。
- **PageIndex Cloud** 使用单独的 `PAGEINDEX_API_KEY`（见
  [`pageindex-cloud/`](../pageindex-cloud/)）。
- **Amazon Bedrock**（`model: bedrock/...`）用 AWS 凭证认证，而非
  `LLM_API_KEY`。把它们放进 `<kb>/.env`（LiteLLM/boto3 从环境读取）；
  不需要 `LLM_API_KEY`：

  ```bash
  # <kb>/.env
  AWS_ACCESS_KEY_ID=...
  AWS_SECRET_ACCESS_KEY=...
  AWS_REGION_NAME=eu-central-1
  ```

  ```yaml
  # <kb>/.openkb/config.yaml
  model: bedrock/eu.anthropic.claude-sonnet-4-6
  parallel_tool_calls: null   # REQUIRED for Bedrock Claude: sending
                              # parallel_tool_calls at all (any value) makes
                              # LiteLLM send a malformed tool_choice that Bedrock
                              # rejects (#175). null tells OpenKB to omit it.
                              # Write it as bare `null` — not `None` or "null".
  ```

**密钥读取顺序**（先匹配先用，已有环境变量始终优先）：

1. shell 环境
2. `<kb>/.env`
3. `~/.config/openkb/.env`（跨所有 KB 共享的全局密钥）

---

## 4. 「当前 KB」在哪里？

多数命令需要知道作用在哪个 KB。解析顺序：

1. `--kb-dir /path/to/kb`（或 `OPENKB_DIR=/path/to/kb`）— 显式覆盖。
2. 从当前目录向上查找 `.openkb/` 文件夹。
3. 由 `openkb use <path>` 注册的全局默认（存在
   `~/.config/openkb/global.yaml`）。

```bash
# 从任意位置对指定 KB 跑查询
openkb --kb-dir ~/research-kb query "what changed in v2?"

# 把某个 KB 设为默认，之后不用再管路径
openkb use ~/research-kb
openkb status        # 现在从任意目录都会解析到 ~/research-kb
```

---

下一步：[`commands/`](../commands/) — 日常的摄入与查询循环。
