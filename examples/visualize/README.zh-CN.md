# Visualize

`openkb visualize` 将 Wiki 的 `[[wikilink]]` 图渲染为自包含的交互式 HTML 页面 —— 快速*看见*已导入文档之间的知识如何连接。

> **本文件夹中的真实产物：** [`graph.html`](graph.html) —— 由一篇论文生成的图（13 个节点，96 条边）。在浏览器中打开，在 **3D / mind-map / radial** 模式间切换；点击节点查看详情。

```bash
openkb visualize
```

```text
Graph written to /Users/you/my-kb/output/visualize/graph.html  (13 nodes, 96 edges)
# opens in your default browser
```

默认生成后会在浏览器中打开。无头环境（CI、远程机器）可跳过启动：

```bash
openkb visualize --no-open
```

> **前提：** 需要已编译的 Wiki。尚未导入任何内容时，会提示先运行 `openkb add`。本文件夹的 [`graph.html`](graph.html) 由 [`../commands/sample-wiki/`](../commands/sample-wiki/) 中的 Wiki 构建。

---

## 图中包含什么

- **节点** —— `wiki/summaries/`、`wiki/concepts/`、`wiki/entities/` 下的每个页面。携带标签、类型、描述、sources，以及入/出度（用于 sizing）。
- **边** —— 页面间有向的 `[[wikilink]]` 引用。自环丢弃，重复边合并。
- **类型** —— 来自页面 frontmatter 或所在目录，用于节点着色和图例筛选。

单个 HTML 文件内嵌图数据与全部渲染代码，因此可分享或提交，无外部依赖。

---

## 三种视图模式

在页面按钮间切换：

| 模式 | 布局 |
| --- | --- |
| **3D** *(default)* | 3D 力导向「星云」—— 轨道、缩放、拖拽固定节点。 |
| **mind-map** | 水平树：OpenKB → 文档（summaries）→ 概念。 |
| **radial** | OpenKB 在中心，文档在辐条上，概念向外辐射；缩放/平移。 |

页面还提供搜索（按标签筛选）、可切换的类型图例、间距滑块和节点检查器 —— 点击节点查看描述、sources 和链接。

---

## 在整体流程中的位置

`visualize` 只读且重跑成本低，与循环中其他命令配合良好：

```bash
openkb add ~/papers/        # ingest a batch
openkb lint --fix           # repair any dangling links so the graph is clean
openkb visualize            # see the shape of what you've built
```

稀疏图且孤立节点很多，通常表示文档尚未共享概念 —— 在同一领域添加更多文档，观察概念 hub 如何随知识复利而增长。
