# LoRA 涉及到的一些数学知识等

用LoRA fine-tuning 能够用少量参数媲美全参数的效果。（**非常重要的一个fine-tuned技术**）

---


### LoRA核心，rank decomposition matrices

==（这是LoRA为什么可以计算量小的重要原因）==

矩阵的秩分解（Rank Decomposition of Matrices），也称为矩阵的秩因子分解，**是指将一个矩阵分解为几个低秩矩阵的乘积的过程**。这种分解在数据压缩、降维和优化问题等方面非常有用。**最常见的秩分解形式是低秩近似，即通过几个秩很小的矩阵的乘积来近似原始矩阵，这有助于减少计算复杂度和存储需求**。
秩分解的方法有多种，包括：
1. 奇异值分解（Singular Value Decomposition, SVD）：这是最常用的秩分解方法。通过SVD，一个\(m \times n\)的矩阵\(A\)可以被分解为三个矩阵\(U\)、\(S\)和\(V\)的乘积，其中\(U\)和\(V\)是正交矩阵，\(S\)是对角矩阵。分解形式如下：
   \[ A = U \cdot S \cdot V^T \]
   通过保留\(S\)中对角线上最大的\(k\)个奇异值（\(k\)是期望的秩），我们可以得到\(A\)的一个低秩近似。
2. 主成分分析（Principal Component Analysis, PCA）：PCA是一种统计技术，也可以用于秩分解。当矩阵\(A\)表示一个高维特征的数据集时，PCA特别有用。PCA寻找数据变化最大的方向（主成分），并将数据投影到这些方向上，从而得到一个低维表示。
3. 非负矩阵分解（Non-negative Matrix Factorization, NMF）：NMF是一种将非负矩阵分解为两个非负矩阵乘积的方法。当处理自然是非负的数据时，如图像数据或文本数据，这种分解非常有用。
4. 张量分解：对于作为高阶张量一部分的矩阵，可以使用张量分解方法，如典范多项式分解（Canonical Polyadic Decomposition, CP）或Tucker分解，将张量分解为若干个秩-1张量的和，这实际上是将矩阵分解为若干个秩-1矩阵的和。
这些方法具有不同的性质，适用于不同类型的数据和应用。选择哪种方法取决于具体问题的需求，如是否需要非负性、可解释性或计算效率。


<br>
<br>
<br>

### 秩分解和特征分解的区别
==（也就是说，rank decomposition 跟 常见的特征分解不是一个东西！线代基础）==

**将矩阵分解为特征值和特征矩阵通常被称为特征分解**（Eigenvalue Decomposition），**而不是秩分解**（Rank Decomposition）。特征分解是一种特定的矩阵分解方法，它将一个方阵（即行数和列数相等的矩阵）分解为特征值和特征向量的乘积。
对于一个方阵 \( A \)，如果它有 \( n \) 个线性无关的特征向量 \( v_1, v_2, ..., v_n \) 和对应的特征值 \( \lambda_1, \lambda_2, ..., \lambda_n \)，那么特征分解可以表示为：
\[ A = V \Lambda V^{-1} \]
其中，\( V \) 是由特征向量构成的矩阵，\( \Lambda \) 是对角矩阵，其对角线上的元素是特征值 \( \lambda_1, \lambda_2, ..., \lambda_n \)。

**特征分解的主要目的是为了简化矩阵的运算**，例如计算矩阵的幂、求解线性微分方程等。特征分解揭示了矩阵的内在结构，比如矩阵的秩等于其非零特征值的个数。

虽然特征分解和秩分解都是矩阵分解的方法，但它们的目的和应用场景不同。**秩分解**通常用于降低矩阵的维度或近似矩阵，而**特征分解**用于分析矩阵的性质和结构。在秩分解中，我们通常关注的是如何用低秩的矩阵来近似原始矩阵，而在特征分解中，我们关注的是矩阵的特征值和特征向量。


<br>
<br>
<br>



### 又涉及到主成分分析（PCA）和奇异值分解（SVD）
