# causal multi-head attention

`xformer` 库中的 `causal multi-head attention` 

---

xformers 是一个由 Meta（Facebook）开发的用于高效和灵活的 Transformer 模型实现的库。它提供了一些优化的模块和工具，用于加速 Transformer 模型的训练和推理，包括不同类型的注意力机制实现。

<br>
<br>

### Causal Multi-Head Attention

在 Transformer 模型中，注意力机制（Attention Mechanism）是核心组件。Causal Multi-Head Attention 是一种特殊类型的注意力机制，通常用于自回归（autoregressive）生成任务，如语言模型生成（例如 GPT-3）。

<br>

#### 什么是 Causal Multi-Head Attention？

1. **Causal Attention**：
   - Causal attention 又称为 Masked Attention。在这种注意力机制中，查询位置 \( i \) 只能关注位置 \( j \) 的信息，前提是 \( j \leq i \)。这意味着模型在预测下一个词时，只能利用之前的词信息，不能看到未来的词。
   - 在自回归任务中，这种因果性质确保了模型生成序列时不会“偷看”未来的词。

2. **Multi-Head Attention**：
   - Multi-Head Attention 是标准 Transformer 中的一部分。它通过将查询、键和值映射到多个子空间，计算多个注意力头（heads），然后将这些头的输出拼接在一起。这允许模型从不同的角度关注输入序列中的不同部分。

<br>

#### Causal Multi-Head Attention 的工作原理

Causal Multi-Head Attention 结合了因果注意力和多头注意力的优点。具体步骤如下：

1. **线性映射**：
   - 输入序列被映射到多个子空间，生成多个查询（Q）、键（K）和值（V）。

2. **计算注意力得分**：
   - 每个注意力头分别计算查询和键的点积，并除以缩放因子（通常是键的维度的平方根）。
   - 计算得分矩阵后，应用因果掩码，使得每个位置 \( i \) 只能关注位置 \( j \)（前提是 \( j \leq i \）。

3. **应用 Softmax**：
   - 掩码后的得分矩阵通过 Softmax 转换为注意力权重。

4. **加权求和**：
   - 每个注意力头对值（V）进行加权求和，生成每个头的输出。

5. **拼接和线性映射**：
   - 将所有头的输出拼接在一起，并通过线性映射生成最终输出。

<br>
<br>

### 示例代码

以下是使用 xformers 库中的 Causal Multi-Head Attention 的示例代码：

```python
import torch
from xformers.components import MultiHeadAttention

# 假设有一个输入序列
input_seq = torch.rand(10, 32, 512)  # (sequence_length, batch_size, model_dim)

# 配置参数
num_heads = 8
model_dim = 512
causal = True  # 表示因果注意力

# 创建 Multi-Head Attention 模块
mha = MultiHeadAttention(
    num_heads=num_heads,
    dim_model=model_dim,
    causal=causal
)

# 前向传播
output = mha(input_seq, input_seq, input_seq)

print(output.shape)  # 输出形状应为 (sequence_length, batch_size, model_dim)
```

在这个示例中，`causal=True` 表示我们使用的是因果多头注意力。**这种设置确保了注意力机制只能关注当前及之前的时间步**，从而保持自回归生成任务的因果性。

### 总结

Causal Multi-Head Attention 是一种在自回归任务中非常重要的注意力机制，通过使用因果掩码确保模型在生成序列时只能使用之前的词信息。xformers 库提供了高效的实现，便于开发者在各种 Transformer 模型中使用这种机制。