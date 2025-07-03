---
title: 神经网络学习笔记3：注意力池化在 Transformer 模型中的应用
date: 2025-05-28 16:23:41
categories:
- 神经网络与深度学习
---

本文介绍了一种高效的序列表示方法——注意力池化（Attention Pooling），并将其应用于文本匹配任务。实验表明，在引入注意力池化层后，模型准确率由79.98%显著提高至81.64%，验证了其有效性。

<!-- more -->

## 注意力池化介绍

在自然语言处理任务中，如何将变长的序列信息压缩为固定长度的向量表示，是一个核心问题。池化操作是实现这一目标的关键方式，常见的方法有平均池化和最大池化。然而，这些方法并不能考虑序列中各部分的重要性差异。

注意力池化（Attention Pooling）提供了一种更灵活且可学习的机制，用于对序列中不同位置的特征赋予不同的权重，从而生成更加精确的全局表示。其本质是一种加权求和机制，其中每个时间步的表示会根据其“重要性”被赋予一个注意力权重。公式如下：

$$
\text{Output} = \sum_{i=1}^T \alpha_i \cdot h_i
$$

其中，$h_i$ 是序列中第 $i$ 个时间步的特征向量，$\alpha_i$ 是其对应的注意力权重，满足 $\sum \alpha_i = 1$。

在项目中，注意力池化由以下 PyTorch 模块实现：

```python
class AttentionPooling(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.att = nn.Linear(input_dim, 1)

    def forward(self, x, mask):
        att_weights = self.att(x).squeeze(-1)
        att_weights = att_weights.masked_fill(~mask.bool(), float('-inf'))
        att_weights = torch.softmax(att_weights, dim=-1)
        pooled = torch.bmm(att_weights.unsqueeze(1), x).squeeze(1)
        return pooled
```

这段代码中，模型首先通过线性层生成每个 token 的注意力得分，随后通过 softmax 获得注意力分布，最后对序列进行加权求和。

## 在实验中的应用

在完成“对话短文本匹配”任务中，我就使用了注意力池化层。该任务的输入为两个经过分词和编码的句子对，通过 Transformer 编码后，使用注意力池化获取整句的表示，再输入一个多层感知机计算匹配分数。

网络结构如下：

* 嵌入层（包含位置编码和 token 类型嵌入）
* 双塔共享 Transformer 编码器
* **注意力池化**
* 特征拼接（$v_1, v_2, |v_1 - v_2|, v_1 * v_2$）
* 全连接网络输出匹配得分

在未使用注意力池化前，模型使用简单的平均池化策略，全验证集上的准确率为 79.98%。引入注意力池化后，模型准确率提升至 81.64%。实验结果表明，注意力池化通过让模型自主学习哪些位置更重要，可以有效增强模型的性能。
