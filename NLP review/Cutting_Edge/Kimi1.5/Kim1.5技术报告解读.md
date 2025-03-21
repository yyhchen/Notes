
**Kimi1.5 采取的训练流程**: 
```mermaid
graph LR
    A[预训练阶段] --> B[普通SFT]
    B --> C[Long-CoT SFT]
    C --> D[强化学习]
    D --> E[Long2short 优化]
    
    A:::pretrain
    B:::sft
    C:::longcot
    D:::rl
    E:::long2short
    
    classDef pretrain fill:#4CAF50,color:white
    classDef sft fill:#2196F3,color:white
    classDef longcot fill:#FF9800,color:white
    classDef rl fill:#9C27B0,color:white
    classDef long2short fill:#FF5722,color:white
    
    click A callback "预训练阶段：多模态数据训练基础能力"
    click B callback "普通SFT：基础指令对齐"
    click C callback "Long-CoT SFT：长文本推理能力强化"
    click D callback "强化学习：优化生成策略"
    click E callback "Long2short：长文本到短文本的转换训练"
```

---

>[!tip] 原论文
>[Kimi k1.5: Scaling Reinforcement Learning with LLMs](https://arxiv.org/abs/2501.12599)

## 1. 预训练阶段的一些细节

Kimi的预训练阶段包含==**三个阶段**==，结合多模态数据（文本、视觉、OCR等）进行分阶段训练:

1. **视觉-语言预训练阶段**  
   建立语言基础能力并逐步融合多模态数据，通过文本与视觉信息的联合训练提升跨模态理解能力

2. **冷启动阶段（冷却阶段）**  
   通过模型融合（Model Fusion）等技术优化模型性能，可能涉及对初始预训练模型的参数调整或架构改进

3. **长文本激活阶段**  
   在预训练后期引入长文本数据集（如扩展至128K上下文长度），增强模型对长文本的理解和生成能力。此阶段通过严格的数据质量控制，确保训练数据的相关性、多样性及平衡性


>[!NOTE] 冷却阶段(退火阶段)
>1. **能力巩固**
>	- 在初始的Vision-language预训练后，模型已具备基础的多模态理解能力。Cooldown阶段通过**精选数据（curated data）​**和**合成数据（synthetic data）​**，进一步强化模型在需要逻辑推理（如数学、代码）和知识处理（如事实性问答）任务上的表现。
>	- 例如，可能针对数学问题生成更多变体，或通过知识图谱增强事实性问答的训练数据。
>---
>
>2. **数据优化**
>	- **精选数据**：选择高质量、高难度的样本（如竞赛题目、专业领域问题），提升模型处理复杂问题的能力。
>	- **合成数据** ：利用规则或生成模型（如自动生成代码测试用例、数学证明步骤）创建针对性数据，填补真实数据中的不足，增强泛化性。
>---
>
>3. **过渡到长上下文训练**：
>	- 在进入**Long-context activation**​（支持128k tokens的上下文）前，cooldown阶段确保模型在**短上下文任务中足够鲁棒**。例如:
>		- 解决数学题时，模型需精准定位关键步骤，避免长上下文中的干扰。
>		- 处理知识问答时，快速检索相关信息，减少冗余推理
>---
>4. **训练策略调整**
>	- 可能降低学习率或调整优化器参数，避免预训练后的性能震荡
>	- 采用课程学习（Curriculum Learning），从简单任务逐步过渡到复杂任务，平衡模型的学习曲线

**对比其他阶段**：

- ​**预训练（Vision-language）​**：广泛学习语言和多模态关联（如图文匹配）。
- ​**Cooldown**：针对性强化推理和知识任务，类似“专项特训”。
- ​**Long-context activation**：扩展上下文容量，适配长文本/多模态输入（如整本书分析、复杂流程图解析）。
> 并且 long-context activation 这一步是渐进拓展的，4k -> 32k -> 128k (原文提到)



>[!warning] 需要注意的是！
>Cooldown 阶段可不是 SFT哦！(**而是针对性预训练**)
>
>在论文的附录B中有介绍到，这里的阶段还是 pre-train，因为训练目标任务仍然是 NTP (next token prediction)

| **维度**    | ​**Cooldown阶段**               | ​**监督微调（SFT）​**             |
| --------- | ----------------------------- | --------------------------- |
| ​**主要目标** | ​**巩固多模态能力**​（推理、知识、跨模态对齐）    | ​**适应特定任务**​（如对话、分类、生成）     |
| ​**数据性质** | 混合使用**精选数据（高难度任务）​**与**合成数据** | 主要依赖**人工标注的高质量任务数据**​（如问答对） |
| ​**训练范围** | 覆盖**多任务、多领域**​（如数学、代码、事实问答）   | 聚焦**单一任务或垂直领域**​（如客服对话）     |
| ​**优化策略** | 可能调整学习率、优化器，但**不显著改变模型架构**    | 可能修改损失函数或添加任务头（如分类器）        |



- **Cooldown**更接近**通用能力的预训练延伸**，而**SFT**是**任务导向的最终适配**。

- 如果cooldown阶段大量使用人工标注的监督数据，则可视为**广义的SFT**；但若以合成/弱监督数据为主，则与SFT有本质区别。

- ==实际论文中需结合具体实现判断==，但核心差异在于**是否以任务性能为直接优化目标**。

---

## 2. 普通SFT阶段的细节

>[!NOTE] 
>数据集大小：100w
>


**训练细节：**

1. epoch 1用的32k，后面的epoch全是128k （不用连续多几个epoch训练32吗？）
2. epoch 1学习率从2 × 10⁻⁵下降到2 × 10⁻⁶, 后面是重新预热到1 × 10⁻⁵，最后下降到1 × 10⁻⁶

> 这种策略被称为 **增加上下文长度的训练（Increasing Context Length Training）** 或者叫**课程学习（Curriculum Learning）





---


## 3. Long-CoT SFT 部分细节

>[!tip] 前提
>构建 高质量 强化学习prompt 对模型推理效果非常好, 可以避免奖励作弊（reward hacking）和过拟合
>
>1. **多样性覆盖**:
>	- **领域广泛性** ：涵盖STEM、编程、通用推理等多学科，支持跨领域适应性
>	- **数据来源** ：整合竞赛题目、文本和图文问答数据，通过自动过滤筛选需复杂推理且易验证的问题。
>	- **标签系统** ：按学科和领域分类，确保均衡分布（如引用文献中的分类方法）
>
>1. **难度平衡**(==重要==)
>	- **动态评估** ：利用SFT模型对提示的难度进行自适应评估, 通过率低代表难度越高
>	- **分级分布** ：混合简单、中等、困难问题，避免模型过度依赖特定难度。(类似课程学习)
>
>1. **可验证性**（==重要==）
>	- **排除易攻击问题** ：移除选择题、判断题、证明题等易通过猜测或错误推理蒙混过关的题型。
>	- **反奖励黑客策略**：要求模型在无推理链（CoT）的情况下尝试回答，若在N次尝试内猜中答案（N=8），则剔除该prompt （*说明这个问题很简单，不需要推理，不利于这种策略训练*）


**Long-CoT SFT的核心目标:**

通过构建高质量的“长链推理”（Long-CoT）数据集，提升模型生成逻辑连贯、细节丰富的推理过程的能力。

**怎么做**：  
1. **数据集设计**：  
   - 用筛选后的高质量问题，搭配**人工验证的完整推理路径**（类似“一步步解题的参考答案”）。  
   - 包含文本和图文混合的问题，覆盖多种推理类型（如数学、代码、常识）。  

2. **训练方法**：  
   - **轻量微调**：在小而精的数据集上训练模型，重点学习如何：  
     - **分步骤规划**（先想好怎么解题再动手）；  
     - **检查中间步骤**（及时发现错误）；  
     - **尝试不同解法**（灵活调整思路）。  

**效果**：模型回答更像人类思考——**逻辑连贯、细节丰富**，尤其在复杂任务（如长文本生成、多步骤推理）中表现更好。  


**一句话总结**：用“一步步教模型怎么想”的数据集，让它学会像人一样详细、严谨地解决问题。


---


## 4.强化学习的部分细节


### 4.1 数据集构造 

(第3部分讲了一点，就是Long-CoT SFT的前提)



---

### 4.2 问题设定:


**优化目标函数**:
$$\max_\theta\mathbb{E}_{(x,y^*)\thicksim\mathcal{D},(y,z)\thicksim\pi_\theta}\left[r(x,y,y^*)\right]\mathrm{~.}$$

>[!tip] **核心要点解释**: 
>1. **方法**：用强化学习（RL）训练模型生成**思维链（CoT）**，通过奖励机制优化策略。
>2. **奖励设计**：
>	- **可验证问题**（如编程）：奖励由预定义规则决定（如测试用例是否通过）
>	- **自由形式问题**（如开放问答）：训练一个**奖励模型**  $r(x, y, y^*)$ ，预测答案 $y$ 是否与真实值 $y^*$ 匹配（输出0/1, ==即奖励模型是一个二分类模型==）
>1. **生成过程**：
>	- 给定问题 $x$ ，模型 $\pi_\theta$ 生成CoT（ $z$ ）和最终答案（ $y$ ），即：$$z \sim \pi_\theta(\cdot|x), y \sim \pi_\theta(\cdot|x, z)$$
>1. **评估标准**：CoT的质量取决于其能否推导出正确答案（奖励值为1）
>2. **优化目标**：最大化模型生成的CoT和答案的期望奖励，以提升策略 $\pi_\theta$ 。
>
>注意：
> **策略 $\pi_\theta$** : 控制生成过程的参数化模型


**优化策略**：

一种基于 **在线策略镜像下降（Online Policy Mirror Descent）** 的训练算法，用于解决强化学习或序列决策问题。
$$\max_\theta\mathbb{E}_{(x,y^*)\thicksim\mathcal{D}}\left[\mathbb{E}_{(y,z)\thicksim\pi_\theta}\left[r(x,y,y^*)\right]-\tau\mathrm{KL}(\pi_\theta(x)||\pi_{\theta_i}(x))\right]$$

>[!tip]
>其中:
>-  $\mathcal{D}$  是数据分布，表示输入 $x$  和目标  $y^*$  的联合分布。
>- $r(x, y, y^*)$  是奖励函数，衡量策略  $\pi_\theta$  的输出  $(y, z)$  与目标  $y^*$  的匹配程度。
>- $\text{KL}(\pi_\theta(x) \| \pi_{\theta_i}(x))$  是策略  $\pi_\theta$  和参考策略  $\pi_{\theta_i}$  的 **KL散度**，用于控制新策略与旧策略之间的差异。
>- $\tau > 0$  是正则化参数，控制 KL 正则项的强度。

**目标函数的意义**：

- 第一项  $\mathbb{E}_{(y, z) \sim \pi_\theta} \left[ r(x, y, y^*) \right]$  表示在当前策略下期望的奖励。
- 第二项  $\tau \text{KL}(\pi_\theta(x) \| \pi_{\theta_i}(x))$  是正则化项，确保新策略不会偏离参考策略太多，从而保持策略的稳定性。
>跟DPO等强化学习类似, 都是奖励函数 + 一个KL散度（约束奖励函数）



这时候，根据优化策略的 **闭式解(Closed-form Solution)** 如下:
$$\pi^*(y, z | x) = \pi_{\theta_i}(y, z | x) \exp(r(x, y, y^*) / \tau) / Z$$
>[!tip]
>其中：
>-  $Z = \sum_{y', z'} \pi_{\theta_i}(y', z' | x) \exp(r(x, y', y^*) / \tau)$  是归一化因子，确保  $\pi^*(y, z | x)$  是一个有效的概率分布。
>
>**归一化因子的作用**: 归一化因子 $Z$  确保策略的概率分布性质，即所有可能的  $(y, z)$  的概率之和为 1
>
>$^*$ **闭式解**就是可以通过数学推导直接得到的显式解


通过对上式取对数（*非常容易推导，不要被吓到了*），可以得到以下约束：
$$r(x, y, y^*) - \tau \log Z = \tau \log \frac{\pi^*(y, z | x)}{\pi_{\theta_i}(y, z | x)}.$$

这一约束允许在优化过程中利用 **离线数据（off-policy data）**，因为  $\pi^*$  的形式可以直接从**参考策略**  $\pi_{\theta_i}$  和**奖励**  $r(x, y, y^*)$  推导出来。

> off-policy data 是相对 on-policy data而言，
> - on-policy data是智能体通过**当前策略与环境实时交互生成数据**，并立即用于更新策略（例如 SARSA）；
> - off-policy data 是智能体使用**预先收集的、由其他策略生成的历史数据**进行训练（例如 Q-Learning、DQN）。



**损失函数**：
根据上述约束，定义替代损失函数（surrogate loss）：

$$L(\theta) = \mathbb{E}_{(x, y^*) \sim \mathcal{D}} \left[ \mathbb{E}_{(y, z) \sim \pi_{\theta_i}} \left[ \left( r(x, y, y^*) - \tau \log Z - \tau \log \frac{\pi_\theta(y, z | x)}{\pi_{\theta_i}(y, z | x)} \right)^2 \right] \right]$$

这一损失函数的目标是使**当前策略**  $\pi_\theta$  尽可能接近**最优策略**  $\pi^*$ ，同时考虑奖励和正则化项。


由于  $\tau \log Z$  难以直接计算，可以通过**采样近似**：

$$\tau \log Z \approx \tau \log \frac{1}{k} \sum_{j=1}^k \exp(r(x, y_j, y^*) / \tau)$$

其中  $(y_1, z_1), \ldots, (y_k, z_k) \sim \pi_{\theta_i}$ 是从参考策略  $\pi_{\theta_i}$  中采样的样本。

此外，还可以用采样奖励的均值  $\bar{r} = \text{mean}(r(x, y_1, y^*), \ldots, r(x, y_k, y^*))$  来进一步简化计算，这种方法在实践中效果良好。


最终，通过**梯度下降优化**替代损失函数  $L(\theta)$ 。对于每个问题  $x$ ，从参考策略  $\pi_{\theta_i}$  中采样  $k$  个响应  $(y_j, z_j)$ ，梯度公式为：

$$\frac{1}{k} \sum_{j=1}^k \left( \nabla_\theta \log \pi_\theta(y_j, z_j | x) (r(x, y_j, y^*) - \bar{r}) - \frac{\tau}{2} \nabla_\theta \left( \log \frac{\pi_\theta(y_j, z_j | x)}{\pi_{\theta_i}(y_j, z_j | x)} \right)^2 \right)$$

>[!tip] **梯度项的意义**
>- 第一项  $\nabla_\theta \log \pi_\theta(y_j, z_j | x) (r(x, y_j, y^*) - \bar{r})$  是奖励驱动的更新，鼓励策略向高奖励的方向调整。
>- 第二项  $\frac{\tau}{2} \nabla_\theta \left( \log \frac{\pi_\theta(y_j, z_j | x)}{\pi_{\theta_i}(y_j, z_j | x)} \right)^2$  是正则化项，确保新策略不偏离参考策略太多。(==用平方是因为可以更敏感， L2正则化?==)





---


## 附录：4.2中闭式解推导

目标是找到一个策略 $\pi^*$，在给定参考策略 $\pi_{\theta_i}$ 的情况下，最大化期望奖励的同时限制策略变化幅度。
$$\max_{\pi} \mathbb{E}_{(x, y^*) \sim \mathcal{D}} \left[ \mathbb{E}_{(y, z) \sim \pi(\cdot|x)} \left[ r(x, y, y^*) \right] - \tau \cdot \text{KL}(\pi(\cdot|x) \| \pi_{\theta_i}(\cdot|x)) \right] \tag{1}$$

根据 KL散度定义:
$$\text{KL}(\pi \| \pi_{\theta_i}) = \mathbb{E}_{(y, z) \sim \pi} \left[ \log \frac{\pi(y, z|x)}{\pi_{\theta_i}(y, z|x)} \right] \tag{2}$$

把 （2）带入 （1）中，结果如下:
$$\max_{\pi} \mathbb{E}_{(x, y^*) \sim \mathcal{D}} \left[ \mathbb{E}_{(y, z) \sim \pi} \left[ r(x, y, y^*) - \tau \log \frac{\pi(y, z|x)}{\pi_{\theta_i}(y, z|x)} \right] \right] \tag{3}$$

把（3）中的期望展开，再加上对数：
$$\max_{\pi} \mathbb{E}_{x, y^*} \left[ \sum_{y, z} \pi(y, z|x) \left( r(x, y, y^*) - \tau \log \pi(y, z|x) + \tau \log \pi_{\theta_i}(y, z|x) \right) \right] \tag{4}$$
提取公因式进一步整理：
$$\max_{\pi} \mathbb{E}_{x, y^*} \left[ \sum_{y, z} \pi(y, z|x) \cdot \left( \frac{r(x, y, y^*)}{\tau} + \log \pi_{\theta_i}(y, z|x) - \log \pi(y, z|x) \right) \right] \cdot \tau \tag{5}$$

由于 $\pi(\cdot|x)$ 是概率分布，需满足归一化约束：

$$\sum_{y, z} \pi(y, z|x) = 1$$

引入拉格朗日乘子 $\lambda$，构造拉格朗日函数：

$$\mathcal{L}(\pi, \lambda) = \mathbb{E}_{x, y^*} \left[ \sum_{y, z} \pi(y, z|x) \left( \frac{r(x, y, y^*)}{\tau} + \log \pi_{\theta_i}(y, z|x) - \log \pi(y, z|x) \right) \right] \cdot \tau - \lambda \left( \sum_{y, z} \pi(y, z|x) - 1 \right) \tag{6}$$

接下来就是拉格朗日乘数法的标准解法，先求导，再领导数为0，就可以得到解（*DNA动了吗*）。
对 $\pi(y, z|x)$ 求导，令导数为零：

$$\frac{\partial \mathcal{L}}{\partial \pi(y, z|x)} = \left( \frac{r(x, y, y^*)}{\tau} + \log \pi_{\theta_i}(y, z|x) - \log \pi(y, z|x) - 1 \right) \cdot \tau - \lambda = 0 \tag{7}$$

整理后得到：
$$\log \pi(y, z|x) = \frac{r(x, y, y^*)}{\tau} + \log \pi_{\theta_i}(y, z|x) - \frac{\lambda}{\tau} - 1 \tag{8}$$

对两边取 $e$ 为底的指数：

$$\pi(y, z|x) = \pi_{\theta_i}(y, z|x) \cdot \exp\left( \frac{r(x, y, y^*)}{\tau} - \frac{\lambda}{\tau} - 1 \right) \tag{9}$$

利用归一化约束 $\sum_{y, z} \pi(y, z|x) = 1$，定义归一化因子 $Z$：

$$Z = \exp\left( \frac{\lambda}{\tau} + 1 \right) = \sum_{y', z'} \pi_{\theta_i}(y', z'|x) \exp\left( \frac{r(x, y', y^*)}{\tau} \right)\tag{10}$$

最终得到闭式解：

$$\pi^*(y, z|x) = \frac{\pi_{\theta_i}(y, z|x) \exp\left( \frac{r(x, y, y^*)}{\tau} \right)}{Z} \tag{11}$$

证明完毕。