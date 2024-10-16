# pretrain 笔记总结

>来自知乎作者 ybq [LLM训练-pretrain](https://zhuanlan.zhihu.com/p/718354385https://zhuanlan.zhihu.com/p/718354385)

---


这篇文章是关于如何从零开始进行大型语言模型（LLM）的预训练（pretrain）工作的详细介绍。文章分为多个部分，涵盖了预训练的背景、数据篇、训练篇、评估篇以及总结和致谢等环节。下面是对文章内容的详尽笔记：

### 背景篇
- 当前，大型模型(dense)如`qwen`、MOE模型`deepseek`、小型模型`minicpm`等已经开源，但训练框架和数据等核心内容仍然闭源。
- 自研预训练模型的意义在于能够为模型迭代做出贡献，优化模型以适应特定领域，以及更好地控制模型的对齐阶段。

<br>

### 数据篇
- **数据爬取**：需要大量数据，可以通过爬网页、购买等方式获取。
- **数据清洗**：使用模型对数据质量打分，选择BERT家族模型进行训练，结合**规则**进行数据过滤。(规则非常重要！)
- **数据去重**：使用minhash等技术对大规模数据进行去重(hadoop, spark)。
- **数据配比**：根据数据类型（如新闻、百科、代码）进行分类，并调整各类数据的比例。
- **数据顺序**：通过**课程学习**的方式，合理安排训练数据的**顺序**，以优化模型学习效果。（学习顺序非常影响效果, 作者推荐 llama 的 [in-context pretrain](#in-context-pretrain)）
- **数据流水线**：pretrain 的两个进程是独立的：“数据处理进程”和“模型训练进程”。避免 GPU 空闲

> 多读各公司的技术报告, 不要随便创新，落地的话跟着别人的架构才能避免踩坑

<br>

### 训练篇
- **Tokenizer**：选择合适的分词器，注意词汇表的扩展和压缩率。（原文有几个细节！**扩词表并不会使得效果特别好**。）
- **模型结构**：建议模仿`llama`的结构，**避免**不必要的创新。（[rope + gqa + rms_norm + swiglu](#rope--gqa--rms_norm--swiglu)，目的是防止踩坑，特别是落地需要快并且省钱）
- **模型参数**：选择合适的模型大小和超参数，考虑训练和推理的算力。
- **训练框架**：选择适合的框架，如Megatron或DeepSpeed，根据训练数据量和模型大小。(`DeepSpeed` 低代码，但是性能损失多，而且慢；`Megatron` bug多，但优化好，是真的。 **T 级别的 token 训练量必须是 megatron**，B 的级别 token 训练量无所谓。)
- **训练技巧**：优化训练效率，分析loss曲线，调整学习率等。（在显存够用的情况下，能不引入 tensor_parallel，pipeline_parallel，sequence_parallel 就不要去引入; **`data_parallel` 几乎是唯一不牺牲算力的并行方式**）

> 一个好模型是“身材匀称”的。也就是说标准的 llama 结构应该是横向和纵向呈某种比例的，所以在增大模型 size 的时候，layer_num 和 hidden_size 要一起递增。
> 
> [rope 的 NTK 外推方法](#rope的ntk外推方法深入解读)已经是各大厂标配的方案：4K/8K + rope 小 base + 90% 数据量 --> 32K/64K + rope 大 base + 10% 数据量

<br>

### 评估篇
- **PPL（Perplexity）**：通过测试集的loss衡量模型效果。
- **Benchmark**：使用改造过的benchmark来评估模型，避免直接使用选择题形式。
- **概率探针**：监控特定token或句子的概率变化，以评估模型的知识掌握程度。

> ppl 只能是自己的模型和自己比，前面说过，由于 tokenizer 的压缩率不同，不同模型的 loss 没有明确可比性，全是字没有词的 tokenizer，loss 一定是最低的
> 
> 现在最主流的 benchmark，形式也都实在是有点单一了。全是选择题，而且是没有 cot 环节的选择题

<br>

### 总结篇
- 文章强调了预训练过程中每个环节的重要性，特别是**数据清洗和处理**。






<br>
<br>
<br>


<br>
<br>
<br>

<br>
<br>
<br>


# 解释


## in-context pretrain


#### Llama的In-context Pretrain工作：让模型直接从示例中学习

**In-context pretrain**，直译为“上下文预训练”，是一种让模型直接从给定的示例中学习任务的方法。与传统的监督学习不同，这种方法不需要大量的标注数据，而是通过提供一些示例，让模型自己去“理解”任务，并生成相应的输出。

#### Llama的In-context Pretrain是如何工作的？

Llama模型在进行In-context pretrain时，通常会采用以下步骤：

1. **准备示例数据**：收集大量包含输入和输出对的示例数据。这些示例数据可以是文本、代码、或者其他形式。
2. **构建输入**：将多个示例按照一定的顺序拼接在一起，形成一个输入序列。
3. **训练模型**：将构建好的输入序列输入到模型中，让模型预测下一个token。
4. **迭代优化**：通过不断调整模型的参数，使得模型生成的输出与真实的输出更加接近。

**举例说明：**

假设我们想让Llama模型完成翻译任务。我们可以提供一些英法互译的示例，例如：

* 输入：Hello, how are you?
* 输出：Bonjour, comment allez-vous ?

在训练过程中，模型会看到大量的这样的示例，并学会将英语句子翻译成法语。当我们给模型一个新的英语句子时，它就可以根据之前学到的知识，生成对应的法语翻译。

#### Llama的In-context Pretrain的优势

* **无需大量标注数据**：相比于传统的监督学习，In-context pretrain对标注数据的依赖较小。
* **灵活性和适应性强**：模型可以通过调整示例来适应不同的任务和领域。
* **效果显著**：在许多自然语言处理任务上，In-context pretrain都取得了非常好的效果。

#### Llama的In-context Pretrain的局限性

* **示例质量的影响**：示例数据的质量对模型的性能影响很大。如果示例数据质量不高，模型的学习效果也会受到影响。
* **计算资源消耗大**：训练大型语言模型需要大量的计算资源。

#### 总结

Llama的In-context pretrain是一种非常有前景的预训练方法，它使得模型能够更好地理解和生成文本。通过这种方式，我们可以训练出更加智能、更加灵活的语言模型。


<br>
<br>
<br>



## rope + gqa + rms_norm + swiglu

这句话中提到的几个名词，都是自然语言处理（NLP）模型中常用的技术组件，特别是在大型语言模型（LLM）中。它们共同构成了Llama模型的底层架构，为模型提供了强大的语言处理能力。

### 各组件的含义

* **rope (Rotary Position Embedding)**：[旋转位置编码](/NLP%20review/base/positional_encoding.md)。是一种位置编码方法，用于向模型提供关于词语在序列中的位置信息。与传统的绝对位置编码相比，旋转位置编码能够更好地捕捉序列中的相对位置关系，从而提高模型的长期依赖建模能力。
* **gqa (Global Query Attention)**：[全局查询注意力](https://zhuanlan.zhihu.com/p/645865956)。这是一种注意力机制的变体，旨在捕捉序列中更全局的依赖关系。它通过对所有查询向量进行加权求和，得到一个全局的查询向量，从而在注意力计算中引入更广泛的上下文信息。
* **rms_norm (Root Mean Square Normalization)**：根均方归一化。这是一种层归一化（Layer Normalization）的变体，通过对每个神经元的输入进行归一化，来稳定模型的训练过程，加速收敛。
* **swiglu (Swish-GELU)**：一种激活函数，结合了Swish和GELU的优点。Swish是一种非线性激活函数，而GELU是一种门控激活函数，它们的组合可以提高模型的非线性表达能力。

### 这些组件在Llama模型中的作用

* **rope**：帮助模型理解词语在句子中的位置关系，从而更好地处理长序列的文本。
* **gqa**：增强模型对全局上下文的理解，提高模型在复杂任务上的表现。
* **rms_norm**：稳定模型的训练过程，加速模型的收敛。
* **swiglu**：增强模型的非线性表达能力，提高模型的拟合能力。


<br>
<br>
<br>



## rope的NTK外推方法：深入解读

**rope** (Rotary Position Embedding) 是一种旋转位置编码，用于在Transformer模型中表示序列中每个token的位置信息。而[NTK(Neural Tangent Kernel) ](https://zhuanlan.zhihu.com/p/210506590)是一种理论工具，用于分析深度神经网络的泛化能力。将这两者结合，就形成了**rope的NTK外推方法**，这是一种用于扩展模型上下文窗口（即模型能够处理的序列长度）的技术。

### 为什么需要NTK外推？

* **模型限制：** 大多数Transformer模型在处理长序列时会遇到困难，因为注意力机制的计算复杂度会随着序列长度的平方增长。
* **应用需求：** 在许多实际应用场景中，我们希望模型能够处理更长的序列，例如生成更长的文本、总结更长的文档等。

### NTK外推的原理

NTK外推的核心思想是：通过分析模型的NTK，我们可以推断出模型在不同上下文长度下的行为。具体来说，我们可以利用NTK来估计模型在处理更长序列时的性能，从而指导我们对模型进行调整。

**rope的NTK外推**则是在rope的基础上，利用NTK的性质来扩展模型的上下文窗口。通过对rope的频率进行调整，可以使得模型在处理更长序列时仍然能够保持较好的性能。

### rope的NTK外推的优势

* **有效扩展上下文窗口：** 通过NTK外推，可以显著扩展模型的上下文窗口，使其能够处理更长的序列。
* **保持模型性能：** 在扩展上下文窗口的同时，能够较好地保持模型在其他任务上的性能。
* **理论基础扎实：** NTK提供了一套理论框架来分析模型的泛化能力，为rope的NTK外推提供了坚实的理论基础。

### 总结

rope的NTK外推是一种非常有前景的技术，它为我们提供了一种有效的方法来扩展Transformer模型的上下文窗口。通过对rope的频率进行调整，并结合NTK的理论分析，我们可以构建出能够处理更长序列的模型，从而在更多的自然语言处理任务中取得更好的效果。


