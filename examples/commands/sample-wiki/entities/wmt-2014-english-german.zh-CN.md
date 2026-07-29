---
sources: ["summaries/attention-is-all-you-need.md"]
type: "Event"
description: "WMT 2014 英德机器翻译基准。"
---

# WMT 2014 英德

## 是什么

WMT 2014 英德（English-German）是用于评估序列到序列模型的机器翻译基准。在 [[summaries/attention-is-all-you-need]] 中，它是测试 [[concepts/transformer-models|Transformer]] 的两个主要任务之一。

## 论文中的关键事实

- 数据集包含约 **450 万** 句对。
- 句子使用 **字节对编码（byte-pair encoding）** 编码，共享源-目标词表约 **37,000** 个词元。
- 训练批次按近似序列长度组成，每批约含 **25,000** 个源词元与 **25,000** 个目标词元。
- 论文在 **newstest2014** 测试集上报告结果。
- Transformer 大模型达到 **28.4 BLEU**，超过先前系统 2 BLEU 以上。
- 基础模型也以远低于先前已发表模型的训练成本取得更优表现。

## 在论文中的角色

该基准是主要的英译德评估，用于证明仅依赖注意力的架构可以超越循环与卷积序列模型。它是 [[concepts/attention-mechanisms]] 在机器翻译中有效性的主要证据。

## 相关页面

- [[summaries/attention-is-all-you-need]]
- [[concepts/transformer-models]]
- [[concepts/attention-mechanisms]]
- [[entities/wmt-2014]]
- [[entities/tensor2tensor]]
