# 精确率（precision）和召回率（recall）关系


---


**常见一些问题：**


### 为什么精确率越高，召回率越低？

在二分类问题中，精确度（Precision）和召回率（Recall）是两个常用的性能指标，用于评估分类模型的表现。它们分别衡量了分类器的两个不同方面：

1. **精确度（Precision）：**
   - 精确度是**指在所有被分类为正类别的样本中，正确被分类为正类别的样本所占的比例**。即在**所有预测**为正类别的样本中，正确预测为正类别的样本的比例。
   - 精确度高表示模型预测为正类别的样本中，真正属于正类别的比例较高。

2. **召回率（Recall）：**
   - 召回率是**指在所有真实的正类别样本中，被正确预测为正类别的样本所占的比例**。即在**所有实际**为正类别的样本中，被正确预测为正类别的样本的比例。
   - 召回率高表示模型能够正确识别出较多的正类别样本，即模型对正类别的覆盖能力较强。

理解了精确度和召回率的定义后，我们可以探讨为什么在某些情况下，精确度越高，召回率越低的现象可能会出现：

1. **高精确度，低召回率的情况：**
   - 当模型更加谨慎地将样本分类为正类别时，可能会提高精确度。这意味着模型更加保守，只在非常确信的情况下将样本预测为正类别，从而减少了误报的可能性。
   - 然而，这种保守性可能会导致模型漏掉一些真实的正类别样本，从而降低了召回率。模型更倾向于避免将负类别样本错误预测为正类别，但同时可能错过了一些真正的正类别样本。

2. **应对不平衡数据：**
   - 当数据集中正类别样本相对较少，而负类别样本相对较多时，模型可能更倾向于对负类别样本进行分类，因为这样可以降低错误分类的风险，提高整体精确度。
   - 但是，由于正类别样本较少，模型可能会错过一些真正的正类别样本，导致召回率较低。

综上所述，高精确度和低召回率的现象可能源自模型对样本分类的保守性，尤其是在面对不平衡数据集时。在实际应用中，需要根据具体的问题和需求，权衡精确度和召回率，并选择合适的调节策略以达到最佳的性能。