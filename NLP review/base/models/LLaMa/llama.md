# LLaMa model

[llama1 技术详解](https://www.zhihu.com/tardis/zm/art/648774481?source_id=1003)

---


### LLaMa 模型架构

LLaMa 的网络基于 Transformer 架构：

- **Pre-normalization：** 为了提高训练稳定性，LLaMa 对每个 Transformer 子层的输入进行归一化，而不是对输出进行归一化。LLaMa 使用了 RMSNorm 归一化函数。
- **SwiGLU 激活函数：** LLaMa 使用 SwiGLU 激活函数替换 ReLU 以提高性能
- **Rotary Embeddings：** 没有使用之前的绝对位置编码，而是使用了旋转位置编码（RoPE），可以提升模型的外推性。