# PyTorch 中的 DataParallel类

---


## 原理

训练流程:

- step1: GPU0 加载 model 和 batch 数据
- step2: 将 batch 数据从 GPU0 **均匀拷贝**到 各个卡上 
- step3: 将 model 从GPU0 **拷贝**到各个卡上
- step4: 各卡**同时**进行前向传播
- step5: GPU0 收集各卡上的输出，并计算loss
- step6: 将 Loss 分发至各卡，进行反向传播，计算梯度
- step7: GPU0 收集各卡上的梯度，并进行汇总
- step8: GPU0 更新模型(参数)

![data parallel](/NLP%20review/assets/dataparallel.png)


<br>
<br>


### 细节

- batch_size 是平分到不同的卡上，比如 2 张卡，batch_size=32，那么每张卡上都是 16 个样本。

**注意：** 每张卡上的数据是**独立**的，所以每张卡上的梯度也是**独立**的，最后汇总梯度的时候，是**相加**而不是相乘。



<br>
<br>



### `nn.DataParallel` 存在的问题

- 实际效果：
  - 虽然是多 GPU 训练，但是训练速度提升并没有很大

- 存在问题：
  - 单进程，多线程，由于 GIL锁的问题，不能充分发挥多卡的优势
  - 由于 `DataParallel` 的训练策略问题，会存在一个主节点占用显存比其他节点高很多的情况
  - 效率较低，每次训练开始都要重新同步模型，大模型的同步时间会较难接受
  - 只适合单机训练，无法支持真正的分布式多节点训练


<br>
<br>


### 不适合训练，适合推理

