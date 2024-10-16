# DPO 算法（偏好对齐用）

---

DPO（Direct Preference Optimization）比起传统基于强化学习PPO的方式，它改进了以下两点：

- **不再训练奖励模型，直接使用人类标注的偏好数据，一步到位训练对齐模型。**
- **不再使用强化学习的方法，通过数学推理，将原始的偏好对齐目标步步简化，最后通过类似于sft的方式，用更简单的步骤训练出对齐模型。**


