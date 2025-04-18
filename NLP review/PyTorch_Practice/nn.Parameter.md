
---

>[!NOTE]
>在 PyTorch中，设计在模型中需要更新的参数（即，可学习参数）必须用 `nn.Parameter` 封装
 

在PyTorch中，`nn.Parameter` 是一个非常重要的类，用于将张量（Tensor）包装为模型的**可学习参数**。它继承自 `torch.Tensor`，并赋予了张量作为模型参数的特定功能。以下是对 `nn.Parameter` 的详细介绍：



### **1. 核心作用**
- **自动梯度计算**：`nn.Parameter` 默认启用梯度计算（`requires_grad=True`），优化器在反向传播时会自动更新其值。
- **参数注册**：当被赋值给 `nn.Module` 的属性时，`nn.Parameter` 会被自动注册到模型的参数列表中（可通过 `model.parameters()` 获取），而普通张量不会。
- **设备迁移支持**：与模型一起移动参数到GPU/CPU时，无需手动操作。



### **2. 与普通张量的区别**
| 特性                | `nn.Parameter`                  | 普通张量 (`torch.Tensor`)        |
|---------------------|---------------------------------|---------------------------------|
| 梯度计算             | 默认 `requires_grad=True`       | 默认 `requires_grad=False`      |
| 参数注册             | 自动添加到 `model.parameters()` | 需手动注册或作为缓冲区（`register_buffer`） |
| 优化器更新           | 是                              | 否（除非显式设置）               |
>可以尝试 `print` 一个 `nn.Parameter(torch.ones(2,2))` 就可以看到 打印出来的 `tensor` 会包含 `requires_grad=True` 的


### **3. 使用示例**
#### **自定义一个简单的线性层**
```python
import torch
import torch.nn as nn

class CustomLinear(nn.Module):
    def __init__(self, in_features, out_features):
        super().__init__()
        # 定义可学习的权重和偏置参数
        self.weight = nn.Parameter(torch.randn(in_features, out_features))
        self.bias = nn.Parameter(torch.randn(out_features))
    
    def forward(self, x):
        return x @ self.weight + self.bias

# 使用示例
model = CustomLinear(3, 5)
input = torch.randn(2, 3)
output = model(input)
print(list(model.parameters()))  # 输出权重和偏置参数
```



### **4. 关键注意事项**
- **必须包装为 `nn.Parameter`**：如果直接用 `torch.Tensor` 初始化参数，它们不会被注册到模型，导致训练时无法更新。
- **动态参数管理**：对于多个参数，可使用 `nn.ParameterList` 或 `nn.ParameterDict` 进行管理。
- **参数初始化**：通常结合 `torch.init` 模块（如 `init.xavier_normal_`）进行初始化。



### **5. 常见错误**
- **忘记包装**：未用 `nn.Parameter` 包裹张量，导致参数未被优化器识别。
- **误用普通张量**：在模块中使用普通张量参与计算但未注册，导致无法更新。



### **总结**
`nn.Parameter` 是PyTorch中定义模型可学习参数的核心工具。通过将张量包装为 `nn.Parameter`，开发者可以确保参数被自动注册、梯度计算和优化器更新，从而简化了模型训练流程。在自定义层或复杂模型时，正确使用 `nn.Parameter` 是保证模型可训练性的关键。