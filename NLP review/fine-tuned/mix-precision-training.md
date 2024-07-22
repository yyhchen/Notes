# 混合精度训练

---

混合精度训练，顾名思义，就是将单精度（FP32）和半精度（FP16）混合起来，用于训练神经网络。混合精度训练可以带来以下好处：

- **减少显存占用**：FP16的显存占用只有FP32的一半，因此可以训练更大的模型或更大的batch size。
- **加速训练**：FP16的计算速度比FP32快，因此可以加速训练过程。

半精度与单精度如图所示：


<img src='/NLP review/assets/float16.png' style="width: auto; height: 100px;">
<img src='/NLP review/assets/float32.png' style="width: auto; height: 70px;">

<br>

**数据表示分为了三个部分：**
1. 最高位表示符号位；
2. 有5位表示exponent位;
3. 有10位表示fraction位;


**计算方法：**

如果 Exponent 位全部为0：

- 如果 Fraction 位全部为0，表示 0；
- 如果 Fraction 位不全为0，表示一个非零的小数；$
(-1)^{sign} \times 2^{exponent-14} \times （0+\frac{fraction}{1024}）
$

如果 Exponent 位全为1：

- 如果 Fraction 位全部为0，表示无穷大；
- 如果 Fraction 位不全为0，表示NaN（Not a Number）；

<br>

**其他情况公式如下：**

$$
FP16 = (-1)^{sign} \times 2^{exponent-15} \times fraction
$$

<br>

## 混合精度训练的原理

混合精度训练的原理是将模型中的部分参数和中间结果用FP16表示，而将其他部分用FP32表示。这样做的好处是可以减少显存占用，同时加速计算。

具体来说，混合精度训练的步骤如下：

1. 将模型中的部分参数和中间结果用FP16表示；
2. 将其他部分用FP32表示；
