# Leaky ReLU函数

---

### 什么是Leaky ReLU？

Leaky ReLU（Leaky Rectified Linear Unit，带泄漏的ReLU）是ReLU函数的一个变种。ReLU函数在输入小于0时输出为0，这可能会导致“神经元死亡”问题，即一些神经元可能永远不会被激活。Leaky ReLU通过给小于0的输入一个小的负斜率来解决这个问题。

<br>

### Leaky ReLU的数学表达式
$$\text{LeakyReLU}\left(\mathrm{x}\right)=\begin{cases} \mathrm{x}&\text{, x > 0}\\ \mathrm{\alpha x}&\text{, x 0}\end{cases}$$


其中，α是一个很小的正数，通常设置为0.01左右。

<br>

### Leaky ReLU的优点

* **缓解“神经元死亡”问题**：通过给负输入一个小的负斜率，Leaky ReLU使得神经元在负输入时仍然有一定的输出，从而避免了“神经元死亡”问题。
* **增强模型的表达能力**：Leaky ReLU的非线性特性比ReLU更强，可以使模型更好地拟合复杂的数据。
* **加速收敛**：Leaky ReLU可以加速模型的收敛速度。

<br>

### Leaky ReLU的缺点

* **α值的选取**：α值的选择对模型的性能有一定的影响，需要通过实验来确定最佳的值。

<br>

### Leaky ReLU与ReLU的比较

| 特性        | ReLU                                    | Leaky ReLU                                |
| ----------- | -------------------------------------- | ---------------------------------------- |
| 负区间输出 | 0                                       | αx                                        |
| 连续性      | 不连续                                    | 连续                                      |
| 可导性      | 在x=0处不可导                            | 在x=0处可导                              |
| 梯度消失问题 | 容易出现梯度消失                        | 缓解梯度消失                            |


<br>

<br>

### 结论

Leaky ReLU是一种有效的激活函数，它在一定程度上克服了ReLU函数的缺点。在实际应用中，Leaky ReLU可以作为ReLU函数的一种替代，尤其是在需要更强的表达能力和更好的鲁棒性时。

