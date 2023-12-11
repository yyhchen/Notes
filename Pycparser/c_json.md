# pycparser: *c_json.py*

---
**说明：**

这段注释是关于如何在Python中使用JSON格式对抽象语法树（AST）进行序列化（**将AST对象转换为JSON字符串**）和反序列化（**将JSON字符串转换为AST对象**）的说明。

具体来说，这段注释描述了两个主要功能：

1. **序列化（Serialization）：** 将AST对象转换为JSON格式。这包括遍历AST树，将每个节点从Python的Node对象转换为一个Python字典。每个节点的属性都被映射到字典中的相应键值对。其中，特别使用了一个名为 `_nodetype` 的元数据字段，表示节点的类名。对于坐标信息（表示在源代码中的位置），使用 `coord` 字段，将其转换为字符串的格式为 "filename:line[:column]"。

2. **反序列化（Deserialization）：** 将JSON格式转换回AST对象。这涉及到相反的转换，即从JSON字符串中遍历生成的树，并将每个字典转换回特定的Node对象。在这个过程中，使用 `_nodetype` 字段确定应该实例化哪个具体的AST节点类。坐标信息的字符串形式也会被转换回Coord对象。

最后，注释提供了一个具体的例子，展示了如何表示一个TypeDecl节点以及其子节点IdentifierType的JSON表示形式。这个例子说明了JSON格式如何直接映射到AST对象的属性和结构，使得在序列化和反序列化过程中能够保留节点之间的关系和结构。


<br>
<br>
<br>


**示例代码：**

```python{.line-numbers}
import json
import sys
import re

sys.path.extend(['.', '..'])

from pycparser import parse_file, c_ast
from pycparser.plyparser import Coord


RE_CHILD_ARRAY = re.compile(r'(.*)\[(.*)\]')
RE_INTERNAL_ATTR = re.compile('__.*__')


class CJsonError(Exception):
    pass


def memodict(fn):
    """ Fast memoization decorator for a function taking a single argument """
    class memodict(dict):
        def __missing__(self, key):
            ret = self[key] = fn(key)
            return ret
    return memodict().__getitem__


@memodict
def child_attrs_of(klass):
    """
    Given a Node class, get a set of child attrs.
    Memoized to avoid highly repetitive string manipulation

    """
    non_child_attrs = set(klass.attr_names)
    all_attrs = set([i for i in klass.__slots__ if not RE_INTERNAL_ATTR.match(i)])
    return all_attrs - non_child_attrs


def to_dict(node):
    """ Recursively convert an ast into dict representation. """
    klass = node.__class__

    result = {}

    # Metadata
    result['_nodetype'] = klass.__name__

    # Local node attributes
    for attr in klass.attr_names:
        result[attr] = getattr(node, attr)

    # Coord object
    if node.coord:
        result['coord'] = str(node.coord)
    else:
        result['coord'] = None

    # Child attributes
    for child_name, child in node.children():
        # Child strings are either simple (e.g. 'value') or arrays (e.g. 'block_items[1]')
        match = RE_CHILD_ARRAY.match(child_name)
        if match:
            array_name, array_index = match.groups()
            array_index = int(array_index)
            # arrays come in order, so we verify and append.
            result[array_name] = result.get(array_name, [])
            if array_index != len(result[array_name]):
                raise CJsonError('Internal ast error. Array {} out of order. '
                    'Expected index {}, got {}'.format(
                    array_name, len(result[array_name]), array_index))
            result[array_name].append(to_dict(child))
        else:
            result[child_name] = to_dict(child)

    # Any child attributes that were missing need "None" values in the json.
    for child_attr in child_attrs_of(klass):
        if child_attr not in result:
            result[child_attr] = None

    return result


def to_json(node, **kwargs):
    """ Convert ast node to json string """
    return json.dumps(to_dict(node), **kwargs)


def file_to_dict(filename):
    """ Load C file into dict representation of ast """
    ast = parse_file(filename, use_cpp=True)    # True是进行预处理#include 和 #define等，False则直接解析成AST
    return to_dict(ast)


def file_to_json(filename, **kwargs):
    """ Load C file into json string representation of ast """
    ast = parse_file(filename, use_cpp=True)
    return to_json(ast, **kwargs)


def _parse_coord(coord_str):
    """ Parse coord string (file:line[:column]) into Coord object. """
    if coord_str is None:
        return None

    vals = coord_str.split(':')
    vals.extend([None] * 3)
    filename, line, column = vals[:3]
    return Coord(filename, line, column)


def _convert_to_obj(value):
    """
    Convert an object in the dict representation into an object.
    Note: Mutually recursive with from_dict.

    """
    value_type = type(value)
    if value_type == dict:
        return from_dict(value)
    elif value_type == list:
        return [_convert_to_obj(item) for item in value]
    else:
        # String
        return value


def from_dict(node_dict):
    """ Recursively build an ast from dict representation """
    class_name = node_dict.pop('_nodetype')

    klass = getattr(c_ast, class_name)

    # Create a new dict containing the key-value pairs which we can pass
    # to node constructors.
    objs = {}
    for key, value in node_dict.items():
        if key == 'coord':
            objs[key] = _parse_coord(value)
        else:
            objs[key] = _convert_to_obj(value)

    # Use keyword parameters, which works thanks to beautifully consistent
    # ast Node initializers.
    return klass(**objs)


def from_json(ast_json):
    """ Build an ast from json string representation """
    return from_dict(json.loads(ast_json))


#------------------------------------------------------------------------------
if __name__ == "__main__":
    if len(sys.argv) > 1:
        # Some test code...
        # Do trip from C -> ast -> dict -> ast -> json, then print.
        ast_dict = file_to_dict(sys.argv[1])
        ast = from_dict(ast_dict)
        print(to_json(ast, sort_keys=True, indent=4))
    else:
        print("Please provide a filename as argument")

```

<br>
<br>
<br>

## 部分函数解释

### memodict装饰器
```python
def memodict(fn):
    """ Fast memoization decorator for a function taking a single argument """
    class memodict(dict):
        def __missing__(self, key):
            ret = self[key] = fn(key)   # 如果通过key找不到值，那就返回一个 fn(key)的值
            return ret
    return memodict().__getitem__       # 如果能够通过key找到值，那就直接返回这个key对应的值
```

**使用方法如下：**
这是一个使用类装饰器实现的简单记忆化（memoization）装饰器，*用于缓存函数的计算结果以提高性能*。它的使用方式如下：

```python
@memodict
def expensive_computation(n):
    print(f"Computing {n}!")
    return n * n

# 第一次调用，会计算并缓存结果
result1 = expensive_computation(5)
print("Result 1:", result1)

# 第二次调用相同的参数，直接从缓存中获取结果！！！！（因为找到了不会调用__missing__函数，而是直接根据key找value）
result2 = expensive_computation(5)
print("Result 2:", result2)

# 不同的参数，会重新计算并缓存结果
result3 = expensive_computation(10)
print("Result 3:", result3)
```

在这个例子中，`expensive_computation` 是一个需要计算成本较高的函数。通过应用 `@memodict` 装饰器，函数的计算结果将被缓存，以避免重复计算相同的参数。

**输出将会是：**

```
Computing 5!
Result 1: 25
Result 2: 25
Computing 10!
Result 3: 100
```

在第一次调用时，由于参数 5 的结果未被缓存，因此会计算并缓存结果。在第二次调用时，由于参数 5 已经被缓存，直接从缓存中获取结果，而不会重新计算。对于不同的参数（例如，10），会进行新的计算并缓存结果。这种方式可以在某些情况下提高函数的性能。


<br>
<br>
<br>



### child_attrs_of 函数
```python
@memodict
def child_attrs_of(klass):
    """
    Given a Node class, get a set of child attrs.
    Memoized to avoid highly repetitive string manipulation

    """
    non_child_attrs = set(klass.attr_names)     # 这一句看不懂要干嘛; 获取给定AST节点类的所有本地属性（不包括子节点）
    print("non_child_attrs", non_child_attrs)
    all_attrs = set([i for i in klass.__slots__ if not RE_INTERNAL_ATTR.match(i)])  # 去掉有下划线的属性
    print(klass.__slots__)  # 所有属性, __slots__是定义ast类的时候预定义的
    print('all_attrs', all_attrs)
    return all_attrs - non_child_attrs  # 减去这些没子节点的节点
```

就比如：
```
non_child_attrs {'declname', 'quals', 'align'}
klass.__slots__ ('declname', 'quals', 'align', 'type', 'coord', '__weakref__')
all_attrs {'coord', 'declname', 'quals', 'type', 'align'}
```


<br>
<br>
<br>


### to_dict 函数（感觉是核心函数）
```python
def to_dict(node):
    """ Recursively convert an ast into dict representation. """
    klass = node.__class__
    # print("node.__class__", klass)  # 类似于node.__class__ <class 'pycparser.c_ast.FileAST'> node.__class__ <class 'pycparser.c_ast.FuncDef'>

    result = {}

    # Metadata
    result['_nodetype'] = klass.__name__    # 节点类型（有哪些）
    # print("klass.__name__", klass.__name__)     # FileAST, FuncDef, Compound ...

    print('for start')
    print("node: ", node)
    print("klass.attr_names: ", klass.attr_names)  # 其实就是找当前层的节点是否是叶子节点！！
    # Local node attributes
    for attr in klass.attr_names:
        result[attr] = getattr(node, attr)      # node 是ast树; result[attr]是当前节点下，key(也就是attr)对应的value
        print('attr: ', attr)
        print("result[attr]: ", result[attr])
    print('for end')
    # Coord object
    if node.coord:
        result['coord'] = str(node.coord)
    else:
        result['coord'] = None

    print('node.children(): ', node.children())     # 当前节点孩子，比如根节点FileAST有两个孩子节点 FuncDef(decl=Decl(name='foo', 和 FuncDef(decl=Decl(name='main',
    # Child attributes
    for child_name, child in node.children():   # 准确的说，应该是，当前节点的非叶子节点的孩子节点
        print("child_name: ", child_name)   # 比如 ext[0], 这里的孩子节点都是非叶子节点 （key）
        print('child：', child)   # 比如 FuncDef(decl=Decl(name='foo', ... （value）      ==》 组合原来是ext=[FuncDef(decl=Decl(name='foo', ...

        # Child strings are either simple (e.g. 'value') or arrays (e.g. 'block_items[1]')
        match = RE_CHILD_ARRAY.match(child_name)    # 匹配形如 some_name[index] 的字符串
        print('match: ', match)
        if match:
            array_name, array_index = match.groups()    # 一般是相同类型的非叶子孩子节点才有
            print('match.groups():', match.groups())
            array_index = int(array_index)
            # arrays come in order, so we verify and append.
            result[array_name] = result.get(array_name, [])     # 如果能通过array.name拿到值，就返回该值，否则返回 []
            print("result[array_name] before: ", result[array_name])
            if array_index != len(result[array_name]):
                raise CJsonError('Internal ast error. Array {} out of order. '
                    'Expected index {}, got {}'.format(
                    array_name, len(result[array_name]), array_index))
            result[array_name].append(to_dict(child))   # 继续访问该节点的孩子节点(深度遍历)
            print("result[array_name] after: ", result[array_name])
        else:
            result[child_name] = to_dict(child)

    # Any child attributes that were missing need "None" values in the json.
    for child_attr in child_attrs_of(klass):
        print('child_attrs_of(klass): ', child_attrs_of(klass))     # 获取当前节点的所有属性
        print('child_attr: ', child_attr)
        if child_attr not in result:
            result[child_attr] = None
    print("最后结果result: ", result)
    return result
```

**说明：**
按照上面的`print()`设置，可以清楚知道代码在干什么，总的来说就是 递归访问孩子节点，构建 `dict`


<br>
<br>
<br>


### to_json 函数
```python
def to_json(node, **kwargs):
    """ Convert ast node to json string """
    """
        将 to_dict 处理好的dict类型数据 变成 json 文件格式
    """
    return json.dumps(to_dict(node), **kwargs)      # 因为to_dict已经是 dict类型类似json格式，故不需要指定如何转换

```



<br>
<br>
<br>




### _parse_coord函数
```python
def _parse_coord(coord_str):
    """ Parse coord string (file:line[:column]) into Coord object. """
    if coord_str is None:
        return None

    vals = coord_str.split(':')
    vals.extend([None] * 3)
    filename, line, column = vals[:3]
    # print('Coord(filename, line, column): ', Coord(filename, line, column))    # Coord(filename, line, column):  c_files/basic.c:1:5
    # print(type(Coord(filename, line, column)))  # <class 'pycparser.plyparser.Coord'>
    return Coord(filename, line, column)
```



<br>
<br>
<br>



### _convert_to_obj函数
```python
def _convert_to_obj(value):
    """
    Convert an object in the dict representation into an object.
    Note: Mutually recursive with from_dict.

    """
    value_type = type(value)
    if value_type == dict:
        return from_dict(value)
    elif value_type == list:
        return [_convert_to_obj(item) for item in value]
    else:
        # String or NoneType
        # print('value type: ', type(value))  # value type:  <class 'str'>; value type:  <class 'NoneType'>
        return value
```



<br>
<br>
<br>




### from_dict函数
```python
def from_dict(node_dict):
    """ Recursively build an ast from dict representation """
    class_name = node_dict.pop('_nodetype')
    # print('class_name: ', class_name)   # FileAST, FuncDef, Compound ...

    klass = getattr(c_ast, class_name)  # 过去c_ast实例中的class_name属性对应的值，即，class_name作为key 对应的value值是 klass
    print('klass', klass)   # klass <class 'pycparser.c_ast.FileAST'> ..

    # Create a new dict containing the key-value pairs which we can pass
    # to node constructors.
    objs = {}
    for key, value in node_dict.items():
        if key == 'coord':
            objs[key] = _parse_coord(value)     # 将value转变成为 coord类型
        else:
            objs[key] = _convert_to_obj(value)  # 将value转变为 str 对象类型

    # Use keyword parameters, which works thanks to beautifully consistent
    # ast Node initializers.
    return klass(**objs)

```


