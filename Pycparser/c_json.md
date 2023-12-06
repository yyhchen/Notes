# pycparser: *c-to-c.py*

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
    ast = parse_file(filename, use_cpp=True)
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