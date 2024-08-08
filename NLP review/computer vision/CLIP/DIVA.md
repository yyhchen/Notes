# DIVA

使用 Diffusion 解决 CLIP中 无法区分**细粒度视觉细节**的问题

- [解读](https://mp.weixin.qq.com/s/nEaXSsQE0OmPcwk4QFDGRw)
- [paper](https://arxiv.org/abs/2407.20171)
- [code](https://github.com/baaivision/DIVA)

---

<br>

### CLIP 缺陷

高度依赖于图像-文本数据对，无法仅在图像数据上实现预期效果

尽管CLIP在零样本任务中表现出色，但由于对比学习范式和训练中使用的噪声图像-文本对，其在感知理解方面存在一些局限性。这些局限性包括难以准确理解长文本和难以辨别相似图像中的细微差异。虽然一些研究试图解决长文本理解问题，但改善CLIP的细粒度视觉感知能力的研究仍然不足。**感知视觉细节的能力对于基础模型至关重要，而CLIP在这方面的不足直接影响了以CLIP作为视觉编码器的视觉和多模态模型的表现。**




<br>



### DIVA 解决方式

[Diffusion Feedback Helps CLIP See Better](https://arxiv.org/abs/2407.20171) 通过 **自监督学习范式解决CLIP无法区分细粒度视觉细节的问题**。 基于文本到图像的扩散模型能够生成具有丰富细节逼真图像的先验，探索了利用扩散模型的生成反馈来优化CLIP表征的潜力。


<img src='/NLP review/assets/DIVA.png'>


<br>

如上图所示，DIVA主要由**两个部分组成**：**一是**需要增强视觉感知能力的CLIP模型，**二是**提供生成反馈的预训练扩散模型。

输入原始图像和空文本（**图中标记为"Null"**）后，CLIP模型会编码相应的视觉特征，这些特征将与**来自扩散模型文本编码器**的空文本嵌入结合，为扩散过程提供条件。对于添加了噪声的图像，扩散模型尝试在上述条件下预测从前一步到当前步骤中添加的噪声。在训练过程中，除了CLIP模型外，所有部分的权重都保持不变，训练目标只是最小化重建损失（即扩散反馈指导）。通过这种方式，通过约束扩散模型更准确地预测添加的噪声，CLIP的原始语义丰富的判别表示将通过扩散反馈逐渐优化为包含更多视觉细节的表示。