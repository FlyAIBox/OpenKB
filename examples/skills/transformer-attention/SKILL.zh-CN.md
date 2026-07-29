---
name: transformer-attention
description: 在讨论 Transformer self-attention、multi-head attention、positional encoding、masked decoder attention，或 attention 为何在序列模型中取代 recurrence/convolution 时使用；不适用于通用 NLP 或无关的 attention 话题。
---

# Transformer Attention 推理

本 skill 编码了原始 Transformer 背后的实用世界观：序列建模在停止逐 token 递推、改为通过 attention 让各位置直接交互时效果更好。用它回答诸如「self-attention 为何有用」「为何需要 positional encoding」「masking 保护什么」或「multi-head attention 何时有用」等问题。

## 何时使用本 skill

- 用户将 Transformer 与 RNN、LSTM、GRU 或 convolutional seq2seq 模型对比
- 用户询问 encoder 或 decoder 中 self-attention、multi-head attention 或 scaled dot-product attention 如何工作
- 用户想了解为何在纯 attention 架构中需要 positional encoding
- 用户正在调试或解释 masked self-attention、autoregressive decoding 或 encoder-decoder attention
- 用户询问 attention 为何能缩短 long-range dependency 路径或提升 parallelism
- 不适用于：心理学、视觉或推荐系统中的通用「attention」
- 不适用于：广泛的现代 LLM 训练、prompting 或 scaling-law 问题，除非焦点是 Transformer 机制本身
- 不适用于：与架构推理无关的实现级优化细节

## 核心决策规则

- **当 recurrence 成为瓶颈时，优先采用纯 attention 计算** — recurrence 强制 sequential hidden-state 更新，并在单个训练样本内阻塞 parallelism。
- **当 long-range dependencies 重要时，优先 self-attention 而非堆叠 recurrence 或 convolution** — 任意 token 可在单层内连接任意其他 token，路径长度保持较短。
- **若模型无 recurrence 或 convolution，须添加显式位置信息** — 仅靠 attention 对排列不敏感（permutation-blind），positional encoding 提供顺序。
- **autoregressive 解码时，mask 未来位置** — 否则模型会泄漏右侧信息，并 conditioning 于尚不应知晓的 tokens。
- **当单一 attention pattern 过于粗糙时，使用 multi-head attention** — 独立 head 让模型并行 attend 不同 subspaces、位置或关系类型。
- **当 key 维度较大导致 dot product 过于尖锐时，按 \(\sqrt{d_k}\) 缩放** — 这使 softmax 梯度可用，并避免 overconfident attention scores。
- **若需要 encoder-to-decoder alignment，使用 encoder-decoder attention，而非 plain self-attention** — decoder 应直接 query 编码后的源序列。
- **比较层类型时，评估 sequential depth 与 maximum path length，而非仅参数量** — Transformer 取胜是因为减少了 sequential operations 与 dependency distance。
- **若任务是句子级序列建模，self-attention 往往在计算上具有吸引力** — 当序列长度低于表示宽度时，其每层复杂度较优，这在翻译中很常见。
- **当单一 head 似乎模糊了 distinct relationships 时，将 averaging 视为局限而非优点** — 多个 head 抵消这种分辨率损失。
- **当输出质量须保持稳定时，将架构与 residual connections、layer normalization、dropout 和 label smoothing 配合使用** — 论文将这些视为使 attention stack 训练良好的组成部分。
- **若 learned positional embedding 有效，不要假设它优于 sinusoidal encoding** — 原始结果发现性能相近；选择 sinusoidal encodings 是出于 extrapolation 潜力。

## 方法

1. 识别当前 attention 角色：encoder self-attention、masked decoder self-attention 或 encoder-decoder attention。
2. 判断问题关乎 ordering、dependency distance 还是 parallelization；这些是架构变更的主要原因。
3. 若问题涉及设计选择，对照论文核心权衡检验：sequential recurrence 与带显式位置信号的 parallel attention。
4. 用 multi-head attention 与 scaling 规则解释 expressiveness 与 training stability。
5. 若用户问「为何不用 RNN？」，从 sequential computation、path length 与 long-range dependency learning 的便利性作答。

## 参考资料

- [[references/original-transformer-paper]]

## 已知缺口

- 本 skill 基于原始 Transformer 论文，不涵盖后续变体，如 sparse attention、rotary position encodings、FlashAttention 或现代 decoder-only LLM 设计。
- 不提供实现代码、各 sublayer 的 tensor shapes，或超出源材料所讨论架构选择的 training-hyperparameter 调优建议。
- 聚焦用 attention 取代 recurrence 的推理；对 convolutional 替代方案除作为对比 baseline 外未深入展开。
