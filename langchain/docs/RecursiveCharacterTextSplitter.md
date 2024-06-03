# RecursiveCharacterTextSplitter

**源于：**

```python
from langchain.text_splitter
```

---


## demo

这里的 `docs` 是存放了 文档内容的 list 

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500, chunk_overlap=150)
split_docs = text_splitter.split_documents(docs)
```


<br>
<br>



## RecursiveCharacterTextSplitter 

`RecursiveCharacterTextSplitter` 是 LangChain 库中推荐用于通用文本的文本分割器。它通过一系列字符**递归地分割文档**，旨在保持文本的语义完整性。以下是对该类的一些关键点的详细解释：
1. **分割方式**：`RecursiveCharacterTextSplitter` 通过指定的字符列表来分割文本。默认的字符列表是 `["\n\n", "\n", " ", ""]`，这意味着分割器首先尝试在两个换行符之间分割，然后是一个换行符，接着是空格，最后是没有任何字符的地方。这种分割方式有助于保持段落、句子和单词的完整性，因为它们通常是最强的语义相关文本片段。
2. **参数**：
   - `chunkSize`：控制最终文档块的最大大小（以字符数为单位）。在需要时，分割器会递归地减小块的大小，直到满足这个条件。
   - `chunkOverlap`：指定文档块之间应有多少重叠。这有助于确保文本不会被不恰当地分割。（**主要作用是确保文本在分割时不会出现不自然的断裂，从而保持文本的连贯性和语义完整性**。）
   - 其他参数包括 `separators`（用于分割的字符列表），`keep_separator`（是否保留分隔符），和 `is_separator_regex`（分隔符是否是正则表达式）。
3. **使用场景**：这个分割器非常适合于需要保持文本语义完整性的场景，例如，当处理具有复杂结构和语义相关性的文本时。它也适用于处理不同语言的文本，包括那些没有明确单词边界的语言，如中文、日文和泰文。对于这些语言，可以通过重写分隔符列表来包含额外的标点符号，以保持单词的完整性。
4. **方法**：`RecursiveCharacterTextSplitter` 提供了多种方法来处理文本和文档，包括 `create_documents`（从文本列表创建文档），`split_documents`（分割文档），和 `transform_documents`（通过分割来转换文档序列）。

综上所述，`RecursiveCharacterTextSplitter` 是一个灵活且功能强大的工具，适用于各种文本分割需求，特别是在需要保持文本语义完整性的情况下。


