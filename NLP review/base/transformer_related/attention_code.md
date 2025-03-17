# attention 两种框架实现形式

---


## pytorch 版本

### attention实现

Transformer 模型中的核心组件之一就是所谓的 "Scaled Dot-Product Attention" 机制。以下代码实现 Scaled Dot-Product Attention：
```python
import torch
import torch.nn as nn
class ScaledDotProductAttention(nn.Module):
    def __init__(self, d_k):
        super(ScaledDotProductAttention, self).__init__()
        self.d_k = d_k
    def forward(self, Q, K, V, attn_mask=None):
        """
        Q, K, V: [batch_size, n_head, len_q/len_k/len_v, d_k]
        attn_mask: [batch_size, n_head, len_q, len_k]
        """
        scores = torch.matmul(Q, K.transpose(-2, -1)) / (self.d_k ** 0.5)  # scores : [batch_size, n_head, len_q, len_k]
        
        if attn_mask is not None:
            scores = scores.masked_fill(attn_mask == 0, -1e9)
        
        attn = nn.Softmax(dim=-1)(scores)  # attn : [batch_size, n_head, len_q, len_k]
        context = torch.matmul(attn, V)  # context: [batch_size, n_head, len_q, d_v]
        
        return context, attn
```
这里的 `Q`, `K`, `V` 是 Transformer 模型中的 Query, Key, Value，它们通常是通过模型中的三个不同的线性层获得的。`attn_mask` 是一个可选的注意力掩码，用于在序列处理中屏蔽掉某些位置的值（例如，在文本处理中，它用于屏蔽掉对未来的依赖）。
请注意，这个代码片段仅实现了注意力机制的一个基本版本。在实际的 Transformer 模型中，通常还会使用多头注意力（Multi-Head Attention）来增强模型的能力。

<br>

#### 多头注意力机制

当然，以下是多头注意力（Multi-Head Attention）的 PyTorch 实现。这个实现基于之前提供的 `ScaledDotProductAttention` 类。
```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_head, d_k, d_v):
        super(MultiHeadAttention, self).__init__()
        self.n_head = n_head
        self.d_k = d_k
        self.d_v = d_v
        self.W_Q = nn.Linear(d_model, d_k * n_head)
        self.W_K = nn.Linear(d_model, d_k * n_head)
        self.W_V = nn.Linear(d_model, d_v * n_head)
        self.fc = nn.Linear(n_head * d_v, d_model)
        self.attention = ScaledDotProductAttention(d_k)
    def forward(self, Q, K, V, attn_mask=None):
        """
        Q, K, V: [batch_size, len_q/len_k/len_v, d_model]
        attn_mask: [batch_size, len_q, len_k]
        """
        residual, batch_size = Q, Q.size(0)
        # Linear projection
        q_s = self.W_Q(Q).view(batch_size, -1, self.n_head, self.d_k).transpose(1, 2)  # q_s: [batch_size, n_head, len_q, d_k]
        k_s = self.W_K(K).view(batch_size, -1, self.n_head, self.d_k).transpose(1, 2)  # k_s: [batch_size, n_head, len_k, d_k]
        v_s = self.W_V(V).view(batch_size, -1, self.n_head, self.d_v).transpose(1, 2)  # v_s: [batch_size, n_head, len_v, d_v]
        if attn_mask is not None:
            attn_mask = attn_mask.unsqueeze(1).repeat(1, self.n_head, 1, 1)  # attn_mask : [batch_size, n_head, len_q, len_k]
        # Scaled Dot-Product Attention
        context, attn = self.attention(q_s, k_s, v_s, attn_mask=attn_mask)  # context: [batch_size, n_head, len_q, d_v]
        # Concatenate and linear projection
        context = context.transpose(1, 2).contiguous().view(batch_size, -1, self.n_head * self.d_v)  # context: [batch_size, len_q, n_head * d_v]
        output = self.fc(context)  # output: [batch_size, len_q, d_model]
        return output, attn
```
在这个实现中，`d_model` 是模型的维度，`n_head` 是头数，`d_k` 是 key 和 query 的维度，`d_v` 是 value 的维度。通常情况下，`d_k` 和 `d_v` 被设置为 `d_model // n_head`，但这是可配置的。
在 `forward` 函数中，我们首先通过线性层将输入 Q, K, V 投影到多个头中，然后使用之前定义的 `ScaledDotProductAttention` 来计算每个头的注意力。最后，我们将所有头的输出拼接起来，并通过另一个线性层进行投影，得到最终的输出。
请注意，这个实现假设所有的输入序列长度是相同的，并且没有使用位置编码。在实际应用中，您可能需要添加位置编码，并确保注意力掩码能够正确地处理不同长度的序列。


<br>
<br>
<br>




## tensorflow 版本

### attention实现

以下是使用 TensorFlow 2.x 实现的 Scaled Dot-Product Attention 机制。这个实现不依赖于任何特定的外部库，只使用 TensorFlow 的内置功能。
```python
import tensorflow as tf
def scaled_dot_product_attention(q, k, v, mask=None):
    """
    q, k, v: [batch_size, n_head, seq_len, d_k]
    mask: [batch_size, n_head, seq_len, seq_len]
    """
    matmul_qk = tf.matmul(q, k, transpose_b=True)  # matmul_qk: [batch_size, n_head, seq_len, seq_len]
    # scale matmul_qk
    d_k = tf.cast(tf.shape(k)[-1], tf.float32)
    scaled_attention_logits = matmul_qk / tf.math.sqrt(d_k)
    # add the mask to the scaled tensor.
    if mask is not None:
        scaled_attention_logits += (mask * -1e9)
    # softmax in the last axis (seq_len_k)
    attention_weights = tf.nn.softmax(scaled_attention_logits, axis=-1)  # attention_weights: [batch_size, n_head, seq_len, seq_len]
    output = tf.matmul(attention_weights, v)  # output: [batch_size, n_head, seq_len, d_v]
    return output, attention_weights
# Example usage:
# Define query, key, and value matrices (for simplicity, let's assume they are already projected)
# q = ...
# k = ...
# v = ...
# mask = ...  # Optional attention mask
# output, attention_weights = scaled_dot_product_attention(q, k, v, mask)
```
这个函数接受 query (q), key (k), 和 value (v) 作为输入，它们通常是通过模型中的线性层获得的。`mask` 是一个可选的注意力掩码，用于在序列处理中屏蔽掉某些位置的值。

<br>


### 多头注意力机制

以下是使用 TensorFlow 2.x 实现的多头注意力（Multi-Head Attention）机制。这个实现基于之前提供的 `scaled_dot_product_attention` 函数。
```python
def multi_head_attention(q, k, v, d_model, num_heads):
    """
    q, k, v: [batch_size, seq_len, d_model]
    """
    # Split the last dimension into (num_heads, depth).
    d_k = d_model // num_heads
    q = tf.reshape(q, (-1, tf.shape(q)[1], num_heads, d_k))
    k = tf.reshape(k, (-1, tf.shape(k)[1], num_heads, d_k))
    v = tf.reshape(v, (-1, tf.shape(v)[1], num_heads, d_k))
    # Transpose the last two dimensions (seq_len, d_k) and (num_heads, d_k).
    q = tf.transpose(q, perm=(0, 2, 1, 3))
    k = tf.transpose(k, perm=(0, 2, 1, 3))
    v = tf.transpose(v, perm=(0, 2, 1, 3))
    # Call scaled_dot_product_attention
    scaled_attention, attention_weights = scaled_dot_product_attention(q, k, v)
    # Transpose back to move the head dimension back: (batch_size, seq_len, num_heads, d_k)
    scaled_attention = tf.transpose(scaled_attention, perm=(0, 2, 1, 3))
    # Concatenate attention heads.
    concat_attention = tf.reshape(scaled_attention, (-1, tf.shape(scaled_attention)[1], d_model))
    # Final linear projection.
    output = tf.keras.layers.Dense(d_model)(concat_attention)
    return output, attention_weights
# Example usage:
# Define query, key, and value matrices (for simplicity, let's assume they are already projected)
# q = ...
# k = ...
# v = ...
# d_model = ...  # Dimensions of the model
# num_heads = ...  # Number of attention heads
# output, attention_weights = multi_head_attention(q, k, v, d_model, num_heads)
```
在这个实现中，`d_model` 是模型的维度，`num_heads` 是头数。`q`, `k`, `v` 是通过模型中的线性层获得的输入。
在 `multi_head_attention` 函数中，我们首先将输入 Q, K, V 分割成多个头，然后使用之前定义的 `scaled_dot_product_attention` 来计算每个头的注意力。最后，我们将所有头的输出拼接起来，并通过一个线性层进行投影，得到最终的输出。
请注意，这个实现假设所有的输入序列长度是相同的，并且没有使用位置编码。在实际应用中，可能需要添加位置编码，并确保注意力掩码能够正确地处理不同长度的序列。


