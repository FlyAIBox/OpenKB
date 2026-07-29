# AGENTS.md — OpenKB 面向编码 Agent 的地图

OpenKB 用 LLM 把原始文档编译成相互链接的 wiki 知识库
（通过 PageIndex 做无向量检索）。本仓库以 **agent-first** 方式开发：
人负责掌舵，Agent 负责执行。改动应优先保证对 Agent 可读。

## 接着读
- `docs/golden-principles.md` — 需遵守的机械规则（能强制执行的会强制）。
- `docs/internal/superpowers/{specs,plans}/` — 设计历史与计划 *（维护者本地，不进 git）*。
- `README.md` / `README.zh-CN.md` — 面向用户的概览与命令。

## 开发命令
- 安装：`pip install -e ".[dev]"`（或 `uv sync --extra dev` — 纯 `uv sync` 会跳过开发工具）
- 跑 CLI：`openkb <command>`（入口：`openkb.cli:cli`）
- 测试：`pytest`
- 检查/格式化/类型：`ruff check .` · `ruff format .` · `mypy openkb`

## 模块地图（openkb/）
- `cli.py` — Click CLI 入口与命令接线 *（较大；见 tech-debt）*。
- `config.py` — 配置加载/校验（LiteLLM 透传、环境变量）。
- `converter.py` — 文档 → markdown 转换（markitdown）。
- `url_ingest.py` — 抓取并摄入 URL（trafilatura）。
- `images.py` — 插图/图片提取与处理。
- `indexer.py` — 长文档的 PageIndex 树索引。
- `mutation.py` — 崩溃安全、串行的 KB 变更。
- `locks.py` — 原子写入 / 文件锁（`atomic_write_text`、portalocker）。
- `state.py` — 运行/会话状态跟踪。
- `frontmatter.py` — YAML frontmatter 往返（OKF）。
- `schema.py` — 页面/内容 schema 常量与辅助函数。
- `lint.py` — 结构性 wiki lint（断链、孤儿页、索引同步）。
- `tree_renderer.py`、`visualize.py`、`watcher.py` — 渲染 / 图谱 / 文件监视。
- `agent/compiler.py` — LLM wiki 编译器 *（较大；见 tech-debt）*。
- `agent/linter.py` — 语义（LLM）wiki lint（矛盾、缺口、过时）。
- `agent/chat.py`、`agent/chat_session.py` — 基于 wiki 的对话 *（chat.py 较大）*。
- `agent/query.py` — 一次性查询生成器。
- `agent/tools.py` — 供 query/linter 使用的共享 wiki 读写工具函数（chat 经 `query.build_chat_agent` 间接使用）。
- `agent/skills.py`、`agent/skill_runner.py`、`skill/` — Skill Factory。
- `deck/`、`templates/`、`prompts/` — deck 输出、模板、prompt 资源。

## 硬性不变量
- 依赖 **精确钉死**（供应链谨慎）。升级前先 vet。
- Wiki 写入必须走 `locks.py` / `mutation.py`（禁止临时写法）。
- 模块保持 < 800 行（`tests/test_file_size.py`）；已豁免文件见 tech-debt。
- 本文件保持短地图 — 细节放进 `docs/`。
