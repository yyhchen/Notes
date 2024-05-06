# BitFit

论文：[BitFit: Simple Parameter-efficient Fine-tuning or Transformer-based Masked Language-models](https://arxiv.org/abs/2106.10199)

---

### 背景

虽然对每个任务进行全量微调非常有效，但它也会为每个预训练任务生成一个独特的大型模型，这使得很难推断微调过程中发生了什么变化，也很难部署， 特别是随着任务数量的增加，很难维护。

理想状况下，我们希望有一种满足以下条件的高效微调方法：

- 到达能够匹配全量微调的效果。
- 仅更改一小部分模型参数。
- 使数据可以通过流的方式到达，而不是同时到达，便于高效的硬件部署。
- 改变的参数在不同下游任务中是一致的。
- 上述的问题取决于微调过程能多大程度引导新能力的学习以及暴露在预训练LM中学到的能力。

虽然，之前的高效微调方法Adapter-Tuning、Diff-Pruning也能够部分满足上述的需求。但是，作者提出了一种参数量更小的稀疏的微调方法BitFit，来满足上述的需求。



<br>
<br>



### 技术原理

BitFit（论文：**BitFit: Simple Parameter-efficient Fine-tuning or Transformer-based Masked Language-models**）是一种稀疏的微调方法，==它训练时只更新bias的参数或者部分bias参数==。

对于Transformer模型而言，冻结大部分 transformer-encoder 参数，只更新bias参数跟特定任务的分类层参数。涉及到的bias参数有attention模块中计算query,key,value跟合并多个attention结果时涉及到的bias，MLP层中的bias，Layernormalization层的bias参数。

在Bert-Base/Bert-Large这种模型里，bias参数仅占模型全部参数量的0.08%～0.09%。但是通过在Bert-Large模型上基于GLUE数据集进行了 BitFit、Adapter和Diff-Pruning的效果对比发现，BitFit在参数量远小于Adapter、Diff-Pruning的情况下，效果与Adapter、Diff-Pruning想当，甚至在某些任务上略优于Adapter、Diff-Pruning。