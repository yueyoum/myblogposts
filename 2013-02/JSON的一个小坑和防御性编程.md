Tags: python

自以为对Python的常用内置库还算熟悉，
但最近就遇到了个坑。


    ```python
    import json

    data = {1: 2, 2: 3, 3: 4}
    serialized_data = json.dumps(data)

    # 然后保存 serialized_data,
    # 在需要的时候取出 serialized_data, 反序列化后，得到字典来进一步处理

    key = 1
    data = json.loads(serialized_data)
    value = data.get(key, 0)
    return value
    ```

就如同上面这段简单的代码，意图也很明了，但在测试中发现 value 总返回0 !

# json序列化dict

这个坑就在于我想当然的以为 loads后，肯定和 dumps 前的是一样的。
但对于 key 为 int 的dict却并不如此：


    ```python
    >>> a = {1: 2}
    >>> json.dumps(a)
    '{"1": 2}'
    >>> json.loads( json.dumps(a) )
    {u'1': 2}
    ```

上面的示例说明了一切，loads过后， 原本int的key，变成了string。
所以value总是0.

这里也引申出另一个问题

# 防御性编程

在开发阶段，不要做太多的错误处理，尽可能的让程序出错，才能尽快的发现问题

如果我上面没用 get 方法，而是直接 `data[key]`， 那么就不会在客户端得到0的
时候去到其他地方排错。

