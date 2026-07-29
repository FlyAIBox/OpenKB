---
sources: ["summaries/attention-is-all-you-need.md"]
type: "Event"
description: "WMT 2014 英法机器翻译基准。"
---

## 概述

WMT 2014 英法（English-French）是论文 [[summaries/attention-is-all-you-need]] 中使用的机器翻译评估任务与数据集。它是比较神经机器翻译系统翻译质量与训练成本的主要基准之一。

## 在论文中的角色

论文在该基准上报告结果，以表明 [[concepts/transformer-models|Transformer]] 以远低于先前模型的训练成本取得优异的翻译质量。

## 文档中的关键事实

- 训练所用数据集包含约 **3600 万** 句对。
- 词元被切分为 **32,000** 词片的词表。
- 论文报告 Transformer 大模型在该任务上的 **BLEU 分数为 41.8**。
- 模型在 **8 块 GPU 上训练 3.5 天**。
- 与先前的单模型系统相比，Transformer 以远低于后者的训练成本达到新的最先进水平。

## 相关概念与实体

- [[concepts/transformer-models]]
- [[concepts/attention-mechanisms]]
- [[entities/wmt-2014]]
- [[entities/google]]
- [[entities/google-brain]]
- [[entities/google-research]]
- [[entities/tensor2tensor]]
- [[summaries/attention-is-all-you-need]]

## 备注

在论文中，该基准与 WMT 2014 英德任务配对，作为主要的翻译评估设置。
