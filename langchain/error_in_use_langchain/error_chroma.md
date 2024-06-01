# 在langchain中使用 chromadb 时遇到的错误

---

## langchain + chroma 做本地知识问题

**background：** langchain + chroma + chatglm2-6b

**原代码：**
```python
运行代码
from langchain.embeddings.huggingface import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
from langchain.text_splitter import CharacterTextSplitter
from langchain.document_loaders import DirectoryLoader
from langchain.chains import RetrievalQA
from transformers import AutoModel, AutoTokenizer

# 加载文件夹中的所有txt类型的文件
loader = DirectoryLoader(r'D:/Notes/NLP review', glob='**/*.md')
# 将数据转成 document 对象，每个文件会作为一个 document
documents = loader.load()

# 初始化加载器
text_splitter = CharacterTextSplitter(chunk_size=1500, chunk_overlap=0)
# 切割加载的 document
split_docs = text_splitter.split_documents(documents)

# 初始化 huggingface 的 embeddings 对象
embed_model_name = "D:\CodeLibrary\ChatGLM\embedtext2vec"
tokenizer = AutoTokenizer.from_pretrained(embed_model_name)
embed_model = AutoModel.from_pretrained(embed_model_name)
embeddings = HuggingFaceEmbeddings(model_name=embed_model_name)

# 将 document 通过 openai 的 embeddings 对象计算 embedding 向量信息并临时存入 Chroma 向量数据库，用于后续匹配查询
docsearch = Chroma.from_documents(split_docs, embeddings)

# 创建问答对象
qa = RetrievalQA.from_chain_type(llm=model, chain_type="stuff", vectorstore=docsearch, return_source_documents=True)
# 进行问答
result = qa({"query": "电影《红高粱》简介？"})
print(result)

```

<br>
<br>
<br>

### 错误1
```
ValueError: Expected EmbeddingFunction.__call__ to have the following signature: odict_keys(['self', 'input']), got odict_keys(['self', 'args', 'kwargs'])
Please see https://docs.trychroma.com/embeddings for details of the EmbeddingFunction interface.
Please note the recent change to the EmbeddingFunction interface: https://docs.trychroma.com/migration#migration-to-0416---november-7-2023 
```

<br>

**错误代码部分:**

```python
docsearch = Chroma.from_documents(split_docs, embeddings)

embeddings = HuggingFaceEmbeddings(model_name=embed_model_name)
```

原因可能是 `langchain` 和 `chromadb` **package 版本之间的问题**

<br>

**解决办法：** `pip install -U langchain chromadb`

<br>
<br>


### 错误2
```
ValidationError: 2 validation errors for LLMChain
llm
  instance of Runnable expected (type=type_error.arbitrary_type; expected_arbitrary_type=Runnable)
llm
  instance of Runnable expected (type=type_error.arbitrary_type; expected_arbitrary_type=Runnable)
```

<br>

**错误（使用）代码部分：**

#### 本地模型部署
```python
from transformers import AutoTokenizer, AutoModel
model_path = "D:\CodeLibrary\ChatGLM\chatglm26b"

tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)
# model = AutoModel.from_pretrained(model_path, trust_remote_code=True, device='cuda')
# 按需修改，目前只支持 4/8 bit 量化
model = AutoModel.from_pretrained(model_path, trust_remote_code=True).quantize(4).half().cuda()
model = model.eval()
```

<br>

**错误信息表明**在创建 `LLMChain` 对象时，`llm` 参数的类型不符合预期。`LLMChain` **期望的 `llm` 参数是一个 `Runnable` 类型的实例**，但是这里提供的 `model` 是 `AutoModel` 类型的实例。

在 `langchain` 库中，`LLMChain` 通常是和语言模型接口一起使用的，这些接口需要实现 `Runnable` 协议。`AutoModel` 来自 `transformers` 库，它没有实现 `Runnable` 协议，因此不能直接用作 `LLMChain` 的 `llm` 参数。

为了解决这个问题，需要使用 `langchain` 提供的 `HuggingfaceAPI` 或者其他支持 `Runnable` 协议的模型。如果想要使用 `transformers` 库中的模型，需要创建一个适配器来使其符合 `Runnable` 协议。

<br>



#### 下面的类定义，它使用 `AutoModel` 和 `AutoTokenizer` 来实现 `Runnable` 协议：
```python
from langchain.llms.base import LLM
from typing import Any, List, Optional
from langchain.callbacks.manager import CallbackManagerForLLMRun
from transformers import AutoTokenizer, AutoModelForCausalLM

import warnings
warnings.filterwarnings("ignore", category=UserWarning, module="torch")

class ChatGLM_LLM(LLM):
    # 基于本地 InternLM 自定义 LLM 类
    tokenizer : AutoTokenizer = None
    model: AutoModelForCausalLM = None

    def __init__(self, model_path :str):
        # model_path: InternLM 模型路径
        # 从本地初始化模型
        super().__init__()
        print("正在从本地加载模型...")
        self.tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)
        self.model = AutoModelForCausalLM.from_pretrained(model_path, trust_remote_code=True).to(torch.bfloat16).cuda()
        self.model = self.model.eval()
        print("完成本地模型的加载")

    def _call(self, prompt : str, stop: Optional[List[str]] = None,
                run_manager: Optional[CallbackManagerForLLMRun] = None,
                **kwargs: Any):
        # 重写调用函数
        response, history = self.model.chat(self.tokenizer, prompt , history=[])
        return response
        
    @property
    def _llm_type(self) -> str:
        return "ChatGLM3-6B"
```


<br>
<br>


### 错误3

```
ValidationError: 2 validation errors for RetrievalQA
retriever
  field required (type=value_error.missing)
vectorstore
  extra fields not permitted (type=value_error.extra)
```

<br>

**错误代码部分：**

```
qa = RetrievalQA.from_chain_type(llm=model, chain_type="stuff", vectorstore=docsearch, return_source_documents=True)
```

<br>

错误信息表明在创建 `RetrievalQA` 对象时，存在两个问题：
1. `retriever` 字段是必需的，但是在代码中并没有提供。
2. `vectorstore` 字段不被允许，可能是因为传递了一个不期望的参数。

`RetrievalQA` 类需要一个 `retriever` 参数，这个参数通常是一个实现了检索功能的对象。在这段代码中，可能想要使用 `docsearch` 作为检索器，但是 `docsearch` 是一个 `Chroma` 对象，它可能不符合 `RetrievalQA` 期望的 `retriever` 类型。
为了解决这个问题，需要确保传递给 `RetrievalQA.from_chain_type` 的参数是正确的。根据 `langchain` 的文档，`RetrievalQA.from_chain_type` 应该接受一个 `retriever` 参数，而不是 `vectorstore`。
这里是一个修改后的示例，它使用 `docsearch.as_retriever()` 来创建一个符合要求的检索器：
```python
# 创建问答对象
qa = RetrievalQA.from_chain_type(
    llm=hf_model,
    chain_type="stuff",
    retriever=docsearch.as_retriever(),
    return_source_documents=True
)
```


