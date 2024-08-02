# What's GGUF

- [GPT-Generated Unified Format](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)
- [huggingface intro gguf](https://huggingface.co/docs/hub/gguf)
- [useage with llama.cpp](https://huggingface.co/docs/hub/gguf-llamacpp)

---

<br>

## GGUF

GGUF就是一种二进制格式文件的规范，原始的大模型预训练结果经过转换后变成GGUF格式**可以更快地被载入使用，也会消耗更低的资源**。

**原因在于** GGUF采用了多种技术来保存大模型预训练结果，包括采用紧凑的二进制编码格式、优化的数据结构、内存映射等

下图展示了 `.gguf` 的详细结构信息:

<img src='/NLP review/assets/gguf.png'>


<br>
<br>


## Compare safetensor
虽然safetensors也被很多厂商采用，但是在大模型高效序列化、数据压缩、量化等方面不足。**而且它只保存了张量数据，没有任何关于模型的元数据信息**。而GGUF格式因为良好的设计，通过各种优化手段实现了快速的模型加载，这对于需要频繁载入不同模型的场景尤为重要，同时将模型的元数据信息压缩到二进制文件中，对于跨平台操作和传输都非常有价值.

