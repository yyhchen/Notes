# DistributedSampler

```python
from torch.utils.data.distributed import DistributedSampler
```

---

## 代码示例

```python
from torch.utils.data import DataLoader
from torch.utils.data.distributed import DistributedSampler

...

# 数据采样器 特殊设置
trainloader = DataLoader(trainset, batch_size=32, collate_fn=collate_func, sampler=DistributedSampler(trainset))

```


<br>
<br>

### `sampler=DistributedSampler(trainset)` 参数的作用

**在 PyTorch 的分布式训练中，`sampler=DistributedSampler(trainset)` 这个参数的作用是为每个 GPU 分配一个不重叠的数据子集，确保每个 GPU 在训练过程中处理的数据是唯一的。**

<br>
<br>

### 为什么需要 `DistributedSampler`？

* **数据分片：** 在多 GPU 训练时，我们需要将数据集分给不同的 GPU，每个 GPU 处理一部分数据。`DistributedSampler` 就负责将数据集均匀地分割成多个子集，每个子集对应一个 GPU。
* **避免数据重复：** 如果不使用 `DistributedSampler`，不同的 GPU 可能处理相同的数据，导致重复计算，降低训练效率。
* **保证训练一致性：** `DistributedSampler` 确保每个 GPU 在每个 epoch 处理的数据顺序是一致的，这对于模型的同步训练非常重要。

### `DistributedSampler` 的工作原理

1. **计算每个 GPU 的数据量：** 根据数据集大小和 GPU 数量，计算每个 GPU 应该处理的数据量。
2. **生成数据索引：** 为整个数据集生成一个索引序列。
3. **分发索引：** 根据每个 GPU 的排名，将索引序列分给不同的 GPU。每个 GPU 只负责处理分配给它的索引对应的样本。
4. **同步 epoch：** 确保所有 GPU 在同一个 epoch 处理相同的数据。

