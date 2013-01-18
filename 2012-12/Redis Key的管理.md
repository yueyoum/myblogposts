Tags: redis, python

Redis就不用介绍了，这里简单说一点自己在开发中的心得.

在程序中多个地方都可能对同一个key进行操作，
所以将这些key统一管理起来是很有必要的。
否则如果是每个地方都自己拼接key，
一个地方修改，那就要整个项目的去找使用此key的地方。
虽然Linux中有 **grep**, **sed** 这样的工具，
但已经隐约感到一阵蛋疼。

统一管理，然后各处引用。还有一个好处就是，
引用名可以起一些很明了的名字。然后真正的key很短。
比如这样：

    ```python
    api_called_times = 'act'
    player_info = lambda p: 'p.{0}'.format(p)
    ```

然后在代码里就不会看着 act 不知道是什么意思，
还得去看文档。

把真实的key设计的很短，是有依据的。[Instagram的Redis实践][instagram]


## 可能会遇到的问题

随着系统越来越大，key越来越多，怎么才能保证后面新加入的key，
和前面的key**不重名**？因为在这种很短的缩写时，起名相同的碰撞是很容易的。


目前我的解决办法就是在 key 的管理文件中，加入一段检查代码。
每次加入新key后，就运行一边。代码片段如下：


    ```python
    #不用传参的key
    api_called_times = 'act'
    
    # 需要参数的key
    player = lambda p: 'p.{0}'.format(p)
    player_levels = lambda p, lv: 'p.{0}.{1}'.format(p, lv)


    if __name__ == '__main__':
        # 每次增加新key后， 运行本文件，检查是否有key重名
        __all_keys = globals()
        __all_keys_str = []
        from types import FunctionType as __FunctionType
        from inspect import getargspec as __getargspec
        for k, v in __all_keys.items():
            if k.startswith('__'):
                continue
            if isinstance(v, __FunctionType):
                args_count = len(__getargspec(v).args)
                args = [0 for i in range(args_count)]
                v = v(*args)
            if v in __all_keys_str:
                raise Exception("Duplicate key: %s" % v)
            __all_keys_str.append(v)
    ```

运行文件时如果抛出异常，则有key重名。

这里需要注意的是，你不确定这个key是否需要传参，以及参数的个数。
上面的代码是通过这样的判断来完成key的生成的：

1.  key是不是FunctionType.如果不是，就直接比较key
2.  如果是，就通过inspect.getargspec确定参数个数，然后全部用0来作为参数最终获得的key再进行比较


[instagram]: http://blog.csdn.net/hewy0526/article/details/7444329
