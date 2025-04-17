
---


## 1. 什么是 RMSNorm？

RMSNorm 是一种旨在**替代 Layer Normalization (LayerNorm)** 的归一化技术，特别是在 Transformer 架构中。它由 Biao Zhang 和 Rico Sennrich 在论文 "[Root Mean Square Layer Normalization" (2019)](https://arxiv.org/abs/1910.07467) 中提出。

>[!NOTE]
其核心思想是：**简化 LayerNorm，只进行重新缩放 (re-scaling) 而不进行重新中心化 (re-centering)，并且使用输入的均方根 (Root Mean Square, RMS) 作为缩放因子。**


## 2. 动机：为什么需要 RMSNorm？

LayerNorm 在 Transformer 等模型中非常成功，它**通过对每个样本的层内激活值进行归一化，稳定了训练过程**，加速了收敛。LayerNorm 的标准做法包含两个步骤：

- **重新中心化 (Re-centering):** 减去均值，使得激活值的均值为 0。
- **重新缩放 (Re-scaling):** 除以标准差，使得激活值的方差为 1。
- **仿射变换 (Affine Transformation):** 乘以一个可学习的增益 (gain, $~\gamma$) 并加上一个可学习的偏置 (bias, $~\beta$)，以恢复模型的表达能力。

RMSNorm 的作者认为，**LayerNorm 的成功主要归功于其重新缩放的不变性，而重新中心化带来的贡献相对较小，甚至可能引入不必要的计算开销。** 他们通过实验发现，移除均值中心化步骤（即减去均值 $\mu$）对模型性能影响不大，但可以简化计算并提高效率。



## 3. RMSNorm 与 LayerNorm 的对比

让我们用公式来直观对比：

假设一个层的激活向量为 $x=(x_1​,x_2​,...,x_n​)$。

- **Layer Normalization (LayerNorm):**
    
    1. 计算均值 $\mu=\frac{1}{n}\sum_{i=1}^nx_i$
    2. 计算标准差 $\large\sigma=\sqrt{\frac{1}{n}\sum_{i=1}^n(x_i-\mu)^2+\epsilon}$​ （ $\epsilon$ 是为了防止除零的小常数）
    3. 归一化: $\hat{x}_i = \frac{x_i - \mu}{\sigma}$
    4. 仿射变换: $y_i=\gamma_i\hat{x}_i+\beta_i$ （$~\gamma_i$ 和 $~\beta_i$ 是可学习参数）
- **Root Mean Square Layer Normalization (RMSNorm):**
    
    1. **不计算均值！**
    2. 计算均方根 (RMS): $RMS(x)=\sqrt{\frac{1}{n}\sum_{i=1}^nx_i^2+\epsilon}$​
    3. 归一化: $\hat{x_i}​= \frac{x_i}{RMS(x)}​​$
    4. **只进行缩放变换**: $y_i = \gamma_i~\hat{x_i}$ （$~\gamma_i$​ 是可学习参数，**没有偏置 $\beta_i$​**）

**主要区别总结:**

| **特性**    | **LayerNorm**   | **RMSNorm**   |
| --------- | --------------- | ------------- |
| **中心化**   | 是 (减去均值 μ)      | **否**         |
| **缩放统计量** | 标准差 (σ)         | **均方根 (RMS)** |
| **可学习参数** | 增益 (γ), 偏置 (β)  | **仅增益 (γ)**   |
| **计算复杂度** | 相对较高 (计算均值和标准差) | 相对较低 (只计算均方根) |

## 4. RMSNorm 的数学步骤详解

对于输入向量 $x=(x_1​,x_2​,...,x_n​)$：

1. **计算平方和的均值 (Mean Square):** $$MS(x) = \frac{1}{n}\sum_{i=1}^nx_i^2$$
2. **计算均方根 (Root Mean Square):** 加入 $\epsilon$ 以保证数值稳定性。$$RMS(x) = \sqrt{MS(x) + \epsilon} = \sqrt{\frac{1}{n}\sum_{i=1}^nx_i^2 + \epsilon}$$​
3. **归一化输入:** 将输入的每个元素除以 RMS 值。 $$\hat{x_i} = \frac{x_i}{RMS(x)}$$​​
4. **应用可学习的缩放因子 (Gain):** 乘以可学习的参数 $~\gamma_i$。注意，这里的 $\gamma$ 通常是一个与 $x$ 维度相同的向量，包含 n 个可学习的参数。 $$y_i = \gamma_i~\hat{x_i}$$​ 输出 $y=(y_1​,y_2​,...,y_n​)$ 就是 RMSNorm 层的最终结果。



## 5. RMSNorm 的优势

- **计算效率高:** 省去了计算均值的步骤，并且没有偏置参数 β，计算量更小。根据原论文，RMSNorm 比 LayerNorm 在 GPU 上快 7% 到 64% (取决于具体实现和硬件)。
- **参数量少:** 每个 RMSNorm 层比对应的 LayerNorm 层少了 n 个偏置参数 (β)。对于大型模型，这能节省一定的参数量。
- **性能相当:** 在许多任务和模型（尤其是 Transformer）上，RMSNorm 能够达到与 LayerNorm 相当甚至有时更好的性能。




## 6. 潜在考虑

- **缺乏中心化:** 虽然经验表明影响不大，但在某些特定场景下，强制激活值以 0 为中心可能是有益的，RMSNorm 放弃了这一点。
- **无偏置项:** 同样，虽然通常可行，但缺乏偏置项 β 可能会限制模型在某些情况下的表达能力。不过，由于后续通常会接线性层（本身带有偏置），这个问题的影响可能不大。




## 7. 应用

RMSNorm 已经被许多现代大型语言模型采用，因为它在保持性能的同时提高了效率。一些知名的例子包括：

- Meta 的 Llama 系列模型 (Llama, Llama 2, Llama 3)
- Google 的 PaLM 和 PaLM 2 (部分变体)
- T5 的一些变体
- 以及其他许多开源和闭源模型



## 总结:

RMSNorm 是对 Layer Normalization 的一种简化和优化，它通过移除均值中心化步骤和偏置参数 β，并使用输入的均方根 (RMS) 进行缩放，实现了更快的计算速度和更少的参数量，同时在许多现代深度学习模型（尤其是 Transformer）中保持了与 LayerNorm 相当的性能。这使得它成为大型模型设计中一个有吸引力的选择。