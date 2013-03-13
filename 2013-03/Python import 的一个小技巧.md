Tags: python

# 常见的需求

一个项目中，N多文件使用了Python标准的json库，
但现在我想换成更快的第三方 ujson 库。

最常见的应该就是这种处理办法了：

    ```python
    try:
        import ujson as json
    except ImportError:
        import json
    ```


但现在我想偷懒，不想去一个文件一个文件的修改。

当然使用 **sed** 一样可以快速的完成这个任务：

    ```bash
    sed -i 's/import json/try:\n    import ujson as json\nexcept ImportError:\n    import json/' *.py
    ```

但这次我想来点不一样的。


# Python模块导入

当要 import 一个模块的时候，python尝试按照如下顺序导入：

1.  如果sys.modules中有这个模块，就直接使用
2.  如果没有，那么就做一次真正的导入，并加入sys.modules字典中

这也就是为什么第二次导入相同模块时非常快的原因。

下面这个实验：

    ```python
    >>> import sys
    >>> 'urllib' in sys.modules
    False
    >>> import urllib
    >>> 'urllib' in sys.modules
    True
    ```


# 最终的解决办法

从上面可以知道，如果 'json' 在sys.modules中已经存在，那么后面再 import json,
就直接从 `sys.modules['json']` 来取。 这里用 hashlib 和 json 来做个实验

    ```python
    >>> import sys
    >>> sys.modules['json'] = __import__('hashlib')
    >>> import json
    >>> json
    <module 'hashlib' from '/usr/lib/python2.7/hashlib.pyc'>
    >>> json.md5('aa').hexdigest()
    '4124bc0a9335c27f086f24ba207a4912'
    ```

