# what's langchain?


---


LangChain 是一个开源的编排框架，主要用于开发由大型语言模型（LLM）驱动的应用程序。它提供了一系列工具和API，这些工具和API在Python和Javascript库中使用，可以简化构建聊天机器人、虚拟代理等LLM驱动型应用程序的过程。

LangChain 几乎可以作为所有LLM的通用接口，**为构建LLM应用程序并将其与外部数据源和软件工作流程集成提供集中式开发环境**。它的基于模块的方法允许开发人员和数据科学家动态比较不同的提示，甚至比较不同的基础模型，而无需重写代码。这种模块化环境还允许程序使用多个LLM：例如，应用程序使用一个LLM解释用户查询，并使用另一个LLM编写响应。

LangChain 提供了多种功能，包括LLM调用、Prompt管理、文档加载器、文本分割、向量数据库支持等。它还支持多种模型接口，如OpenAI、HuggingFace、AzureOpenAI等，并提供了Fake LLM用于测试。此外，LangChain还支持缓存、用量记录和流模式。


<br>
<br>


## demo 演示

### 1. 模型 配置定义

```python
import os
from langchain.llms import OpenAI


os.environ['OPENAI_API_KEY'] = ''

GPT = OpenAI(model_name='gpt-3.5-turbo')
```

<br>


### 2. prompt template定义，结合成 chain (问答演示)
```python
from langchain import LLMChain, PromptTemplate


template = """
I want you to act as a naming consultant for new companies.
What is a good name for a company that makes {product}?
"""

prompt = PromptTemplate(
    input_variables=["product"],
    template=template,
)

llm_chain = LLMChain(
    prompt=prompt,
    llm=GPT
)

product="Table tennis racket"
print(llm_chain.run(product))
```

<br>

### 3. 多重提问

- 错误示范
```python
qs = [
    {'product': "Running shoes"},
    {'product': "provide cryptocurrency manageent service to clients"},
    {'product': "Chinese food in Africa"},
    {'product': "cleaning service to individual home owners"}
]
llm_chain.generate(qs)
```

这里的错误是，必须得将字符串整个到一起(变成一个字符串数组元素)，不能这样分开定义(多个json元素), 否则无法迭代回答
(Note that the below format doesn't feed the questions in iteratively but instead all in one chunk.)

<br>

- 正确示范
```python
qs = [
    "Running shoes",
    "provide cryptocurrency manageent service to clients",
    "Chinese food in Africa",
    "cleaning service to individual home owners"
]
print(llm_chain.run(qs))
``` 

运行结果：
```
Running shoes: Velocity Athletics
Provide cryptocurrency management service to clients: BlockSafe
Chinese food in Africa: Orient Fusion
Cleaning service to individual home owners: PureClean Homes
```
