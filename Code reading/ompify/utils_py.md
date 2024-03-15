# utils_py 详解
---


## `tree_to_token_index`函数 和 `index_to_code_token` 函数 详解

```python
def tree_to_token_index(root_node):
    if (len(root_node.children)==0 or root_node.type=='string') and root_node.type!='comment':
        return [(root_node.start_point,root_node.end_point)]
    else:
        code_tokens=[]
        for child in root_node.children:
            code_tokens+=tree_to_token_index(child)     # list类型是可以直接相加的
        return code_tokens


def index_to_code_token(index,code):
    start_point=index[0]
    end_point=index[1]
    if start_point[0]==end_point[0]:
        s=code[start_point[0]][start_point[1]:end_point[1]]
    else:
        s=""
        s+=code[start_point[0]][start_point[1]:]
        for i in range(start_point[0]+1,end_point[0]):
            s+=code[i]
        s+=code[end_point[0]][:end_point[1]]   
    return s
```

这两个函数`tree_to_token_index`和`index_to_code_token`一起工作，用于将代码的抽象语法树（AST）转换为令牌（token）序列，这是理解和操作源代码的重要步骤。下面是这两个函数的详细解释：

### `tree_to_token_index`函数

这个函数的目的是递归遍历抽象语法树（AST），并收集所有叶子节点（即实际的代码令牌）的位置信息。AST是代码结构的树形表示，其中每个节点代表代码的一个构造，例如一个语句、表达式或一个操作符。叶子节点通常代表代码中的基本元素，如变量名、常量、关键字等。

- **参数**：
  - `root_node`：当前递归遍历到的AST节点。

- **逻辑**：
  - 如果当前节点是叶子节点（即没有子节点），且节点类型不是`comment`（注释），则直接返回该节点的开始和结束位置（`start_point`和`end_point`）。如果节点类型是`string`，即使它有子节点也将其视为叶子节点，因为字符串通常被视为不可分割的单元。
  - 如果当前节点不是叶子节点，函数则遍历该节点的所有子节点，并递归调用自身，收集所有子节点的位置信息。
  - 最后，函数返回包含所有叶子节点位置信息的列表。

### `index_to_code_token`函数

这个函数的目的是根据提供的位置信息（即由`tree_to_token_index`函数返回的开始和结束位置），从原始代码中提取相应的令牌文本。

- **参数**：
  - `index`：一个包含开始位置和结束位置的元组。位置用行号和列号表示。
  - `code`：原始代码的行列表，每个元素代表代码的一行。

- **逻辑**：
  - 首先，函数根据开始位置（`start_point`）和结束位置（`end_point`）确定令牌的范围。
  - 如果令牌在同一行内，直接根据开始列号和结束列号提取该行内的字符串。
  - 如果令牌跨越了多行，函数则从开始列到第一行的末尾提取字符串，然后遍历每一行直到结束行，最后从结束行的开始到结束列提取字符串，并将这些字符串拼接起来。
  - 最后，函数返回提取的令牌字符串。

通过这两个函数，我们可以从AST中提取出所有代码令牌的文本，这对于后续的代码分析和处理非常重要，因为它允许我们基于代码的实际文本内容进行操作，而不仅仅是基于其结构。


### 举例解释 `index_to_code_token` 的逻辑

假设我们有以下的Python代码片段，存储在名为`code`的变量中，这里`code`是一个字符串列表，每个元素代表代码的一行：

```python
code = [
    "def my_function(x):",  # Line 0
    "    if x > 0:",        # Line 1
    "        return 'positive'",  # Line 2
    "    else:",            # Line 3
    "        return 'negative'"   # Line 4
]
```

现在，假设我们想要提取`if x > 0:`这段代码的令牌文本。根据AST和`tree_to_token_index`函数的输出，我们可能得到这段代码的开始和结束位置信息，这里用`(start_line, start_column), (end_line, end_column)`表示。对于`if x > 0:`，假设我们得到的位置信息是：

```python
index = ((1, 4), (1, 11))
```

这表示令牌开始于第1行的第4列（注意，在Python中索引从0开始，所以第1行实际上是代码的第2行），结束于同一行的第11列。现在，我们将使用`index_to_code_token`函数来提取这个令牌的文本：

```python
start_point = index[0]  # (1, 4)
end_point = index[1]    # (1, 11)

# 因为令牌在同一行内，我们可以直接根据开始和结束列号提取文本：
s = code[start_point[0]][start_point[1]:end_point[1]]
```

在这个例子中，`start_point[0]`是1，代表代码的第2行；`start_point[1]`是4，`end_point[1]`是11，代表我们要从第5个字符提取到第11个字符（包括起始，不包括终止位置）。因此，根据`code`列表，我们提取的文本是：

```python
s = "if x > 0:"
```

这正是我们期望提取的令牌文本。

现在，让我们考虑一个跨多行的例子。假设我们要提取的令牌是整个函数定义：

```python
index = ((0, 0), (4, 21))
```

这表示令牌开始于第0行的第0列，结束于第4行的第21列。因为这个令牌跨越了多行，我们需要分段提取文本，然后将这些部分拼接起来。根据`index_to_code_token`函数的逻辑：

```python
# 从第0行的第0列开始提取到第0行末尾
s = code[0][0:]  
# 依次拼接第1行到第3行的完整内容
for i in range(1, 4):
    s += code[i]
# 拼接第4行的开始到第21列的内容
s += code[4][:21]
```

通过这种方式，我们能够提取从第一行的`def my_function(x):`一直到最后一行的`'negative'`的整个函数定义。

希望这个例子能帮助你更好地理解`index_to_code_token`函数的工作原理。