# 不同的注意力机制

- [参考资料](https://blog.csdn.net/TFATS/article/details/133042246)

- [transformer讲解](https://transformers.run/c1/transformer/)


---

- [不同的注意力机制](#不同的注意力机制)
    - [1. 多头注意力机制（Multi-Head Attention, MHA）](#1-多头注意力机制multi-head-attention-mha)
    - [2. 多头查询注意力机制（Multi-Query Attention, MQA）](#2-多头查询注意力机制multi-query-attention-mqa)
    - [3. 分组查询注意力机制（Grouped-Query Attention, GQA）](#3-分组查询注意力机制grouped-query-attention-gqa)
    - [区别和联系](#区别和联系)

---

![attention difference](/NLP%20review/assets/mha_mqa_gqa.png)
> [GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints](https://arxiv.org/abs/2305.13245v3) 中列出的图，很清楚表示了三者的区别和联系; GQA 是另外两种的折中选择


<br>


在深度学习中，特别是自然语言处理（NLP）领域，多头注意力机制（Multi-Head Attention, MHA）、多头查询注意力机制（Multi-Query Attention, MQA）和分组查询注意力机制（Grouped-Query Attention, GQA）是三种常见的注意力机制变体。它们在处理序列数据时各有特点，下面我将分别介绍它们的内容，然后再讨论它们的区别和联系。

<br>

<br>

### 1. 多头注意力机制（Multi-Head Attention, MHA）

**内容：**
- **多头注意力机制**是Transformer模型中的核心组件之一。它通过将输入的查询（Query）、键（Key）和值（Value）分别线性投影到多个不同的子空间（即“头”），然后在每个子空间中独立计算注意力权重，最后将所有头的输出拼接起来并通过一个线性层进行整合。
- **公式：**
  $$\text{MHA}(Q, K, V) = \text{Concat}(\text{head}_1, \text{head}_2, \dots, \text{head}_h) W^O$$

  其中，每个头的计算为：
  $$\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

  这里的 \($W_i^Q$\)、\($W_i^K$\) 和 \($W_i^V$\) 是第 \(i\) 个头的线性投影矩阵。


<br>

<br>


### 2. 多头查询注意力机制（Multi-Query Attention, MQA）

**内容：**
- **多头查询注意力机制**是对MHA的一种优化。在MQA中，查询（Query）被投影到多个头中，而键（Key）和值（Value）则共享同一个投影矩阵。这样做的好处是减少了计算量和内存占用，因为键和值的投影矩阵只需要存储一次。
- **公式：**
 $$ \text{MQA}(Q, K, V) = \text{Concat}(\text{head}_1, \text{head}_2, \dots, \text{head}_h) W^O$$
  其中，每个头的计算为：
 $$ \text{head}_i = \text{Attention}(QW_i^Q, KW^K, VW^V)$$
  这里的 \(W^K\) 和 \(W^V\) 是共享的键和值投影矩阵。

<br>

<br>

### 3. 分组查询注意力机制（Grouped-Query Attention, GQA）

**内容：**
- **分组查询注意力机制**是MHA和MQA的一种折中方案。在GQA中，查询（Query）被分成多个组，每个组内的查询共享一个键（Key）和值（Value）的投影矩阵，但不同组之间的投影矩阵是不同的。这样既保留了MHA的多头特性，又减少了MQA中的冗余计算。
- **公式：**
  $$\text{GQA}(Q, K, V) = \text{Concat}(\text{group}_1, \text{group}_2, \dots, \text{group}_g) W^O$$
  其中，每个组的计算为：
  $$\text{group}_i = \text{Concat}(\text{head}_{i,1}, \text{head}_{i,2}, \dots, \text{head}_{i,h_i}) W_i^O$$
  每个头的计算为：

  $$\text{head}_{i,j} = \text{Attention}(Q_{i,j}W_{i,j}^Q, KW_i^K, VW_i^V)$$
  这里的 \(W_i^K\) 和 \(W_i^V\) 是第 \(i\) 组的共享键和值投影矩阵。

<br>
<br>

### 区别和联系

**区别：**
- **MHA**：查询、键和值都被投影到多个头中，每个头独立计算注意力权重，然后拼接输出。
- **MQA**：查询被投影到多个头中，而键和值共享同一个投影矩阵，减少了计算量和内存占用。
- **GQA**：查询被分成多个组，每个组内的查询共享一个键和值的投影矩阵，不同组之间的投影矩阵不同，既保留了多头的特性，又减少了冗余计算。

**联系：**
- 这三种注意力机制都是基于Transformer模型的注意力机制的变体，旨在提高计算效率或模型性能。
- MQA和GQA都可以看作是对MHA的优化，通过减少键和值的投影矩阵数量来降低计算复杂度。
- GQA可以看作是MHA和MQA的折中方案，既保留了多头的特性，又减少了计算量。

总的来说，MHA是最基础的多头注意力机制，MQA和GQA则是在MHA的基础上进行了优化，以适应不同的计算资源和性能需求。
