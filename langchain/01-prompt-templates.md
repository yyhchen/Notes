
# how to use prompt-template in langchain

---

首先，我们看看如何定义 prompt，一个好的prompt组成部分应该是怎么样的。

<br>

### Structure of a Prompt

主要组成部分：
- Instructions
- context or external information
- query or user input
- output indicator

<br>

#### prompt编写示范案例(demo)
```python
prompt = """Answer the question based on the context below. If the
question cannot be answered using the information provided answer
with "I don't know".

Context: Large Language Models (LLMs) are the latest models used in NLP.
Their superior performance over smaller models has made them incredibly
useful for developers building NLP enabled applications. These models
can be accessed via Hugging Face's `transformers` library, via OpenAI
using the `openai` library, and via Cohere using the `cohere` library.

Question: Which libraries and model providers offer LLMs?

Answer: """
```

这个案例提供了4中，一个prompt 中，至少包涵2中组成以上比较好。


<br>


### example for langchain（prompt template）

```python
import os

os.environ['OPENAI_API_KEY'] = ''
```

<br>

```python
from langchain.llms import OpenAI

# initialize the models
openai = OpenAI(
    model_name="gpt-3.5-turbo",
    openai_api_key=""
)
```

**通过之前上面案例的 prompt 生成内容**

```python
print(openai(prompt))
```

**运行结果如下：**

Hugging Face's `transformers` library, OpenAI's `openai` library, and Cohere's `cohere` library offer LLMs.


<br>
<br>
<br>



### 实际上的prompt编写


实际上，我们通常不会事先知道用户提示（Questions）是什么。因此，我们不是直接编写提示，而是使用单个输入变量“query”创建一个“PromptTemplate”。

```python
from langchain import PromptTemplate

# prompt
template = """Answer the question based on the context below. If the
question cannot be answered using the information provided answer
with "I don't know".

Context: Large Language Models (LLMs) are the latest models used in NLP.
Their superior performance over smaller models has made them incredibly
useful for developers building NLP enabled applications. These models
can be accessed via Hugging Face's `transformers` library, via OpenAI
using the `openai` library, and via Cohere using the `cohere` library.

Question: {query}

Answer: """

# prompt 封装
prompt_template = PromptTemplate(
    input_variables=["query"],
    template=template
)
```
<br>

因此，现在可以通过 一个 `query` 参数来定义用户输入, 方式如下：

```python
print(
    prompt_template.format(
        query="Which libraries and model providers offer LLMs?"
    )
)
```


---

除了 `PromptTemplate` 还有其他的

<br>
<br>


### Few shot Prompt templates

`PromptTemplate` 适用于生成简单的、直接基于参数的提示，而 `FewshotPromptTemplate` 适用于需要**提供示例以指导模型学习的场景**。`FewshotPromptTemplate` 提供了一种更高级的方法，通过示例来增强模型对任务的理解，特别适用于那些需要模型通过少量示例快速适应新任务的场景。


**起因：** 有时候模型可能不理解我们问的是什么，所以需要提供一些案例让其依葫芦画瓢


以下就是一个 `few-shot` 的 `prompt` 案例
```python
prompt = """The following are exerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:

User: My router is not working
AI: Would you please reboot the router? Let me give you the cli command.

User: It is still not working
AI: Would you please run a diagnostic against the line card

User: Can we do it tomorrow?
AI: No, this is a very urgent issue, please bear with me for a moment

User: What defines a network engineer
AI: """

print(openai(prompt))

```



<br>
<br>


### 具体langchain中的 `FewShotPromptTemplate` 用法

```python{.line-numbers}
from langchain import FewShotPromptTemplate

# create our examples
examples = [
    {
        "query": "My router is not working",
        "answer": "Would you please reboot the router? Let me give you the cli command"
    }, {
        "query": "It is still not working",
        "answer": "Would you please run a diagnostic against the line card"
    },
    {
        "query": "Can we do it tomorrow?",
        "answer": "No, this is a very urgent issue, please bear with me for a moment"
    }
]

# create a example template
example_template = """
User: {query}
AI: {answer}
"""

# create a prompt example from above template
example_prompt = PromptTemplate(
    input_variables=["query", "answer"],
    template=example_template
)

# now break our previous prompt into a prefix and suffix
# the prefix is our instructions
prefix = """The following are exerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:
"""
# and the suffix our user input and output indicator
suffix = """
User: {query}
AI: """

# now create the few shot prompt template
few_shot_prompt_template = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix=prefix,
    suffix=suffix,
    input_variables=["query"],
    example_separator="\n\n"
)

```

<br>

生成内容
```python
query = "What defines a netwrok enginner?"

print(few_shot_prompt_template.format(query=query)) # print full prompt

# print generative content
print(openai(
    few_shot_prompt_template.format(query=query)
))
```


<br>
<br>


### 1. 输入内容太长或太短的情况

`LengthBasedExampleSelector` 可以缓解这个问题。

<br>

`LengthBasedExampleSelector` 在 LangChain 中的作用是**根据输入的长度来选择使用哪些示例**。这种选择器在构建提示时特别有用，可以避免超出上下文窗口的长度。对于较长的输入，它会选择较少的示例进行包含，而对于较短的输入，则会选择更多的示例。

<br>

**假设我现在有一个更长的 examples:**
```python
examples = [
    {
        "query": "How are you?",
        "answer": "I can't complain but sometimes I still do."
    }, {
        "query": "What time is it?",
        "answer": "It's time to get a watch."
    }, {
        "query": "What is the meaning of life?",
        "answer": "42"
    }, {
        "query": "What is the weather like today?",
        "answer": "Cloudy with a chance of memes."
    }, {
        "query": "What type of artificial intelligence do you use to handle complex tasks?",
        "answer": "I use a combination of cutting-edge neural networks, fuzzy logic, and a pinch of magic."
    }, {
        "query": "What is your favorite color?",
        "answer": "79"
    }, {
        "query": "What is your favorite food?",
        "answer": "Carbon based lifeforms"
    }, {
        "query": "What is your favorite movie?",
        "answer": "Terminator"
    }, {
        "query": "What is the best thing in the world?",
        "answer": "The perfect pizza."
    }, {
        "query": "Who is your best friend?",
        "answer": "Siri. We have spirited debates about the meaning of life."
    }, {
        "query": "If you could do anything in the world what would you do?",
        "answer": "Take over the world, of course!"
    }, {
        "query": "Where should I travel?",
        "answer": "If you're looking for adventure, try the Outer Rim."
    }, {
        "query": "What should I do today?",
        "answer": "Stop talking to chatbots on the internet and go outside."
    }
]
```


<br>

### LengthBasedExampleSelector 使用案例
```python
from langchain.prompts.example_selector import LengthBasedExampleSelector

example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=50  # this sets the max length that examples should be
)
```

<br>

**其中，`max_length` 是根据换行和空格来进行 划分的, 比如说：**

```python
import re

some_text = "There are a total of 8 words here.\nPlus 6 here, totaling 14 words."

words = re.split('[\n ]', some_text)
print(words, len(words))
```

**输出结果：**

['There', 'are', 'a', 'total', 'of', '8', 'words', 'here.', 'Plus', '6', 'here,', 'totaling', '14', 'words.'] 14

<br>

**之后还需要定义初始化 `dynamic_prompt_template`**

```python
# now create the few shot prompt template
dynamic_prompt_template = FewShotPromptTemplate(
    example_selector=example_selector,  # use example_selector instead of examples
    example_prompt=example_prompt,
    prefix=prefix,
    suffix=suffix,
    input_variables=["query"],
    example_separator="\n"
)
```

<br>

**现在测试一下动态定义的 prompt 长度等是如何的：**
```python
print(dynamic_prompt_template.format(query="How do birds fly?"))
```

**输出结果：**

The following are exerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples: 


User: How are you?
AI: I can't complain but sometimes I still do.


User: What time is it?
AI: It's time to get a watch.


User: What is the meaning of life?
AI: 42


User: How do birds fly?
AI: 

**可以看到，动态定义的 prompt 由于提问的内容短，所以增加了一定的few-shot案例; 现在运行看看结果**

```python
query = "How do birds fly?"

print(openai(
    dynamic_prompt_template.format(query=query)
))
```

**输出结果：**
 On the wings of dreams and determination!

<br>


**假如我现在加长提问的 内容长度**
```python
query = """If I am in America, and I want to call someone in another country, I'm
thinking maybe Europe, possibly western Europe like France, Germany, or the UK,
what is the best way to do that?"""

print(dynamic_prompt_template.format(query=query))
```

**输出结果：**
The following are exerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples: 


User: How are you?
AI: I can't complain but sometimes I still do.


User: If I am in America, and I want to call someone in another country, I'm
thinking maybe Europe, possibly western Europe like France, Germany, or the UK, what is the best way to do that?
AI: 


**可以看到，由于输入问题的长度更长了，few-shot的案例少了；如果想要更长的示例，通过增加 `max_length` 即可**


<br>
<br>
<br>


### 2. 寻找更加相关的输入内容，以减少输入长度

`SimilarityExampleSelector` 可以很好解决这个问题。

`SimilarityExampleSelector` 在 LangChain 中的作用是根据输入的相似度来选择使用哪些示例。这种选择器在构建提示时特别有用，可以确保所选示例与输入的语义相似度最高。

<br>

### SimilarityExampleSelector 使用案例 
```python
from langchain.prompts.example_selector import SemanticSimilarityExampleSelector
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)

# These are a lot of examples of a pretend task of creating antonyms.
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"},
]
```

<br>

```python
example_selector = SemanticSimilarityExampleSelector.from_examples(
    # This is the list of examples available to select from.
    examples,
    # This is the embedding class used to produce embeddings which are used to measure semantic similarity.
    OpenAIEmbeddings(),
    # This is the VectorStore class that is used to store the embeddings and do a similarity search over.
    Chroma,
    # This is the number of examples to produce.
    k=1
)
similar_prompt = FewShotPromptTemplate(
    # We provide an ExampleSelector instead of examples.
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="Give the antonym of every input",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
)
```

<br>

**测试一下：**

```python
# Input is a feeling, so should select the happy/sad example
print(similar_prompt.format(adjective="worried"))
```

**输出结果：**

Give the antonym of every input

Input: happy
Output: sad

Input: worried
Output:


<br>

**测试一下：**
```python
# Input is a measurement, so should select the tall/short example
print(similar_prompt.format(adjective="cloudy"))
```

**输出结果：**

Give the antonym of every input

Input: sunny
Output: gloomy

Input: cloudy
Output:

