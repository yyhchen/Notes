# 推理优化相关

[原文地址](https://zhuanlan.zhihu.com/p/698308542)

---

<img src='/NLP review/assets/kacacahe_compression.png'>


LLM 推理服务性能最关键的因素： `KVCache 显存大小`

KVCache显存占公式:

$$\begin{aligned}KVCacheMemroyCostPerToken&=2\times L\times H\times N\times D&(1)\\KVCacheMemroyTotalCost&=T\times KVCacheMemroyCostPerToken&(2)\end{aligned}$$


其中：

- $L$：模型层数（Layers）
- $H$：模型隐藏层大小（HeadDim）
- $N$：模型头数（NumHeads）
- $D$：模型维度（DataType）
- $T$：推理时序列长度(Tokens数)


在长文本推理服务场景中，我们的目标是在给定显存空间，推理服务要处理的 Token 数尽可能的多（即 $T$ 尽可能大），根据上述公式，可知 提升 $T$ 有两种策略:

1. 对 $T$ 进行维度压缩，用较少的 token 信息表示全量 token 信息
2. 对 $L$、$H$、$N$、$D$ 进行维度压缩，降低每个 token 的显存开销

<br>
<br>

#### 优化策略
1. **压缩KVCache长度**
   - 静态Token稀疏化：通过算法设计找出重要Token

<img src='/NLP review/assets/streamingllm.png'>

   - 动态Token稀疏化：维护相关性高的历史Token集合

<img src='/NLP review/assets/eviction_algorithm.png'>

   - Prefix Caching工程优化：复用Tokens的KVCache

<br>

2. **压缩每个Token的KVCache**
   - 通过调整Layers、HeadDim、NumHeads、HiddenSize和DataType来降低显存开销

<br>
<br>

#### 详细策略解析

1. **压缩KVCache长度**
   - **StreamingLLM**：保留序列初始Tokens，减少显存占用
   - **H2O**：动态Token稀疏化，使用贪心算法维护历史Token集合
   - **Prefix Caching**：提升Prefill效率，节省显存

2. **压缩每个Token的KVCache**
   - **LayerSkip**：通过“早退”策略优化推理性能
   - **YOCO**：多层共享KVCache，减少显存占用
   - **MQA、GQA、MLA**：通过不同方式压缩KVCache头维度或HiddenSize维度
   - **低比特量化**：降低KVCache的精度，减少显存占用

#### 总结
- 大部分工作聚焦于算法及模型结构优化
- 工程优化针对特定场景
- 超线性优化需算法和硬件突破
