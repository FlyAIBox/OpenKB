# 原始 Transformer 论文

本 skill 的源材料是论文 **Attention Is All You Need**。提炼出的核心观点如下：

- 当你需要并行训练以及 distant tokens 之间的短路径时，序列转换（sequence transduction）应基于 attention 而非 recurrence 来构建
- self-attention 是让每个位置在一层内读取整个序列的机制
- multi-head attention 通过让模型同时在多个 learned subspaces 中 attend，恢复表达能力
- positional encoding 是必需的，因为仅靠 attention 无法感知顺序
- masked decoder self-attention 通过阻止 future-token leakage，保持 autoregressive generation

论文的实用对比并非「attention 对抗一切」；而是针对 RNN 与 convolutional sequence models 的一项具体工程权衡。该架构通过减少 sequential computation 取胜，同时仍通过 specialized attention blocks 处理 alignment、dependency tracking 与 output generation。
