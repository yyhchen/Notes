# 几种位置编码

---

### transformer原生的绝对位置编码




<br>
<br>



### RoPE

[RoFormer: Enhanced Transformer with Rotary Position Embedding](https://arxiv.org/abs/2104.09864)

和相对位置编码相比，**RoPE 具有更好的外推性**，目前是大模型相对位置编码中应用最广的方式之一。

<br>

**注：什么是大模型外推性？**

外推性是**指大模型在训练时和预测时的输入长度不一致，导致模型的泛化能力下降的问题**。例如，如果一个模型在训练时只使用了512个 token 的文本，那么在预测时如果输入超过512个 token，模型可能无法正确处理。这就限制了大模型在处理长文本或多轮对话等任务时的效果。

而RoPE能够提升大模型的外推能力（2k推8k，甚至更长）


[十分钟读懂旋转编码（RoPE）](https://www.zhihu.com/tardis/zm/art/647109286?source_id=1003)