# GPTQ

- [论文](https://arxiv.org/abs/2210.17323)
- [huggingface GPTQ](https://huggingface.co/docs/transformers/main/en/quantization/gptq)

---

<br>
<br>


## 📝 论文总结
通常，量化方法可以分为以下两类：

**训练后量化 (Post Training Quantization, PTQ)：** 适度地使用一些资源来量化预训练好的模型，如一个校准数据集和几小时的算力。

**量化感知训练 (Quantization Aware Training, QAT)：** 在训练或进一步微调之前执行量化。

GPTQ 属于**训练后量化**，这对于大模型而言格外有趣且有意义，因为对其进行全参数训练以及甚至仅仅是微调都十分昂贵。

<br>

具体而言，GPTQ 采用 int4/fp16 (W4A16) 的**混合量化方案**，**其中模型权重被量化为 int4 数值类型，而激活值则保留在 float16**。在推理阶段，**模型权重被动态地反量化回 float16** 并在该数值类型下进行实际的运算。

<br>

**该方案有以下两方面的优点：**

- int4 量化能够**节省接近4倍的内存**，这是因为反量化操作发生在算子的计算单元附近，而不是在 GPU 的全局内存中。

- 由于用于权重的位宽较低，因此可以**节省数据通信的时间**，从而潜在地提升了推理速度。

<br>

**GPTQ 论文解决了分层压缩的问题：**

给定一个拥有权重矩阵 $W_l$ 和输入 $X_l$ 的网络层 $l$，我们期望获得一个量化版本的权重矩阵 $\hat{W_l}$ 以最小化均方误差 (MSE)： ${\hat{W_l}}^*=argmin_{\hat{W_l}}\|W_lX-\hat{W_l}X\|_2^2$。**简单的说，也就是要找到一个量化后的矩阵能够逼近原来的权重矩阵为目标进行loss优化**

一旦每层都实现了上述目标，就可以通过组合各网络层量化结果的方式来获得一个完整的量化模型。

<br>

为解决这一分层压缩问题，论文作者采用了最优脑量化 (Optimal Brain Quantization, OBQ) 框架 ([Frantar et al 2022](https://arxiv.org/abs/2208.11580)) 。OBQ 方法的出发点在于其观察到：以上等式可以改写成权重矩阵 $W_l$ ​每一行的平方误差之和 $\sum_{i=0}^{d_{row}}\|W_{l[i,:]}X-\hat{W}_{l[i,:]}X\|_2^2$

<br>

这意味着我们可以独立地对每一行执行量化。即所谓的 **per-channel quantization**。对每一行 $W_{l[i,:]}$，OBQ 在每一时刻只量化一个权重，同时更新所有未被量化的权重，以补偿量化单个权重所带来的误差。所选权重的更新采用一个闭环公式，并利用了海森矩阵 (Hessian Matrices)。

<br>

GPTQ 论文通过引入一系列优化措施来改进上述量化框架，在降低量化算法复杂度的同时保留了模型的精度。

相较于 OBQ，GPTQ 的量化步骤本身也更快：OBQ 需要花费 2 个 GPU 时来完成 BERT 模型 (336M) 的量化，而使用 GPTQ，量化一个 Bloom 模型 (176B) 则只需不到 4 个 GPU 时。


<br>
<br>


## 🌟 重要亮点

*   **高精度压缩**： GPTQ 可以将大型语言模型压缩到每个参数 3 或 4 位，而困惑度损失可以忽略不计。
*   **高效性**： GPTQ 的运行时间仅为几个 GPU 小时，远远低于其他训练后量化方法。
*   **可扩展性**： GPTQ 可以扩展到具有数千亿个参数的模型，而不会降低性能。
*   **生成推理加速**： GPTQ 可以将大型语言模型在单个 GPU 上运行，并实现比 FP16 实现快 3.25 倍的生成推理速度。
*   **极端量化**： GPTQ 可以将模型量化为 2 位或三元值，在极端量化条件下仍能保持合理的精度。