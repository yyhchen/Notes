
---


`Beam search` 是一种启发式搜索算法，常用于自然语言处理中的**解码阶段**，特别是在机器翻译、语音识别和文本生成等任务中。它的主要目的是在解码过程中保持一个候选列表（即“beam”），这个列表中的每个元素都是一个部分构建的序列（如句子的一部分），并且根据某种评分机制（如概率）对这些候选进行排序和筛选。

`Beam search` 的**核心思想**是在每个时间步（或每个词汇的生成步骤）不仅仅考虑单一的最优候选，而是维护一个大小为 B 的候选集（B 通常被称为 beam width 或 beam size），这样可以在解码过程中探索多条路径。在每一步，算法都会根据当前的候选集扩展出新的候选集，并从中选择评分最高的 B 个候选，作为下一步的候选集。这个过程一直持续到生成序列的结束符被添加到某个候选中，或者达到序列的最大长度限制。


下面通过一个简单的例子来说明 beam search 的过程：
假设我们有一个语言模型，它在每个时间步都会为下一个词汇生成一个概率分布。我们要使用这个模型来生成一个句子，并且设置 beam size 为 3。
1. 第一步，模型生成了三个词汇的概率分布，我们选择概率最高的三个词汇作为当前步的候选：["我"，"你"，"他"]。
2. 第二步，对于每个候选，模型生成接下来一个词汇的概率分布。我们为每个候选词汇组合计算一个联合概率，并选择概率最高的三个组合作为新的候选集。例如，可能的新候选集是：["我 爱"，"你 是"，"他 的"]。
3. 重复这个过程，直到某个候选序列以句子结束符（如句号）结束，或者达到序列的最大长度。


`Beam search` 允许算法在解码过程中考虑多个可能的候选，从而**在一定程度上平衡了贪心算法（每次选择最优候选）和穷举搜索（考虑所有可能的候选）之间的权衡**。通过调整 beam size，我们可以控制算法在解码过程中探索的广度和深度。较大的 beam size 可以提高找到最优解的可能性，但同时也会增加计算成本。
需要注意的是，Beam search 并不保证能找到最优解，因为它可能会在解码过程中丢弃最终的最优路径。此外，Beam search 也不保证能得到多样性很强的输出，因为它倾向于选择概率较高的候选，这可能会导致生成的句子比较单一。为了解决这个问题，有时会采用一些额外的技术，如随机化或长度惩罚，来增加输出句子的多样性。



# (伪代码)Beam Search 在模型中如何使用的 


```python

# 1. model
class Model(nn.Module):
	def __init__(self, vocab_size, embed_dim, hidden_dim):
		super().__init__()
		self.embedding = nn.Embedding(vocab_size, embed_dim)
		self.lstm = nn.LSTM(embed_dim, hidden_dim)
		self.fc = nn.Linear(hidden_dim, vocab_size)

	def forward(self, x, hidden):
		x = self.embdding(x)
		output, hidden = self.lstm(x, hidden)
		logits = self.fc(output)
		return logits, hidden


# Beam Search 算法
def generate_beam_search(model, start_token, beam_width=3, max_length=10):
    # 初始化隐藏状态和候选序列
    hidden = None
    # (序列, 累积概率, 隐藏状态)
    candidates = [(torch.tensor([start_token]), 0.0, hidden)]  
    
    for _ in range(max_length):
        new_candidates = []
        for seq, score, hidden in candidates:
            # 如果序列已结束（遇到结束符），直接保留
            if seq[-1].item() == end_token:
                new_candidates.append((seq, score, hidden))
                continue
            
            # 前向传播得到当前时间步的 logits
            with torch.no_grad():
                logits, hidden = model(seq.unsqueeze(0), hidden)
                
                # (1, vocab_size)  
                probs = torch.softmax(logits[:, -1, :], dim=-1)  
            
            # 选择 Top-k 候选（k=beam_width）
            top_probs, top_tokens = torch.topk(probs.squeeze(0), beam_width)
            
            # 生成新的候选序列
            for token, prob in zip(top_tokens, top_probs):
                new_seq = torch.cat([seq, token.unsqueeze(0)], dim=0)
                new_score = score + torch.log(prob).item()  # 累积对数概率
                new_candidates.append((new_seq, new_score, hidden))
        
        # 保留全局 Top-beam_width 候选
        new_candidates.sort(key=lambda x: -x[1])  # 按概率降序
        candidates = new_candidates[:beam_width]
    
    # 返回概率最高的完整序列
    best_seq = max(candidates, key=lambda x: x[1])[0]
    return best_seq


lstm_model = Model(vocab_size, embed_dim, hidden_dim)
output_seq = generate_beam_search(lstm_model, start_token, beam_width=3, max_length=10)

```