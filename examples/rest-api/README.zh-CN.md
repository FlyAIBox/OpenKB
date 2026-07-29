# REST API 与 Knowledge Workbench

本指南涵盖 OpenKB REST API（FastAPI）及内置的 Knowledge Workbench Web UI。

> 服务器启动后，交互式 API 参考文档可在 [`/docs`](http://127.0.0.1:7566/docs)（OpenAPI/Swagger）实时访问 — 可直接将 `/openapi.json` 导入 Postman。

## Knowledge Workbench（Web UI）

OpenKB 内置一个 Web 单页应用 — **Knowledge Workbench** — 由 REST 服务器在 `/` 直接提供，无需单独的前端进程即可获得完整浏览器界面。构建后的 UI 打包在 `openkb` wheel 中，正常安装即已包含：

```bash
# 1. 安装 API extra（构建后的 UI 随包分发）
pip install "openkb[web]"

# 2. 启动服务器 — 本地使用无需配置
openkb-web --host 127.0.0.1 --port 7566   # 在 http://127.0.0.1:7566/ 提供 API + Workbench
```

可选环境变量：

- `OPENKB_KB_ROOT` — REST 创建的知识库存储位置（默认 `~/.config/openkb/kbs`）。
- `OPENKB_API_TOKEN` — 设置后要求 bearer 认证（见 [Authentication](#authentication-and-common-behavior)）；不设置则本地开放使用。

> **从源码 checkout？** 构建产物（`openkb/web/`）被 git 忽略，因此 editable 安装（`pip install -e ".[web]"`）在构建一次前没有 UI：`cd frontend && npm install && npm run build`（输出到 `openkb/web/`）。或用 Vite 开发服务器 `npm run dev`（将 `/api` 代理到运行中的 `openkb-web`）。无 bundle 时，`openkb-web` 仅在 `/api/v1` 提供 REST API，`/` 返回 404。

在浏览器中打开 `http://127.0.0.1:7566/`。未设置 `OPENKB_API_TOKEN` 时会立即连接本地 API — 无需提示。（若配置了 token，**Connection** 对话框会要求输入一次并在浏览器中缓存；也可手动打开以将 UI 指向远程 API base。）Workbench 随后提供：

- **Overview** — index/concept/summary/report 统计卡片、可点击 concept chips、最近文档，以及 last-compile/lint 活动。
- **Documents** — 拖放多文件上传，带 per-file SSE 进度、hash 表及确认后删除。
- **Query** — 流式回答，右侧面板 timeline 实时显示 `tool_call` 推理；GFM Markdown 渲染（粗体、表格、代码等）。
- **Chat** — 多轮流式对话及持久化 session 列表：加载历史、恢复 session、删除 session。
- **Maintenance** — lint（可选 auto-fix）、recompile（全部或单文档，SSE 日志），以及 file-watcher 开关。

右侧 **Inspector** timeline 显示每次流式操作的 vectorless retrieval 与推理步骤。从 Workbench 创建新 KB 会继承项目根目录的 `config.yaml` 与 LLM 凭据（`.env`），开箱即可运行 query。UI 响应式 — 窄屏下三栏折叠为单列并带汉堡导航。

## REST API

OpenKB 还提供 FastAPI 服务，供 Postman、前端或其他 HTTP 客户端使用知识库。

如需要，安装 API 依赖：

```bash
pip install -e ".[web]"
```

启动 API 服务器：

```powershell
$env:OPENKB_API_TOKEN="test-token"
$env:OPENKB_KB_ROOT="D:\project\OpenKB\kbs"
.\.venv\Scripts\python.exe -m openkb.api --host 127.0.0.1 --port 7566
```

### Authentication and common behavior

认证为**可选**，由服务器环境变量 `OPENKB_API_TOKEN` 控制：

- **未设置（默认）** — API 无认证。这是 local-first 默认行为，`openkb-web` 与 Workbench 无需配置即可使用。
- **已设置** — 每个请求须携带 token，Workbench 会提示一次（缓存在浏览器中）：

  ```text
  Authorization: Bearer <OPENKB_API_TOKEN>
  ```

  缺失或错误的 token 以 `401` 拒绝。

> **对外暴露服务器？** 绑定到非 loopback 主机（如 `--host 0.0.0.0`）时务必设置 `OPENKB_API_TOKEN` — 否则 API 及其可访问的每个 KB 对全世界开放。此种情况下 `openkb-web` 会打印警告。

`OPENKB_KB_ROOT` 可选，控制 REST 创建的知识库存储位置。未设置时 OpenKB 使用 `~/.config/openkb/kbs`。

REST 客户端用 `kb` 标识知识库，而非文件系统路径。例如 `postman-kb` 解析为 `$OPENKB_KB_ROOT/postman-kb`。

各端点通用状态码：

- `200` — 成功。
- `400` — 无效请求体、未知 `kb`，或非 OpenKB 目录的 KB（缺少 `.openkb/` 或 `wiki/`）。
- `401` — 缺失或错误的 bearer token（仅当设置了 `OPENKB_API_TOKEN`）。
- `404` — 引用的 document/watcher 未找到（remove/recompile/watch-stop）。
- `409` — 标识符歧义（remove/recompile），`detail` 中含 `candidates`。
- `500` — 服务器错误。

所有 JSON 端点使用 `Content-Type: application/json`。`/api/v1/add` 是唯一使用 `multipart/form-data` 的端点。

### Streaming (SSE)

接受 `"stream": true` 的端点（`query`、`chat`、`remove`、`recompile`），以及 `add`（通过 `stream` 表单字段）和 `watch/events`（始终流式），返回 Server-Sent Events（`Content-Type: text/event-stream`）。每帧格式为：

```text
event: <name>
data: <json-object>
```

SSE 事件名：

- `start` — 流已打开；`data` 含 `endpoint`。
- `delta` — 增量回答文本（query/chat），`{"text": "..."}`。
- `tool_call` — agent 工具调用（query/chat），含调用详情。
- `uploaded` / `file_start` / `file_done` — 单文件进度（add）。
- `plan` — 执行计划（remove：完整计划；recompile：目标列表）。
- `progress` — 阶段进度（remove：`wiki_cleanup`）。
- `doc` — 单文档 recompile 完成（recompile stream）。
- `final` — 终端成功 payload（与非流式 JSON 体一致）。
- `error` — 失败，`{"message": "..."}`（remove 可能带 `code`）。
- `done` — 流关闭；始终最后发出。

### Endpoints

所有端点位于 `/api/v1` 下。

| Method | Path                    | Body         | Streams  | Purpose                                |
| ------ | ----------------------- | ------------ | -------- | -------------------------------------- |
| GET    | `/kbs`                  | —            | no       | 列出 KB root 下的知识库                |
| POST   | `/init`                 | JSON         | no       | 创建知识库                             |
| POST   | `/add`                  | multipart    | optional | 上传并编译文档                         |
| POST   | `/query`                | JSON         | yes      | 一次性提问                             |
| POST   | `/chat`                 | JSON         | yes      | 多轮 chat session                      |
| POST   | `/chat/sessions`        | JSON         | no       | 列出持久化 chat sessions               |
| POST   | `/chat/sessions/load`   | JSON         | no       | 加载 session 历史                      |
| POST   | `/chat/sessions/delete` | JSON         | no       | 删除 session                           |
| POST   | `/list`                 | JSON         | no       | 列出 documents、summaries、concepts    |
| POST   | `/status`               | JSON         | no       | KB 目录/index 统计                     |
| POST   | `/lint`                 | JSON         | no       | 结构 + 语义 lint 报告                  |
| POST   | `/remove`               | JSON         | yes      | 删除文档并清理                         |
| POST   | `/recompile`            | JSON         | yes      | 重编译一个或全部文档                   |
| POST   | `/watch/start`          | JSON         | no       | 启动文件系统 watcher                   |
| POST   | `/watch/stop`           | JSON         | no       | 停止 watcher                           |
| POST   | `/watch/status`         | JSON         | no       | Watcher 状态与最近事件                 |
| GET    | `/watch/events`         | query params | always   | Watcher 事件的 SSE 流                  |

#### Initialize a KB

```http
POST /api/v1/init
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`InitRequest`）：

| Field             | Type   | Required | Default | Notes                         |
| ----------------- | ------ | -------- | ------- | ----------------------------- |
| `kb`              | string | yes      | —       | 新 KB 名称                    |
| `model`           | string | no       | `null`  | LLM model 覆盖                |
| `api_key`         | string | no       | `null`  | 写入 KB 本地 `.env`           |
| `openai_api_base` | string | no       | `null`  | OpenAI 兼容 gateway URL       |

`api_key` 与 `openai_api_base` 在创建 KB 时写入 KB 本地 `.env`；响应中不回显 secret 值。

当 `model`、`api_key`、`openai_api_base` 均未提供（Workbench 默认）时，新 KB 继承操作者项目根目录的 `config.yaml` 以及服务器工作目录 `.env` 中的 LLM 凭据（过滤掉服务器级 `OPENKB_*` 变量），因此从 UI 创建的 KB 开箱即可 query。

Response（`InitResponse`，`200`）：`kb`（string）、`created`（bool，KB 已存在时为 `false`）、`env_written`（`{api_key: bool, openai_api_base: bool}`）、`message`（string）。错误：`400` 无效/已存在 KB，`500` 其他。

#### Add Documents

```http
POST /api/v1/add
Authorization: Bearer test-token
```

`multipart/form-data`：

| Field    | Type | Value                 |
| -------- | ---- | --------------------- |
| `kb`     | Text | `postman-kb`          |
| `stream` | Text | `true` 或 `false`     |
| `files`  | File | 一个或多个文档        |

支持的类型与 CLI 一致：`.pdf`、`.md`、`.markdown`、`.docx`、`.pptx`、`.xlsx`、`.xls`、`.html`、`.htm`、`.txt`、`.csv`。`stream: true` 发出 per-file SSE 事件（`uploaded`、`file_start`、`file_done`、`final`）；`stream: false` 返回单一 JSON 体。未上传文件时 `400`。

Response（`AddResponse`，`200`）：`kb`、`files`（每项 `{original_name, saved_path, status, message}`）、`added_count`、`skipped_count`（已 indexed）、`failed_count`。

#### Query

```http
POST /api/v1/query
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`QueryRequest`）：

| Field      | Type   | Required | Default | Notes                                |
| ---------- | ------ | -------- | ------- | ------------------------------------ |
| `kb`       | string | yes      | —       |                                      |
| `question` | string | yes      | —       |                                      |
| `stream`   | bool   | no       | `true`  | SSE 与单一 JSON                      |
| `save`     | bool   | no       | `false` | 将回答写入 `wiki/explorations/`      |

非流式 Response（`QueryResponse`，`200`）：`answer`（string）、`saved_path`（string\|null，`save: true` 时设置）。Stream 事件：`start`、`delta`、`tool_call`、`final`（`{answer, saved_path}`）、`error`、`done`。失败时 `500`。

#### Chat

```http
POST /api/v1/chat
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`ChatRequest`）：

| Field        | Type   | Required | Default | Notes                      |
| ------------ | ------ | -------- | ------- | -------------------------- |
| `kb`         | string | yes      | —       |                            |
| `message`    | string | yes      | —       |                            |
| `session_id` | string | no       | `null`  | 恢复已有 session           |
| `stream`     | bool   | no       | `true`  | SSE 与单一 JSON            |

非流式 Response（`ChatResponse`，`200`）：`session_id`（传回以继续）、`answer`、`turn_count`（session 内总轮数）。Stream 事件：`start`（含 `session_id`）、`delta`、`tool_call`、`final`、`error`、`done`。

#### Chat Sessions

列出、加载或删除 KB 的持久化多轮 sessions。这些为 Workbench 中的 session 侧栏提供数据。

```http
POST /api/v1/chat/sessions
Authorization: Bearer test-token
```

Request（`KbRequest`）：`{ "kb": "postman-kb" }`。
Response（`ChatSessionListResponse`）：`{ "kb", "sessions": [{ id, title, turn_count, updated_at, model }] }`，按 `updated_at` 降序。

```http
POST /api/v1/chat/sessions/load
Authorization: Bearer test-token
```

Request（`ChatSessionLoadRequest`）：`{ "kb", "session_id" }`。
Response（`ChatSessionLoadResponse`）：`{ session_id, title, turn_count, user_turns: [...], assistant_texts: [...] }`。
客户端交错 `user_turns` 与 `assistant_texts` 以渲染历史。session 不存在时 `404`。

```http
POST /api/v1/chat/sessions/delete
Authorization: Bearer test-token
```

Request（`ChatSessionDeleteRequest`）：`{ "kb", "session_id" }`。
Response（`ChatSessionDeleteResponse`）：`{ deleted: true }`。未找到时返回 `404`。

#### List

```http
POST /api/v1/list
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`KbRequest`）：`kb`。

Response（`ListResponse`，`200`）：`documents`（`[{hash, name, type, display_type, pages}]`）、`document_count`、`summaries`、`concepts`、`reports`（页面名列表）。

#### Status

```http
POST /api/v1/status
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`KbRequest`）：`kb`。

Response（`StatusResponse`，`200`）：`directories`（各文件夹文件数）、`raw_count`、`total_indexed`、`last_compile` 与 `last_lint`（ISO 时间戳或 `null`）。

#### Lint

```http
POST /api/v1/lint
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`LintRequest`）：

| Field | Type   | Required | Default | Notes                                      |
| ----- | ------ | -------- | ------- | ------------------------------------------ |
| `kb`  | string | yes      | —       |                                            |
| `fix` | bool   | no       | `false` | 先 rewrite/strip 损坏的 `[[wikilinks]]`    |

`fix: true` 时，在 KB ingest lock 下 rewrite/strip 损坏的 wikilinks（镜像 `openkb lint --fix`），再运行报告，因此报告反映 fix 后状态。语义 lint 为多轮 LLM agent 运行，无论 `fix` 与否响应都可能需数十秒到数分钟；`fix` 本身为本地毫秒级文件 rewrite。

Response（`LintResponse`，`200`）：

| Field                 | Type         | Notes                                       |
| --------------------- | ------------ | ------------------------------------------- |
| `skipped`             | bool         | 无 indexed 文档时为 `true`                  |
| `reason`              | string\|null | 如 `no_documents_indexed`                   |
| `message`             | string       | 状态；`fix: true` 时含 fix 摘要             |
| `structural_report`   | string\|null | 本地结构 lint markdown                      |
| `knowledge_report`    | string\|null | LLM 语义 lint markdown                      |
| `report_path`         | string\|null | `wiki/reports/` 下报告                      |
| `lint_files_changed`  | int\|null    | `fix` rewrite 的文件数（否则 `null`）       |
| `lint_ghosts_removed` | int\|null    | `fix` 剥离的 ghost links（否则 `null`）     |

#### Remove Documents

删除文档并清理其 wiki 页面、images、registry 与 PageIndex 状态，与 `openkb remove` 同一 pipeline。

```http
POST /api/v1/remove
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`RemoveRequest`）：

| Field        | Type   | Required | Default | Notes                                   |
| ------------ | ------ | -------- | ------- | --------------------------------------- |
| `kb`         | string | yes      | —       |                                         |
| `identifier`   | string | yes      | —       | 文件名、`doc_name` slug 或子串          |
| `keep_raw`   | bool   | no       | `false` | 保留源文件                              |
| `keep_empty` | bool   | no       | `false` | 保留现已空的 concept/entity 页面        |
| `dry_run`    | bool   | no       | `false` | 仅预览                                  |
| `stream`     | bool   | no       | `false` | SSE 与单一 JSON                         |

Response（`RemoveResponse`，`200`）：`status`（`removed`、`partial`、`dry_run`）、`name`、`doc_name`、`actions`（每项 `{tag, target}`）、`concepts_deleted`、`entities_deleted`、`lint_files_changed` 与 `lint_ghosts_removed`（删除后 scoped `lint --fix` 计数）、`pageindex_message`/`pageindex_error`、`message`、`candidates`。错误：`404` 无匹配，`409` 多匹配（含 `candidates`）。Stream 事件：`start`、`plan`、`progress`、`final`、`error`、`done`。

#### Recompile

重编译一个或全部文档，镜像 `openkb recompile`。

```http
POST /api/v1/recompile
Content-Type: application/json
Authorization: Bearer test-token
```

Request（`RecompileRequest`）：

| Field            | Type   | Required | Default | Notes                             |
| ---------------- | ------ | -------- | ------- | --------------------------------- |
| `kb`             | string | yes      | —       |                                   |
| `doc_name`       | string | no       | `null`  | 单文档；与 `all_docs` 二选一 omit |
| `all_docs`       | bool   | no       | `false` | 重编译全部文档                    |
| `dry_run`        | bool   | no       | `false` | 仅预览                            |
| `refresh_schema` | bool   | no       | `false` | 先 re-extract PageIndex schema    |
| `stream`         | bool   | no       | `false` | SSE 与单一 JSON                   |

Response（`RecompileResponse`，`200`）：`status`（`done`）、`total`、`recompiled`、`skipped`、`docs`（每项 `{name, doc_name, type, status, elapsed, message}`）、`targets`/`candidates`（计划/歧义时存在）。错误：`404` 无匹配，`409` 歧义（含 `candidates`），`500` 其他。Stream 事件：`start`、`plan`（`{targets}`）、`doc`（单文档结果）、`final`、`error`、`done`。

#### Watch（文件变更时自动编译）

启动/停止/检查文件系统 watcher，自动编译拖入 `raw/` 的文件（同 `openkb watch`），以及 watcher 事件的 SSE 流。

```http
POST /api/v1/watch/start
POST /api/v1/watch/stop
POST /api/v1/watch/status
GET  /api/v1/watch/events
Content-Type: application/json
Authorization: Bearer test-token
```

`watch/start`（`WatchStartRequest`）：`kb`、`debounce`（秒，默认 `2.0`，须 `> 0`）。`watch/stop` 与 `watch/status` 仅需 `kb`。

```json
{ "kb": "postman-kb", "debounce": 2.0 }
```

三者均返回 `WatchStatusResponse`：`kb`、`active`（bool）、`started_at`（epoch 或 `null`）、`raw_dir`、`debounce`、`counters`（`{added, updated, failed, ...}`）、`recent_events`（`[{ts, event, data}]`）。该 KB 无 active watcher 时 `watch/stop` 返回 `404`。

`GET /api/v1/watch/events` 始终为 SSE。Query params：`kb`（必填）、`max_events`（int，`>=1`，N 个事件后停止）、`timeout_seconds`（float，`>=0`，该秒数后停止）。Stream 事件：`start`、watcher 自身事件（如 `added`、`updated`、`failed`、`final`）、`error`、`done`。

完整 OpenAPI schema 见 [`/openapi.json`](http://127.0.0.1:7566/openapi.json) — 可导入 Postman 或任何 OpenAPI 兼容客户端。
