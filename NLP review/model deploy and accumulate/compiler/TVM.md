# TVM related

---

TVM（Tensor Virtual Machine）是一个开源的机器学习编译器框架，主要用于CPU、GPU和机器学习加速器。它旨在让机器学习工程师能够在任何硬件后端优化和高效运行计算。

在模型部署中，TVM扮演着重要的角色。它允许开发者使用任意的训练框架（如TensorFlow、Pytorch、mxnet）来训练模型，然后通过TVM的转换，这些模型可以被部署到任何一个目标平台（如GPU、Arm、Android CPU、Android GPU、IOS等）上运行推理（Inference）。TVM在这个过程中帮助优化网络的运行效率。

TVM堆栈分为两个主要组件：TVM编译器（TVM compiler）和TVM运行时（TVM runtime）。TVM编译器负责模型的编译和优化，而TVM运行时则在目标设备上运行。集成编译后的模块不需要在目标设备上构建整个TVM，只需在电脑上构建TVM编译器堆栈，然后用来交叉编译要部署到目标设备上的模块。

**TVM的工作流程包括以下几个步骤：**
1. 从Tensorflow、PyTorch或Onnx等框架导入模型。
2. 将模型翻译成Relay，这是TVM的高级模型语言。
3. 将Relay表示降低到张量表达式（Tensor Expression，TE），用于描述张量计算的专属域语言。
4. 使用AutoTVM或AutoScheduler模块搜索最佳调度（schedule）。
5. 选择最佳配置进行模型编译。
6. 将模型降低到张量级的中间表示（TIR）。
7. 编译成机器码。

这个过程最终生成可以部署到生产中的优化模型。TVM支持多种不同的编译器后端，包括LLVM、NVCC（NVIDIA的编译器）等。
总的来说，TVM在模型部署中的作用是优化和转换模型，使其能够在各种硬件平台上高效运行。这对于需要在多种不同设备上部署深度学习模型的应用来说尤为重要。


