# AWQ (Activation-aware Weight Quantization)

激活感知权重量化

- [论文](https://arxiv.org/abs/2306.00978)
- [huggingface AWQ](https://huggingface.co/docs/transformers/v4.36.1/zh/main_classes/quantization)


--- 

<br>
<br>


## 📝 论文总结

本文提出了一种名为 AWQ 的新型权重量化方法，旨在降低大型语言模型 (LLM) 的内存占用和推理成本。**AWQ 的核心思想是识别并保护模型中最重要的权重，从而在保证性能的前提下减少模型大小**。与现有方法相比，AWQ 在各种任务和模型上都取得了更好的量化性能，并且对指令调整的 LLM 和多模态 LLM 也表现出色。此外，作者还实现了一个高效的推理框架，将 AWQ 的内存节省转化为实际的速度提升。

<br>

下图展示了 `AWQ` 是怎么工作的，只取 1% 的 **"显著" 权重**，即可降低 模型的 PPL，但是呢混合精度会使的硬件效率变低，`AWQ` 就通过 "缩放" 每个通道 来保护 **"显著" 权重**的重要性 

<img src='/NLP review/assets/awq.png'>

<br>
<br>


## 🌟 重要亮点

* **激活感知权重保护**： AWQ 通过观察激活分布来识别并保护最重要的权重，而不是直接量化所有权重。
* **每通道缩放**： AWQ 使用每通道缩放来减少显着权重的量化误差，从而提高量化模型的性能。
* **无需反向传播或重建**： AWQ 不依赖于反向传播或重建过程，因此可以更好地保留 LLM 的泛化能力。
* **适用于指令调整和多模态 LLM**： AWQ 不仅可以应用于标准 LLM，还可以应用于指令调整的 LLM 和多模态 LLM。
* **高效推理框架**： 作者实现了一个高效的推理框架，将 AWQ 的内存节省转化为实际的速度提升，在桌面和移动 GPU 上实现了 3.2-3.3 倍的加速。
* **极低比特量化**： AWQ 还可以应用于极低比特量化，例如 INT2，进一步降低模型大小并提高推理效率。


<br>
<br>


## 🔨 使用方法

结合 [AutoAWQ](https://github.com/casper-hansen/AutoAWQ) 使用。

<br>

**详细可参考:**

- [huggingface AWQ](https://huggingface.co/docs/transformers/v4.36.1/zh/main_classes/quantization)
- [Qwen2 AWQ](https://qwen.readthedocs.io/zh-cn/latest/quantization/awq.html)