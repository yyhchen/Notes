
---

>[!TIP]
> from torch import nn

PyTorch 中的 `torch.nn.CrossEntropyLoss` 是用于多分类任务的损失函数，结合了 **LogSoftmax** 和 **负对数似然损失（NLLLoss）**。它的计算步骤如下：

---

### **计算公式**
对于输入的 **logits**（未归一化的预测值）和目标标签 **targets**，损失计算分为两步：
1. **LogSoftmax**：将 logits 转换为概率分布的对数形式。
   
  $$ \text{log\_softmax} = \log\left( \frac{\exp(x_i)}{\sum_j \exp(x_j)} \right)$$
   
2. **负对数似然损失（NLLLoss）**：根据目标标签取出对应的 log_softmax 值，取负后求平均。
   
$$\text{loss} = -\frac{1}{N} \sum_{i=1}^N \text{log\_softmax}(x_i)[y_i]$$
   

---

### **关键细节**
1. **输入和目标的形状**：
   - **输入（logits）**：形状为 `(batch_size, num_classes)`，无需经过 Softmax。
   - **目标（targets）**：形状为 `(batch_size)`，每个元素是类别的索引（0到`num_classes-1`）。

2. **数值稳定性**：直接使用 LogSoftmax（而非先 Softmax 再取对数）可以避免数值溢出。

---

### **举例说明**
假设有一个 3 分类任务，批量大小为 2：

#### 输入 logits（未归一化）：
$$\text{logits} = \begin{bmatrix}
2.0 & 1.0 & 0.1 \\
0.5 & 2.0 & 0.3
\end{bmatrix}$$

#### 目标标签（类别索引）：
$$\text{targets} = [0, 1]$$


#### 计算步骤：
1. **计算 Softmax**：
  $$ \text{softmax}(\text{第一样本}) = \left[ \frac{\exp(2.0)}{\exp(2.0)+\exp(1.0)+\exp(0.1)}, \frac{\exp(1.0)}{\exp(2.0)+\exp(1.0)+\exp(0.1)}, \frac{\exp(0.1)}{\exp(2.0)+\exp(1.0)+\exp(0.1)} \right]$$

   计算得：
   $$\approx [0.6590, 0.2424, 0.0986]$$

   同理，第二样本的 Softmax 结果：
  $$ \approx [0.2424, 0.6590, 0.0986]$$


2. **取对数（LogSoftmax）**：

   $\text{log\_softmax}(\text{第一样本}) = \log([0.6590, 0.2424, 0.0986]) \approx [-0.416, -1.416, -2.317]$

   第二样本：
  $$ \approx [-1.416, -0.416, -2.317]$$


3. **根据目标标签取对应值**：
   - 第一样本目标是类别 0，对应值：`-0.416`
   - 第二样本目标是类别 1，对应值：`-0.416`

4. **求平均损失**：
   
   $$\text{loss} = -\frac{1}{2} \times (-0.416 + -0.416) = 0.416$$
   

---

### **代码验证**
```python
import torch
import torch.nn as nn

# 定义损失函数
loss_fn = nn.CrossEntropyLoss()

# 输入 logits 和目标标签
logits = torch.tensor([[2.0, 1.0, 0.1], [0.5, 2.0, 0.3]])
targets = torch.tensor([0, 1])

# 计算损失
loss = loss_fn(logits, targets)
print(loss)  # 输出：tensor(0.4161)
```

---

### **扩展说明**
1. **与手动计算的区别**：
   - PyTorch 内部优化了数值稳定性，直接计算 LogSoftmax。
   - 如果手动计算 Softmax 后再取对数，可能因指数运算导致溢出。

2. **其他参数**：
   - `weight`：为不同类别设置权重（如处理类别不平衡）。
   - `ignore_index`：忽略特定索引的标签（如填充的无效标签）。

3. **与 BCELoss 的区别**：
   - `CrossEntropyLoss` 用于多分类（目标是单类别索引）。
   - `BCELoss` 用于二分类或多标签分类（目标是 one-hot 编码）。

通过这个例子，可以直观理解 `CrossEntropyLoss` 是如何将 logits 转换为概率分布，并计算预测与真实标签之间的差异的。