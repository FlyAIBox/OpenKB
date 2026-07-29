---
sources: ["summaries/attention-is-all-you-need.md"]
type: "Product"
description: "用于实现与评估 Transformer 的开源 TensorFlow 工具包"
---

## 概述

Tensor2Tensor 是基于 TensorFlow 的序列建模研究与评估工具包。在 [[summaries/attention-is-all-you-need]] 中，它被描述为作者用于开发、训练与评估 Transformer 的代码库。

## 在论文中的角色

论文将 Tensor2Tensor 视为 Transformer 工作背后实现努力的重要组成部分。它用于替换较早的代码库，并加速实验、模型调参与评估。

## 文档中的关键事实

- 用于设计、实现、调参与评估 Transformer 变体。
- 帮助替换较早的内部代码库。
- 支持作者的翻译实验与更广泛的模型开发。
- 在论文结尾说明中作为与该工作相关的公开代码库被提及。

## 与 Transformer 的关系

Tensor2Tensor 与 [[summaries/attention-is-all-you-need]] 中所述 [[concepts/transformer-models|Transformer 模型]] 的开发密切相关。该工具包为依赖 [[concepts/attention-mechanisms|注意力机制]] 与 [[concepts/positional-encoding|位置编码]] 的架构提供了实验基础设施。

## 相关实体

- [[entities/google]]
- [[entities/google-brain]]
- [[entities/google-research]]
- [[entities/wmt-2014-english-german]]
- [[entities/wmt-2014-english-french]]
