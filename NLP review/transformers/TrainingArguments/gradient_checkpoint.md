# gradient_checkpoint

---


> 使用 TrainingArguments加入 `gradient_checkpoint=True`参数，需要先使用 `model.enable_input_require_grads()`



## 原因

在`transformers`库中，使用`TrainingArguments`加入`gradient_checkpoint=True`参数时，需要先调用`model.enable_input_require_grads()`的原因与梯度检查点（gradient checkpointing）机制的工作原理有关。

梯度检查点是一种优化技术，**用于减少深度学习模型训练过程中的内存消耗。它通过在反向传播过程中仅存储部分中间梯度，而不是存储整个网络的所有中间激活值**。这样，就可以在不牺牲模型性能的情况下，减少GPU内存的使用。

然而，梯度检查点要求模型的输入张量（tensors）是可微分的（即它们的`requires_grad`属性设置为`True`），因为在反向传播过程中需要计算这些输入相对于损失函数的梯度。在某些情况下，模型的输入可能默认不会跟踪梯度，特别是当使用某些特定的数据加载或预处理方式时。

以下是详细步骤和原因：

1. **调用`enable_input_require_grads()`**：此方法确保模型的所有输入张量都会跟踪梯度。这是使用梯度检查点所必需的，因为梯度检查点需要在反向传播过程中计算和存储输入相对于损失的梯度。

2. **设置`TrainingArguments`中的`gradient_checkpoint=True`**：此参数告诉训练循环使用梯度检查点技术。当设置为`True`时，模型在前向传播过程中会保存必要的中间状态，以便在反向传播时重新计算丢失的梯度。

3. **训练循环中的配合**：在使用梯度检查点时，训练循环需要特别处理。在PyTorch中，这通常意味着使用`torch.utils.checkpoint`模块，它可以自动管理梯度检查点的保存和重计算。

4. **内存优化**：通过结合使用`enable_input_require_grads()`和`gradient_checkpoint=True`，可以在保持模型性能的同时，显著减少训练过程中的内存占用。

总结来说，`enable_input_require_grads()`确保输入张量可以参与梯度计算，而`gradient_checkpoint=True`则激活了梯度检查点机制，两者结合使用可以在训练大型模型时节省大量内存。
