# OpenKB 产品与架构全景

> 面向知识库产品经理和软件架构师，基于当前仓库实现整理。

## 1. 产品定位

OpenKB 是一个本地优先、CLI 优先的 LLM 知识库系统。它将原始文档一次性
“编译”为结构化、相互链接的 Wiki，再让查询、对话、技能、知识图谱和演示文稿
共享这一知识底座。

它与传统 RAG 的主要区别是：

- 传统 RAG 通常在每次查询时重新召回文档片段。
- OpenKB 在摄取阶段生成摘要、概念、实体和交叉链接，将推理结果持久化。
- 新文档会更新已有概念和实体，使知识跨文档持续积累。
- 长文档通过 PageIndex 树状索引进行推理式检索，不依赖向量数据库。

一句话概括：

> 将分散文档编译为可维护的知识资产，再用同一知识底座持续生成工作成果。

## 2. 目标用户与适用场景

主要用户包括：

- 需要整理论文、报告和网页资料的研究者与知识工作者。
- 建设产品、技术或行业情报库的产品经理。
- 需要本地知识底座的开发者和智能体用户。
- 使用 Obsidian、Git、Claude Code、Codex 或 Gemini CLI 的用户。
- 通过 REST API 或 Web 工作台集成知识能力的小型团队。

适合的场景：

- 研究资料库和专业文献库。
- 产品、技术和竞争情报库。
- 项目知识沉淀与团队 Wiki。
- 个人或小团队的第二大脑。
- 需要跨多份文档综合分析的专业知识库。

当前不宜直接作为以下系统的替代品：

- 超大规模、多租户企业知识平台。
- 强事务数据库或复杂权限系统。
- 跨主机多写、高可用的分布式知识存储。
- 对文档摄取时延非常敏感的实时处理系统。

## 3. 两层产品能力

### 3.1 Wiki 基础层

负责知识的编译与维护：

- 初始化知识库。
- 摄取文件、目录、URL 和 PageIndex Cloud 文档。
- 将 Office、PDF、Markdown、HTML、CSV 等格式标准化。
- 对长 PDF 建立 PageIndex 树状索引。
- 提取文本、表格、图表和图像。
- 生成文档摘要、跨文档概念和命名实体。
- 建立 Wiki 双向链接并维护总索引。
- 监听新文件、删除来源、重新编译。
- 执行结构和语义健康检查。

### 3.2 生成器层

负责将 Wiki 转化为用户成果：

- `openkb query`：生成带依据的一次性回答。
- `openkb chat`：开展可恢复的多轮对话。
- `openkb visualize`：生成交互式知识图谱。
- `openkb skill new`：从知识库提炼智能体技能。
- `openkb deck new`：生成单文件 HTML 演示文稿。
- Web 工作台：浏览知识、上传文档、查询、对话和维护知识库。

## 4. 端到端业务流程

```text
初始化知识库
    ↓
采集文件、目录、URL 或云端文档
    ↓
计算 SHA-256、识别重复内容
    ↓
短文档 / 长 PDF 分流
    ├── 短文档：markitdown → Markdown
    └── 长 PDF：PageIndex → 文档树 + 页级内容 + 图像
    ↓
LLM 生成来源摘要
    ↓
读取已有概念和实体
    ↓
规划 create / update / related
    ↓
并发生成或改写概念页、实体页
    ↓
过滤无效 Wiki 链接、建立双向回链
    ↓
更新 index.md、log.md 和哈希注册表
    ↓
原子提交 Wiki 变更
    ↓
查询、对话、技能、图谱和演示文稿
    ↓
lint、watch、remove、recompile 持续治理
```

### 4.1 短文档路径

1. 使用 markitdown 转换为 Markdown。
2. LLM 读取完整文本。
3. 生成初始摘要。
4. 规划并生成概念和实体。
5. 根据实际存在的页面重写摘要中的 Wiki 链接。
6. 更新索引和双向链接。

### 4.2 长 PDF 路径

1. 达到 `pageindex_threshold` 时进入 PageIndex。
2. PageIndex 生成分层文档树和文档描述。
3. 页级内容保存为 JSON，图像保存到 `wiki/sources/images/`。
4. 编译器根据树摘要生成或更新概念、实体。
5. 查询时先读树，再精确获取所需页段，不拉取整本 PDF。

### 4.3 查询路径

查询智能体遵循逐层深入策略：

1. 读取 `wiki/index.md`，定位相关文档、概念和实体。
2. 读取摘要页了解文档全貌。
3. 读取概念页进行跨文档综合。
4. 对人物、组织、地点、产品等优先读取实体页。
5. 需要细节时读取短文档全文，或精确获取长文档页段。
6. 问题涉及图表或图片时读取原始图像。
7. 基于 Wiki 内容生成回答。

## 5. 系统逻辑架构

```text
┌─────────────────────────────────────────────────────────────┐
│ 交互层：CLI · REST API · SSE · Web Knowledge Workbench     │
├─────────────────────────────────────────────────────────────┤
│ 应用层：add/remove/recompile · query/chat · lint/watch      │
│         skill/deck/visualize                                │
├─────────────────────────────────────────────────────────────┤
│ 智能体层：compiler · query agent · chat agent · LLM linter  │
├─────────────────────────────────────────────────────────────┤
│ 文档层：markitdown · trafilatura · PyMuPDF · PageIndex      │
├─────────────────────────────────────────────────────────────┤
│ 知识层：sources · summaries · concepts · entities · index   │
├─────────────────────────────────────────────────────────────┤
│ 可靠性层：locks · mutation journal · atomic write · registry│
├─────────────────────────────────────────────────────────────┤
│ 外部能力：LiteLLM → OpenAI / Claude / Gemini / 本地模型     │
│           可选 PageIndex Cloud                              │
└─────────────────────────────────────────────────────────────┘
```

系统记录是文件型 Wiki，而不是向量数据库。上层智能体通过受限工具读取或写入，
下层通过文件锁、原子替换和事务日志保障一致性。

## 6. 核心组件

### 6.1 入口与应用编排

- `openkb/cli.py`：Click CLI、命令装配和主要业务入口。
- `openkb/api.py`：FastAPI 应用及核心 REST 接口。
- `openkb/api_*_router.py`：知识库、页面、文档、图谱、配置和输出路由。
- `openkb/watch_service.py`：Web/API 场景下的文件监听服务。

### 6.2 内容采集与转换

- `openkb/converter.py`：格式识别、markitdown 转换、长短文档分流。
- `openkb/url_ingest.py`：URL 下载、内容类型检测和网页正文提取。
- `openkb/images.py`：PDF 图像与页面内容提取。
- `openkb/state.py`：以 SHA-256 为主键的文档注册表。

### 6.3 长文档索引

- `openkb/indexer.py`：PageIndex 本地/云端索引及分页内容持久化。
- `openkb/tree_renderer.py`：将 PageIndex 树渲染成可供 LLM 导航的摘要。

### 6.4 Wiki 编译器

- `openkb/agent/compiler.py`：摘要、概念、实体、回链和索引编译。
- `openkb/schema.py`：Wiki 目录、页面类型和 OKF 约定。
- `openkb/frontmatter.py`：YAML frontmatter 解析与往返写入。

编译器的关键控制措施包括：

- 对 LLM JSON 输出进行解析和修复。
- 校验概念及实体数据形状。
- 对实体类型执行配置白名单约束。
- 通过已知目标白名单过滤幽灵 Wiki 链接。
- 使用信号量限制概念和实体生成并发。
- 由代码维护 frontmatter、回链和总索引。

### 6.5 查询与对话

- `openkb/agent/query.py`：查询智能体、检索策略和流式事件。
- `openkb/agent/chat.py`：交互式多轮对话及斜杠命令。
- `openkb/agent/chat_session.py`：会话持久化与恢复。
- `openkb/agent/tools.py`：Wiki 文件、页段、图像和受限写入工具。

查询智能体默认只读。对话智能体的写入范围限制在：

- `wiki/explorations/**`
- `output/**`

它不能直接修改 `sources/`、概念页、实体页、配置或凭据。

### 6.6 下游生成器

- `openkb/skill/`：技能创建、验证、评估、版本和回滚。
- `openkb/deck/`：HTML 演示文稿创建与校验。
- `openkb/visualize.py`：知识图谱生成。
- `openkb/agent/skills.py`：发现本地安装的 Agent Skills。

### 6.7 治理组件

- `openkb/lint.py`：断链、孤儿页、索引同步和 frontmatter 检查。
- `openkb/agent/linter.py`：矛盾、缺口、过时和冗余等语义检查。
- `openkb/page_ops.py`：页面删除和引用清理。
- `openkb/log.py`：知识库操作日志。

### 6.8 一致性与崩溃恢复

- `openkb/locks.py`：进程内读写锁、跨进程 advisory 文件锁和原子写。
- `openkb/mutation.py`：路径快照、事务日志、回滚和崩溃恢复。
- `openkb/add_coordinator.py`：文档摄取事务协调。

突变流程：

```text
获取 KB 独占锁
    ↓
自动恢复上次中断的 active journal
    ↓
为受影响路径创建快照
    ↓
写入 active journal
    ↓
执行编译和文件变更
    ↓
写入 committed 状态
    ↓
清理快照和 journal
```

如果执行失败，系统按路径深度逆序恢复。如果回滚本身失败，会保留事务日志并停止
后续批量提交，避免新的提交被下一次恢复覆盖。

## 7. 知识库目录模型

```text
<knowledge-base>/
├── raw/                       # 原始输入副本
├── wiki/
│   ├── sources/               # 短文档 Markdown / 长文档分页 JSON
│   │   └── images/            # 提取的图像
│   ├── summaries/             # 每个来源一份摘要
│   ├── concepts/              # 跨来源概念综合
│   ├── entities/              # 人物、组织、地点、产品等
│   ├── explorations/          # 保存的查询和对话结果
│   ├── reports/               # Lint 报告
│   ├── AGENTS.md              # Wiki Schema 与智能体操作约定
│   ├── index.md               # 全库内容目录
│   └── log.md                 # 操作日志
├── .openkb/
│   ├── config.yaml            # KB 配置
│   ├── hashes.json            # 内容哈希注册表
│   ├── chats/                 # 对话会话
│   ├── files/                 # PageIndex 数据
│   ├── journal/               # 突变事务日志
│   ├── staging/               # 暂存和回滚快照
│   └── ingest.lock            # KB 锁
├── skills/                    # KB 本地技能
└── output/                    # 图谱、技能、幻灯片等输出
```

## 8. 配置与模型架构

配置优先级：

```text
内置默认值
    ↓
~/.config/openkb/global.yaml
    ↓
<kb>/.openkb/config.yaml
```

主要配置包括：

- `model`：LiteLLM `provider/model`。
- `language`：Wiki 和回答语言。
- `pageindex_threshold`：长 PDF 分流阈值，默认 20 页。
- `entity_types`：可生成的实体类型。
- `concurrency`：摄取期间的 LLM 并发上限。
- `timeout`、`extra_headers`、`parallel_tool_calls`：模型调用适配参数。

REST 多知识库场景使用按请求解析的 `LlmCredentialBundle`，避免不同知识库的 API
密钥、请求头和超时配置互相串扰。

## 9. 关键架构优势

### 9.1 知识透明

Markdown 和 JSON 可直接查看、审计、版本化和迁移；也可在 Obsidian 中使用。

### 9.2 知识可复用

摄取阶段产生的摘要、概念和实体可被后续所有查询和生成器复用。

### 9.3 长文档上下文友好

通过树状索引导航长 PDF，避免将整份文档塞入模型上下文。

### 9.4 模型提供商解耦

LiteLLM 统一接入 OpenAI、Claude、Gemini 和兼容接口的本地模型。

### 9.5 本地可靠性

读写锁、原子写、路径快照和事务日志降低中断导致知识库损坏的风险。

### 9.6 智能体权限受限

查询智能体只读；对话和生成器仅能写入探索记录和输出目录。

## 10. 当前边界与风险

### 10.1 产品成熟度

项目元数据标记为 Alpha，更适合本地使用、团队试点和技术验证。

### 10.2 扩展规模

当前以文件系统为存储核心。数据库存储、超大型知识库的分层主题索引及大型嵌套
文档集合仍属于后续演进方向。

### 10.3 长文档类型

PageIndex 长文档主路径目前主要面向 PDF，非 PDF 长文档尚未获得同等能力。

### 10.4 一致性范围

`portalocker` 提供的是本地文件系统 advisory 锁，不保证跨主机、NFS 或云同步目录
中的一致协调。

### 10.5 LLM 成本与非确定性

一次摄取可能包含摘要、规划、概念、实体和摘要重写等多轮调用。它降低了后续重复
推理成本，但增加了摄取时延、模型费用和输出非确定性。

### 10.6 人工编辑治理

概念和实体会随新来源持续改写，`recompile` 也可能覆盖人工修改。实际使用中应配合：

- Git 版本控制。
- `--dry-run` 预览。
- 自定义 `wiki/AGENTS.md`。
- 人工审核关键知识页面。

### 10.7 Web 安全

Web 服务默认面向本地且不启用认证。对外暴露前必须设置
`OPENKB_API_TOKEN`，企业部署还需要补充 TLS、权限、审计和网络隔离。

## 11. 架构演进建议

若要从本地知识工作台演进为企业知识平台，建议依次考虑：

1. 将 CLI 中的业务逻辑拆分为独立应用服务，减少 CLI 与 API 耦合。
2. 拆分体量较大的 `cli.py`、`agent/compiler.py` 和 `agent/chat.py`。
3. 引入后台任务队列，显式管理摄取进度、重试、取消和配额。
4. 增加人工审核、变更差异和知识页面版本工作流。
5. 抽象存储接口，引入数据库元数据和对象存储。
6. 增加租户、角色、文档级权限和审计日志。
7. 建立模型成本、摄取时延、失败率和知识健康度可观测性。
8. 在多节点部署前重新设计分布式锁和事务边界。

## 12. 典型使用闭环

```bash
# 初始化
mkdir my-kb && cd my-kb
openkb init

# 摄取资料
openkb add paper.pdf
openkb add ~/papers/
openkb add https://example.com/article

# 使用知识
openkb query "这些资料的主要结论是什么？"
openkb chat

# 生成成果
openkb visualize
openkb skill new domain-expert "像领域专家一样分析问题"
openkb deck new introduction "生成主题入门介绍"

# 持续维护
openkb lint
openkb watch
openkb recompile --all
```

## 13. 关键源码索引

- 产品概述：`README.zh-CN.md`
- 模块地图：`AGENTS.md`
- Wiki Schema：`openkb/schema.py`
- 文档转换：`openkb/converter.py`
- PageIndex：`openkb/indexer.py`
- 知识编译：`openkb/agent/compiler.py`
- 查询策略：`openkb/agent/query.py`
- Agent 工具：`openkb/agent/tools.py`
- 对话：`openkb/agent/chat.py`
- 配置：`openkb/config.py`
- 状态注册表：`openkb/state.py`
- 锁与原子写：`openkb/locks.py`
- 突变事务：`openkb/mutation.py`
- REST API：`openkb/api.py`
- 结构健康检查：`openkb/lint.py`
- 语义健康检查：`openkb/agent/linter.py`

