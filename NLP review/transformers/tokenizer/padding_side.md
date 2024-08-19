# padding_side 影响

---


## padding_side

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
tokenizer.padding_side = "left"
```

- padding_side 是指 padding 的方向，默认是 "right"，即从右往左填充
- 如果设置为 "left"，则从左往右填充



<br>
<br>


## 为什么 `decoder-only` 的模型要设置成 `left`?

> [原答案](https://github.com/LianjiaTech/BELLE/issues/240)

首先，我理解 padding_side 的限制和预训练阶段是怎么 padding 的应该关系不大，因为至少就llama模型而言，预训练的阶段没有做 padding，而是把语料切分成相同长度的块（不知道像GPT2这样比较靠前的模型预训练阶段是不是也是这么操作的）。

其次，我们在训练模型的时候，将 padding 的位置放在哪里其实都一样，对模型训练来说没有任何影响，因为不管是 `<pad> <pad> <input> <target>`，还是 `<input> <target> <pad> <pad>` 都是相同的长度。 而在推理阶段，如果你是以单条生成的方式进行推理的话，padding_side也是没有影响的，因为单条样本的推理其实不需要做 padding 操作。 但实际推理的过程，免不了需要通过输入一整个 batch 的数据，提高推理的吞吐量。这样输入在送入模型的过程中就涉及到了padding操作，如果我们采用 right padding 的话（一般是通过 collate_fn 函数，将一个 batch 中的数据 padding 到当前batch中的样本最大序列长度），比如：

当 batch_size:3 , padding_side = 'right'，我们输入 `<target>` 之前的部分，让模型自回归的预测 `<target>`

```sh
I like cat <pad> <pad> <target>
you suck <pad> <pad> <pad> <target>
Trump is the American president <target>
```


这时候对于前两条数据来说，**语言模型的推理过程就与训练阶段出现了偏差，也即 input 和 要预测的 target 之间出现了不该出现的 `<pad>`**。但如果在训练阶段和推理阶段采用 left padding，就可以自然的避免这个问题了。


当 padding_side = 'left'
```sh
<pad> <pad> I like cat <target>
<pad> <pad> <pad> you suck <target>
Trump is the American president <target>
```

这样input和要预测的target之间就不再有<pad>分隔了，与训练阶段是一致的。

