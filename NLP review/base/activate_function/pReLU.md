# Parametric ReLU（PReLU）函数

**PReLU函数** 是一种常用的激活函数，它在ReLU的基础上引入了可学习的参数α，使得函数在x≤0的部分不再是恒等于0，而是具有一个小的负斜率。这有助于缓解ReLU函数在负半轴上的“死神经元”问题，提高神经网络的学习能力

---

<br>

## 1. PReLU的公式

PReLU 也是 ReLU 的改进版本：


$$\text{PReLU(x)}=\begin{cases}\quad\mathrm{x}&,\mathrm{x>0}\\\alpha_{\mathrm{i}}\mathrm{x}&,\mathrm{x\leq0}&&\end{cases}$$

如下图所示：

![pReLU](/NLP%20review/assets/prelu.png)



<br>
<br>


PReLU函数中，参数$\alpha$通常为0到1之间的数字，**并且通常相对较小**。

- 如果 $\alpha_i=0$ ，则PReLU(x)变为 ReLU。
- 如果 $\alpha_i>0$，则PReLU(x)变为Leaky ReLU。
- 如果 $\alpha_i$ 是可学习的参数，则PReLU(x)为PReLU函数。

<br>

PReLU函数的特点：

- 在负值域，PReLU的斜率较小，这也可以避免**Dead ReLU问题**(*也就是<0部分的神经元总是输出0，导致梯度为0，无法更新参数*)。
- 与ELU相比，PReLU 在负值域是线性运算。尽管斜率很小，但不会趋于0。

