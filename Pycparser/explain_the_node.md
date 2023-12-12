# 解释下在AST各个节点的意义
---


<br>

**以下方式均可获取到AST，但意义不一样**

* ast = parse_file(filename, use_cpp=False)
* ast = c_parser.CParser().parse(text, filename='<stdio>)


```C
int foo() {}

int main() {
  foo();
  return 0;
}
```

利用 `parse_file` 解析得到的 **ast** 结果如下：
```
FileAST(ext=[FuncDef(decl=Decl(name='foo',
                               quals=[
                                     ],
                               align=[
                                     ],
                               storage=[
                                       ],
                               funcspec=[
                                        ],
                               type=FuncDecl(args=None,
                                             type=TypeDecl(declname='foo',
                                                           quals=[
                                                                 ],
                                                           align=None,
                                                           type=IdentifierType(names=['int'
                                                                                     ]
                                                                               )
                                                           )
                                             ),
                               init=None,
                               bitsize=None
                               ),
                     param_decls=None,
                     body=Compound(block_items=None
                                   )
                     ),
             FuncDef(decl=Decl(name='main',
                               quals=[
                                     ],
                               align=[
                                     ],
                               storage=[
                                       ],
                               funcspec=[
                                        ],
                               type=FuncDecl(args=None,
                                             type=TypeDecl(declname='main',
                                                           quals=[
                                                                 ],
                                                           align=None,
                                                           type=IdentifierType(names=['int'
                                                                                     ]
                                                                               )
                                                           )
                                             ),
                               init=None,
                               bitsize=None
                               ),
                     param_decls=None,
                     body=Compound(block_items=[FuncCall(name=ID(name='foo'
                                                                 ),
                                                         args=None
                                                         ),
                                                Return(expr=Constant(type='int',
                                                                     value='0'
                                                                     )
                                                       )
                                               ]
                                   )
                     )
            ]
        )
```
<br>
<br>
<br>

---


**其中一些节点的解释如下：**

观察第一行，*发现有ext数组*，**ext数组表示的是==外部声明==** 举例展示ext用法:

`ast.ext[0]` 如下，其实就是一个 `foo` 函数的ast
```
FuncDef(decl=Decl(name='foo',
                  quals=[
                        ],
                  align=[
                        ],
                  storage=[
                          ],
                  funcspec=[
                           ],
                  type=FuncDecl(args=None,
                                type=TypeDecl(declname='foo',
                                              quals=[
                                                    ],
                                              align=None,
                                              type=IdentifierType(names=['int'
                                                                        ]
                                                                  )
                                              )
                                ),
                  init=None,
                  bitsize=None
                  ),
        param_decls=None,
        body=Compound(block_items=None
                      )
        )
```

<br>

`ast.ext[1]` 如下，其实就是第二个函数 `main` 函数的ast
```
FuncDef(decl=Decl(name='main',
                  quals=[
                        ],
                  align=[
                        ],
                  storage=[
                          ],
                  funcspec=[
                           ],
                  type=FuncDecl(args=None,
                                type=TypeDecl(declname='main',
                                              quals=[
                                                    ],
                                              align=None,
                                              type=IdentifierType(names=['int'
                                                                        ]
                                                                  )
                                              )
                                ),
                  init=None,
                  bitsize=None
                  ),
        param_decls=None,
        body=Compound(block_items=[FuncCall(name=ID(name='foo'
                                                    ),
                                            args=None
                                            ),
                                   Return(expr=Constant(type='int',
                                                        value='0'
                                                        )
                                          )
                                  ]
                      )
        )
```
<br>

举个例子访问 `ext[]` 里面的结构，比如选择`ext[1]`

分别访问 `ext[1]` 中的节点 `decl`, `param_decls`, `body`, **结果如下：**
```
Decl(name='main',
     quals=[
           ],
     align=[
           ],
     storage=[
             ],
     funcspec=[
              ],
     type=FuncDecl(args=None,
                   type=TypeDecl(declname='main',
                                 quals=[
                                       ],
                                 align=None,
                                 type=IdentifierType(names=['int'
                                                           ]
                                                     )
                                 )
                   ),
     init=None,
     bitsize=None
     )
```

```
None
```

```
Compound(block_items=[FuncCall(name=ID(name='foo'
                                       ),
                               args=None
                               ),
                      Return(expr=Constant(type='int',
                                           value='0'
                                           )
                             )
                     ]
         )
```

**也就是说，只要有 `=`，且在同一层级上，就可以通过 `=` 前面的属性值进行遍历访问 (树的深度遍历)**




<br>
<br>
<br>

在`pycparser`库的`Decl`节点中，比如 `ext[1]` 中的 `decl` 属性，以下是这些属性的含义：

1. **`quals` (Qualifiers):**
   - **Type:** List of strings
   - **Description:** 表示声明中的类型限定符，例如 `const`, `volatile` 等。这是一个字符串列表，因为一个声明可能具有多个类型限定符。

2. **`align` (Alignment):**
   - **Type:** int or None
   - **Description:** 表示声明的对齐方式。如果对齐方式未指定，则为`None`。对于某些体系结构，可以在声明中指定对齐方式，以确保数据在内存中的对齐。

3. **`storage` (Storage Class Specifiers):**
   - **Type:** List of strings
   - **Description:** 表示声明的存储类说明符，例如 `auto`, `register`, `static`, `extern` 等。这是一个字符串列表，因为一个声明可能具有多个存储类说明符。

4. **`funcspec` (Function Specifiers):**
   - **Type:** List of strings
   - **Description:** 表示声明的函数说明符，例如 `inline`, `noreturn` 等。这是一个字符串列表，因为一个声明可能具有多个函数说明符。

5. **`type` (Type):**
   - **Type:** c_ast.Node
   - **Description:** 表示声明的类型。通常是`TypeDecl`、`PtrDecl`、`ArrayDecl`等`c_ast`模块中定义的类型节点。该属性指定了声明的基本类型及其修饰符，例如指针、数组等。

这些属性一起描述了一个声明的完整信息，包括类型及其限定符、对齐方式、存储类说明符、函数说明符等。在`pycparser`中，`Decl`节点用于表示C语言中的声明语句，如变量声明、函数声明等。


---

## 完整AST各个节点描述如下：

在提供的AST描述中，每个节点表示C语言源代码中的不同结构。以下是每个节点的含义：

1. **`FileAST`节点:**
   - **Description:** 表示整个C语言文件的抽象语法树的根节点。
   - **Attributes:**
      - `ext`: 包含了文件中所有外部声明的列表，每个元素都是一个外部声明（external declaration）。

2. **`FuncDef`节点 (Function Definition):**
   - **Description:** 表示C语言中的函数定义。
   - **Attributes:**
      - `decl`: 包含函数声明信息的节点，包括函数名、类型等。
      - `param_decls`: 函数参数的声明列表。
      - `body`: 包含函数体的节点，通常是一个 `Compound` 节点。

3. **`Decl`节点 (Declaration):**
   - **Description:** 表示C语言中的声明语句，可以是变量声明、函数声明等。
   - **Attributes:**
      - `name`: 声明的标识符名称。
      - `quals`: 类型限定符（type qualifiers）的列表。
      - `align`: 对齐方式的列表。
      - `storage`: 存储类说明符（storage class specifiers）的列表。
      - `funcspec`: 函数说明符（function specifiers）的列表。
      - `type`: 表示声明的类型，是一个 `c_ast` 模块中定义的类型节点。

4. **`FuncDecl`节点 (Function Declaration):**
   - **Description:** 表示函数声明的节点。
   - **Attributes:**
      - `args`: 表示函数参数的声明列表。
      - `type`: 表示函数的返回类型，是一个 `TypeDecl` 节点。

5. **`TypeDecl`节点 (Type Declaration):**
   - **Description:** 表示类型声明的节点，通常包括基本类型及其修饰符。
   - **Attributes:**
      - `declname`: 声明的标识符名称。
      - `quals`: 类型限定符的列表。
      - `align`: 对齐方式。
      - `type`: 表示基本类型，是一个 `IdentifierType` 节点。

6. **`IdentifierType`节点:**
   - **Description:** 表示标识符类型的节点，例如 `int`、`char` 等。
   - **Attributes:**
      - `names`: 包含类型名称的列表。

7. **`Compound`节点:**
   - **Description:** 表示C语言中的复合语句块，通常是函数体的一部分。
   - **Attributes:**
      - `block_items`: 包含在复合语句中的语句列表。

8. **`FuncCall`节点:**
   - **Description:** 表示函数调用的节点。
   - **Attributes:**
      - `name`: 被调用函数的标识符。
      - `args`: 表示函数调用的参数列表。

9. **`Return`节点:**
   - **Description:** 表示函数中的返回语句。
   - **Attributes:**
      - `expr`: 返回语句中的表达式。

此AST描述了一个包含两个函数定义的C语言源文件。第一个函数是 `foo`，它没有参数，没有函数体；第二个函数是 `main`，它调用了 `foo` 函数，并在函数体中包含了一个返回语句。

