# Transformer-XL

- [Transformer-xl: Attentive language models beyond a fixed-length context](https://arxiv.org/abs/1901.02860)

- [Transformer-XL及其进化XLNet](https://zhuanlan.zhihu.com/p/355366424)

---


### Transformer-XL 改进 

Transformer-XL相对于Vanilla Transformer的一个重要改进是引入了一种称为"循环机制"（Recurrence Mechanism）的新型机制，用于解决处理长序列时的梯度传播和上下文建模问题。

在Vanilla Transformer中，由于自注意力机制的局限性，模型的上下文建模能力受到了限制，特别是在处理长序列时。为了解决这个问题，Transformer-XL引入了循环机制，允许模型在处理长序列时保持长期依赖关系，并且能够有效地处理长文本序列。

循环机制的主要思想是**在模型的隐藏状态之间引入一种循环连接，以将历史信息传递到未来的时间步**。这种循环连接允许模型在每个时间步中记忆之前的隐藏状态，并在下一个时间步中使用这些记忆，从而实现了对长期依赖关系的建模。

具体来说，循环机制由两个关键组件组成：

1. **相对位置编码（Relative Positional Encoding）**：Transformer-XL引入了一种新的相对位置编码方法，允许模型在每个时间步中考虑到相对于当前位置的历史位置信息。这种相对位置编码能够有效地捕捉到文本序列中不同位置之间的相对关系，从而帮助模型更好地理解长序列中的上下文信息。

2. **循环缓存机制（Recurrent Buffer Mechanism）**：Transformer-XL引入了一种称为循环缓存（Recurrent Buffer）的机制，用于存储历史隐藏状态，并在下一个时间步中将这些历史隐藏状态与当前隐藏状态进行连接。这种循环连接允许模型在处理长序列时保持长期依赖关系，从而更好地捕捉到文本序列中的全局信息。

通过引入循环机制，Transformer-XL能够在处理长序列时取得更好的性能，特别是在语言建模和文本生成等任务中。这种循环机制有效地解决了Vanilla Transformer中的上下文建模问题，使得模型能够更好地处理长序列数据。



<br>
<br>


### 在循环缓存机制中，为什么上一个segment不参加反向传播？
在Transformer-XL的循环缓存机制中，上一个segment不参与反向传播的原因是为了避免梯度混淆（gradient confusion）问题。

梯度混淆是指**由于循环连接的引入，模型在计算梯度时可能会出现混淆，即梯度来自过去的时间步可能会影响当前时间步的梯度计算，从而导致梯度的混乱和不稳定性**。这会对模型的训练和收敛产生负面影响，甚至导致模型的性能下降。

为了解决梯度混淆问题，Transformer-XL采取了一种称为"区段级联（Segment-Level Recurrent Mechanism）"的策略。在这种策略下，模型将输入序列划分为多个不重叠的区段（segments），并且每个区段都有一个对应的循环缓存，用于存储历史隐藏状态。

在训练过程中，**模型只使用当前区段的循环缓存来计算当前时间步的隐藏状态，并且只将当前区段的梯度传递给下一个区段，而不传递给前一个区段。这样可以确保梯度只在当前区段内流动**，避免了梯度混淆问题的发生。

因此，上一个segment不参与反向传播是为了保持梯度的清晰和稳定，从而有助于模型的训练和收敛。