# PageIndex Cloud 工作流

OpenKB 借助 [PageIndex](https://github.com/VectifyAI/PageIndex) 基于树状、无需向量的检索来扩展长文档。本指南涵盖长文档在 OpenKB 中的三种流转方式：**本地**索引、**云端** OCR/索引，以及**导入**已在 PageIndex Cloud 中索引的文档。

> **试用文档：** [`../docs/deepseek-r1.pdf`](../docs/deepseek-r1.pdf)（约 22 页，略高于阈值 → 走长文档路径），或
> [`../docs/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf`](../docs/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf)
>（700+ 页 → 会触发云端 OCR 与分页窗口）。

---

## 短文档 vs. 长文档：阈值

当你 `openkb add` 一个 PDF 时，页数决定走哪条路径：

| 页数 | 路径 | 引擎 |
| --- | --- | --- |
| `< pageindex_threshold`（默认 20） | short-doc | markitdown → LLM 读取全文 |
| `≥ pageindex_threshold` | long-doc | PageIndex 树状索引 |

对于 long-doc 路径，**本地**还是**云端**运行仅取决于一个环境变量：

| `PAGEINDEX_API_KEY` | 长文档引擎 |
| --- | --- |
| 未设置 | 本地（pymupdf 文本 + 图片提取） |
| 已设置 | PageIndex Cloud OCR（Markdown + 图表），云端调用失败时回退到本地 |

```bash
# Local long-doc indexing — no key, no network
openkb add ../docs/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf

# Cloud OCR for the same long PDF — just set the key first
export PAGEINDEX_API_KEY="pi-..."
openkb add ../docs/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf
```

无论哪种方式，产物相同（`wiki/sources/<doc>.json` +
`wiki/summaries/<doc>.md` + 概念/实体页面），且文档在 `openkb list` 中显示为类型 `pageindex`。

---

## 导入已在云端索引的文档

若文档**已在 PageIndex Cloud 中索引**，你完全不需要本地 PDF —— 用 `doc_id` 导入即可：

```bash
export PAGEINDEX_API_KEY="pi-..."
openkb add --from-pageindex-cloud <DOC_ID>
```

这会从云端拉取文档树（结构 + 描述）和 OCR 后的页面内容，然后在本地编译概念与实体 —— 与本地长 PDF 完全一致，但磁盘上无需文件。注册类型为 `pageindex_cloud`。

它**不会**做的事：

- 从不修改云端语料 —— 导入是只读的。
- **幂等** —— 重复导入同一 `doc_id` 会被跳过。
- 对导入文档执行 `openkb remove` 仅清理你**本地**的 Wiki 产物；PageIndex Cloud 中的文档不受影响。

> **分页窗口：** PageIndex 单次页面内容请求上限为 1000 页。OpenKB 按 1000 页窗口拉取，当某窗口返回不足 1000 页时停止，因此任意长度文档（包括 700+ 页书籍）都能完整导入。

### 查找 `doc_id`

用 PageIndex 客户端列出云端集合中的内容：

```python
import os
from pageindex import PageIndexClient

client = PageIndexClient(api_key=os.environ["PAGEINDEX_API_KEY"])
col = client.collection()

for doc in col.list_documents():
    print(doc["doc_id"], "—", doc.get("doc_name"))
```

```text
pi-cmn3k8...  — attention-is-all-you-need.pdf
pi-x7f0aa...  — deepseek-r1.pdf
```

然后：

```bash
openkb add --from-pageindex-cloud pi-cmn3k8...
```

---

## 一次完整导入

```bash
$ export PAGEINDEX_API_KEY="pi-..."
$ openkb add --from-pageindex-cloud pi-cmn3k8...
Importing from PageIndex Cloud: pi-cmn3k8...
  Fetching structure + OCR pages...
  Compiling concepts and entities...
  [OK] attention-is-all-you-need imported from PageIndex Cloud.

$ openkb add --from-pageindex-cloud pi-cmn3k8...
  [SKIP] Already imported from PageIndex Cloud: pi-cmn3k8...
```

它写入注册表（`.openkb/hashes.json`）的条目 —— 注意没有 `raw_path`，因为没有本地文件：

```json
{
  "name": "attention-is-all-you-need.pdf",
  "doc_name": "attention-is-all-you-need-abc12345",
  "type": "pageindex_cloud",
  "path": "pageindex-cloud:pi-cmn3k8...",
  "source_path": "wiki/sources/attention-is-all-you-need-abc12345.json",
  "doc_id": "pi-cmn3k8..."
}
```

导入后，该文档与任何其他文档行为一致 —— `query`、`chat`、`recompile`、`visualize` 和 `skill new` 都能识别它。
