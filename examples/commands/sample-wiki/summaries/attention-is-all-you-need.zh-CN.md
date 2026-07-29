---
type: "Summary"
description: "介绍 Transformer——一种完全基于注意力的序列翻译模型。"
doc_type: short
full_text: "sources/attention-is-all-you-need.md"
---

# Attention Is All You Need

## 摘要

本文提出 [[concepts/transformer-models|Transformer]]，一种全新的序列转换架构，用纯注意力计算取代循环层与卷积层。核心主张是：自注意力足以建模编码器-解码器任务中的依赖关系，同时实现更高程度的并行化与更快的训练。

## 核心思想

传统序列到序列系统依赖循环神经网络或卷积，并常与注意力机制结合。Transformer 完全去除循环结构，在编码器与解码器中均使用堆叠的 [[concepts/attention-mechanisms|注意力机制]]，并配合逐位置前馈网络。

该设计带来三大优势：

- 更高的训练并行度
- 长程依赖的更短路径
- 以更少计算获得更强的翻译质量

## 架构

Transformer 采用编码器-解码器结构：

- **编码器**：6 个相同层，每层包含
  - 多头自注意力
  - 逐位置前馈网络
  - 残差连接与层归一化
- **解码器**：6 个相同层，每层包含
  - 掩码自注意力
  - 编码器-解码器注意力
  - 逐位置前馈网络
  - 残差连接与层归一化

模型使用：

- 缩放点积注意力
- [[concepts/attention-mechanisms|多头注意力]]
- 可学习的词元嵌入
- 正弦 [[concepts/positional-encoding|位置编码]]

## 注意力机制

注意力将查询与一组键-值对映射为输出向量，该向量由值的加权和计算得到。论文定义了：

- **缩放点积注意力**：注意力分数除以键维度的平方根进行缩放，以避免点积过大。
- **多头注意力**：多个可学习投影使模型能同时关注不同的表示子空间与位置。

作者认为，与单头注意力相比，多头注意力提升了表达能力。

## 为何选择自注意力

论文从以下方面比较自注意力与循环层、卷积层：

- 每层计算复杂度
- 可并行计算量
- 位置间最大路径长度

主要结论：

- 自注意力具有恒定的序列深度
- 它缩短了相距较远的词元之间的路径
- 对于典型句子长度输入，它比循环结构更高效

论文中的表格显示：自注意力每层具有恒定的序列操作数与恒定的最大路径长度，而循环与卷积替代方案需要更长的顺序计算或更长的依赖路径。

## 位置信息

由于架构不含循环或卷积，必须显式添加位置信息。论文使用固定的正弦编码，因其可能泛化到更长序列并便于相对位置推理。可学习的位置嵌入也经过测试，效果相近。

## 训练设置

模型在 [[entities/wmt-2014|WMT 2014]] [[entities/wmt-2014-english-german|英德]] 与 [[entities/wmt-2014-english-french|英法]] 翻译数据上训练，使用：

- 字节对编码或词片词表
- Adam 优化器
- 学习率预热与逆平方根衰减
- Dropout 与标签平滑

大模型在 8 块 P100 GPU 上训练约 3.5 天；基础模型训练约 12 小时。

## 结果

Transformer 在当时取得了最先进的结果：

- **[[entities/wmt-2014-english-german|WMT 2014 英德翻译]]**：大模型 BLEU 28.4
- **[[entities/wmt-2014-english-french|WMT 2014 英法翻译]]**：大模型 BLEU 41.8

这些结果超越了包括集成系统在内的先前方法，同时训练成本显著更低。

## 消融与变体

论文评估了架构变体，发现：

- 更多注意力头在一定范围内有帮助
- 键维度过小会损害性能
- 更大模型表现更好
- Dropout 对泛化很重要
- 可学习位置嵌入与正弦编码效果相近

## 超越翻译的泛化

为检验模型是否能迁移到机器翻译之外，作者将其应用于英语成分句法分析。Transformer 在有监督与半监督设置下均表现竞争力，表明该架构能很好地泛化到结构化预测任务。

## 解释与可视化

注意力可视化表明，各头学习到不同行为，例如：

- 长距离依赖追踪
- 指代消解
- 句法结构识别

作者将其作为证据，表明注意力头能捕获可解释的语言模式。

## 结论

本文确立了 Transformer 作为循环与卷积序列模型的简洁有效替代方案。其纯注意力设计提升了并行度、降低了训练成本，并实现了最先进的翻译质量。该工作成为自然语言处理及更广泛领域中现代 [[concepts/transformer-models|Transformer 模型]] 的基础。

## 相关概念
- [[concepts/attention-mechanisms]]
- [[concepts/transformer-models]]
- [[concepts/positional-encoding]]

## 实体
- [[entities/google]]
- [[entities/google-brain]]
- [[entities/google-research]]
- [[entities/university-of-toronto]]
- [[entities/nips-2017]]
- [[entities/wmt-2014]]
- [[entities/wmt-2014-english-german]]
- [[entities/wmt-2014-english-french]]
- [[entities/tensor2tensor]]
