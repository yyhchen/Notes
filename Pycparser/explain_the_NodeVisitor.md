# 解释下pycparser中c_ast.py文件中 NodeVisitor类怎么使用

---

### 在 `c_ast.py` 文件中 `NodeVisitor类如下`
```python{.line-numbers}
class NodeVisitor(object):
    """ A base NodeVisitor class for visiting c_ast nodes.
        Subclass it and define your own visit_XXX methods, where
        XXX is the class name you want to visit with these
        methods.

        For example:

        class ConstantVisitor(NodeVisitor):
            def __init__(self):
                self.values = []

            def visit_Constant(self, node):
                self.values.append(node.value)

        Creates a list of values of all the constant nodes
        encountered below the given node. To use it:

        cv = ConstantVisitor()
        cv.visit(node)

        Notes:

        *   generic_visit() will be called for AST nodes for which
            no visit_XXX method was defined.
        *   The children of nodes for which a visit_XXX was
            defined will not be visited - if you need this, call
            generic_visit() on the node.
            You can use:
                NodeVisitor.generic_visit(self, node)
        *   Modeled after Python's own AST visiting facilities
            (the ast module of Python 3.0)
    """

    _method_cache = None

    def visit(self, node):
        """ Visit a node.
        """

        if self._method_cache is None:
            self._method_cache = {}

        visitor = self._method_cache.get(node.__class__.__name__, None)
        if visitor is None:
            method = 'visit_' + node.__class__.__name__  # 这里拿到的是类名 就比如FuncDey, TypeDecl，FuncCall这样的
            visitor = getattr(self, method, self.generic_visit)   # 如果method匹配不到，就选择generic_visit方法；这句代码，是能够拿到visit_xx方法的关键
            self._method_cache[node.__class__.__name__] = visitor

        return visitor(node)

    def generic_visit(self, node):
        """ Called if no explicit visitor function exists for a
            node. Implements preorder visiting of the node.
        """
        for c in node:
            self.visit(c)
```


<br>
<br>
<br>


**在自己编写的程序中:**
```python{.line-numbers}
class ReturnVisitor(c_ast.NodeVisitor):
    def __init__(self, typo):
        self.typo = typo

    def visit_Return(self, node):
        print('node: ', node)
        print('type: ', node.expr.type)
        print('node.__class__.__name__: ', node.__class__.__name__)

text = r"""
int foo() {}

int main() {
  foo();
  return 0;
}
"""
node = c_parser.CParser().parse(text) # 这里还可以通过 node = parse_file('', use_cpp=False) 拿到ast

rv = ReturnVisitor('int')
rv.visit(node)

```
<br>

**运行结果如下：**
```
node:  Return(expr=Constant(type='int',
                     value='0'
                     )
       )
type:  int
node.__class__.__name__:  Return
```

<br>
<br>

可以看到，通过 `NodeVisitor` 中的 `visit` 方法 触发了我的 `ReturnVisitor` 类 中的 `visit_Return` 方法;

**核心代码是 ` visitor = getattr(self, method, self.generic_visit)` 这里的self是 `NodeVisitor` 的实例，也是能够拿到 `visit_xxx` 方法的关键**



<br>
<br>
<br>


