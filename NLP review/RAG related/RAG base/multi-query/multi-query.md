# Retrival-Augmented Generation

---

<br>

### 多重查询（Multi-Query）

也就说，用户输入不准确，用户不知道正确的问题是什么，怎么办？

这时候就需要用到多重查询，找 语义 相同的句子，经过大模型去拓展多个问题

- 多重查询

![alt text](image-1.png)


<br>

- 搜索过程

![alt text](image.png)


<br>

- 流程图
![alt text](image-2.png)

（这里我有个问题，Q1～Q3 对应的索引 加上 文档 内容不会太多了吗？大模型的上下文窗口是有限制的，这不会涉及到一个 截断的问题吗？ ）