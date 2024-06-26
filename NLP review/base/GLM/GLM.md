# GLM（Generative Language Model） 模型

[GLM：ChatGLM的基座模型](https://zhuanlan.zhihu.com/p/618166630)

---

自然语言处理NLP 技术中通常包含**三类任务**：自然语言理解NLU（包括文本分类、分词、句法分析、信息抽取等）、有条件生成任务（seq-seq，如翻译任务、QA）和无条件生成任务（用预训练模型直接生成内容）。

当前**预训练模型**也主要包括三类：自编码模型、自回归模型和编码解码模型。但这些预训练模型都不足以在所有 NLP 任务中都能展现出良好的性能。清华大学针对上述三种任务提出了一种基于 **自回归空白填充（Autoregressive Blank Infilling）** 的通用语言模型（GLM），在兼顾三方面任务，且性能表现良好。在持续的探索中，GLM系列模型发布了GLM-130B、ChatGLM、ChatGLM-6B、ChatGLM2-6B等大模型组。


<br>
<br>

### Autoregressive Blank Infilling 流程

Autoregressive blank infilling 是一种用于训练语言模型的方法，它要求模型在给定部分遮蔽（masked）的输入序列的情况下，依次生成遮蔽部分的内容。这种方法通常用于**自回归语言模型**，如 GPT-style 模型，它们在生成文本时是逐个 token 生成，每个生成的 token 都依赖于之前生成的 token。

以下是 autoregressive blank infilling 的基本流程：
1. **选择遮蔽位置**：首先，从原始文本中随机选择一些连续的 token 作为遮蔽 span。这些遮蔽的 span 可以有不同的长度，并且可以有不同的数量。选择遮蔽 span 的方式可以是随机的，也可以是基于某些规则，比如确保遮蔽的 span 覆盖不同的语法结构或语义信息。
2. **遮蔽输入序列**：将选定的遮蔽 span 替换为一个特殊的遮蔽 token（例如，"[MASK]"）。这样，输入序列就变成了一个包含遮蔽部分的新序列。
3. **自回归生成**：模型从输入序列的开始部分开始，逐个 token 地生成输出。当模型生成到遮蔽 token 时，它需要预测遮蔽 span 中的第一个 token。一旦模型生成了第一个 token，它将这个 token 添加到输入序列中，并继续生成下一个 token，直到遮蔽 span 被完全填充。
4. **重复生成**：如果输入序列中有多个遮蔽 span，模型会重复上述过程，逐一生成每个遮蔽 span 中的 token，直到所有的遮蔽 span 都被填充。
5. **损失函数**：在训练过程中，模型的损失函数通常是交叉熵损失，它衡量模型生成的 token 序列与原始序列中遮蔽 span 的真实 token 序列之间的差异。
6. **更新模型参数**：通过反向传播计算损失函数关于模型参数的梯度，并使用梯度下降或其他优化算法更新模型参数。


Autoregressive blank infilling 方法迫使模型学习如何根据上下文信息生成连续的 token 序列，这有助于提高模型对语言的理解和生成能力。这种方法在训练过程中可以增加模型的不确定性，迫使模型更加全面地学习语言的各种特征。



<br>
<br>
<br>


### GLM 模型架构
**GLM使用单个Transformer，并对架构进行了修改:**

(1)调整layer normalization和residual connection的顺序。(先归一化，再残差连接)

(2)使用单一线性层进行输出token预测。

(3)将ReLU激活函数替换为GeLUs。

