# Distributed Data Parallel

## 1. 简介

DistributedDataParallel（DDP）是PyTorch中的一种并行训练方法，它可以将模型和数据分布在多个GPU或多个机器上，以加速训练过程。DDP通过在多个进程之间同步模型参数和梯度，从而实现并行训练。(**真正的分布式并行**)


<br>
<br>


## 2. 原理

训练流程：
- step1: 使用多个进程，每个进程都加载数据和模型
- step2: 各进程同时进行前向传播，得到输出
- step3: 各进程分别计算loss，反向传播，计算梯度
- step4: **各进程间通信**，将梯度在各卡同步
- step5: 各进程分别更新模型


详细如下图:

![distributed data parallel](/NLP%20review/assets/distributeddataparallel.png)



<br>
<br>


## 3. 分布式训练中的基本概念

- group: 进程组，一个分布式任务对应一个进程组，一般就是所有卡都在一个组里

- world_size: 全局的并行数，一般情况下等于总的卡数

- node: 节点，可以是一台机器，或者一个容器，节点内包含多个GPU

- rank (global_rank): 整个分布式训练任务内的进程 序号

- local_rank: 每个 node 内部的相对进程序号

- backend: 通信后端，如nccl、gloo等

- master_addr: master节点的ip地址

- master_port: master节点的端口号

<br>
<br>


## 分布式训练中的通信

- 概念：在分布式模型训练中，通信是不同节点之间进行信息交换以及协调训练任务的关键组成部分

- 通信类型:
  - 1. 点对点通信：将数据从一个进程传输到另一个进程称为点对点通信
  - 2. **集合通信**：一个分组中所有进程的通信模式称之为 集合通信
    - 6 种通信类型：Broadcast、Reduce、Gather、Scatter、All Reduce、All Gather

> 集合通信可以参考 [集合通信](https://scale-py.godaai.org/ch-mpi/collective.html)

- 通信后端：
  - NCCL：NVIDIA Collective Communications Library，是NVIDIA提供的用于GPU之间通信的库，支持集合通信
  - Gloo：是PyTorch中默认的通信后端，支持集合通信
  - MPI：Message Passing Interface，是用于并行计算的标准通信协议，支持集合通信

<br>
<br>



## 在 `transformers` 库使用 DDP 和 DP

在 `transformers` 库中，无论是 `DP` 还是 `DDP` 使用的代码是一致的, 那么程序要怎么区分到底是 ``DP`` 还是 `DDP` 呢?

在于启动方式的不一样，使用  命令 `torchrun --nproc_per_node=2 ddp.py` 即可使用 `DDP`, 默认用 `python dp.py` 启动的文件是 `DP`