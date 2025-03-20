# RLHF

---

[知乎 RLHF](https://zhuanlan.zhihu.com/p/657490625)


<br>
<br>

### ChatGPT需要有大致三个阶段的训练过程

- **Pretraining:** 在大规模“无监督”的语料上训练，训练任务是预测下一个词。
- **Supervised Fine-Tuning（SFT）:** 在人类标注上进行微调，所谓人类标注就是人类写Prompt，人类写答案。然后语言模型学习模仿人类是如何作答的。这部分通常要求数据集多样性很好，也因为标注成本很高，通常量级很小。
- **Reinforcement Learning with human feedback（RLHF）:** 对于同一个Prompt把模型的多个输出给人类排序，获取人类偏好标注。用人类的偏好标注，训练一个reward model。训练得到的reward model会作为PPO算法中的reawrd function，来继续优化SFT得到的模型。