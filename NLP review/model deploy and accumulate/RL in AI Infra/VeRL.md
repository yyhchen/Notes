
1. [知乎 # [AI Infra] VeRL 框架入门&代码带读](https://zhuanlan.zhihu.com/p/27676081245)
2. [VeRL docs](https://verl.readthedocs.io/en/latest/index.html)

---
强化学习（RL）对大模型复杂推理能力提升有关键作用，然而，**RL 复杂的计算流程以及现有系统局限性，也给训练和部署带来了挑战**。[VeRL](https://link.zhihu.com/?target=https%3A//github.com/volcengine/verl)是字节跳动seed团队和香港大学开发的强化学习仓库。该框架采用混合编程模型，融合单控制器（Single-Controller）的灵活性和多控制器（Multi-Controller）的高效性，可更好实现和执行多种RL算法，显著提升训练吞吐量，降低开发和维护复杂度。

本文会先简单介绍VeRL框架涉及的一些概念，并且简单阅读整理VeRL框架的一些核心算法逻辑，以方便开发者对该框架加深了解。除了VeRL以外，还有[OpenRLHF](https://link.zhihu.com/?target=https%3A//github.com/OpenRLHF/OpenRLHF)等非常优秀的国产开源训练框架，设计理念都非常简洁，且各有一些独特的优势。


