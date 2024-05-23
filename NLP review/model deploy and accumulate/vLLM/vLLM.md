# vLLM


---


vLLM**是一个用于大型语言模型（LLM）的高吞吐量和内存高效推理及服务引擎**。它主要采用了**PagedAttention技术**，这种技术可以有效管理注意力机制的键值对（keys和values），显著提高吞吐量并优化显存利用率。vLLM能够支持多种流行的Hugging Face模型，如GPT-J、GPT-NeoX、LLaMA等。
PagedAttention技术的**核心在于改进了传统LLM推理中的显存管理**。在自回归模型中，keys和values通常被称为KV缓存，这些数据存储在GPU的显存中，用于生成下一个token。由于这些KV缓存非常大且大小动态变化，传统的系统由于显存碎片和过度预留，会导致60%-80%的显存浪费。**PagedAttention受到操作系统中虚拟内存和分页技术的启发，允许在不连续的内存空间中存储连续的keys和values，从而更灵活地管理这些KV缓存**。这种技术使得显存利用接近理论上的最优值，浪费比例低于4%，并通过更好的显存管理，使得单次可以使用更大的batch size，进一步利用GPU的并行计算能力。

此外，vLLM还**支持内存共享**，这是PagedAttention的另一个关键特性。当使用单个提示产生多个不同的序列时，可以通过将不同序列的逻辑块映射到同一个物理块来实现显存共享。这种内存共享机制可以显著降低复杂采样算法对显存的需求，从而提高吞吐量。

vLLM的使用非常灵活和简便，支持与流行Hugging Face模型的无缝集成，支持多种解码算法，包括并行采样、束搜索等，还支持张量并行主义以实现分布式推理。此外，vLLM还支持流式输出和与OpenAI兼容的API服务器，同时支持NVIDIA和AMD GPUs。


- [大模型推理加速工具：vLLM](https://zhuanlan.zhihu.com/p/642802585)
- [vLLM doc](https://docs.vllm.ai/en/latest/index.html)
- [vLLM github](https://github.com/vllm-project/vllm?tab=readme-ov-file)