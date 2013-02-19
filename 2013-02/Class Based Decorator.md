Tags: python

之前在写装饰器的时候，基本都是写一个函数。
大概像这样：

    ```python
    def check(arg):
        def deco(func):
            @functools.wraps(func)
            def wrap(data, *args, **kwargs):
                # ......
                if data not valid:
                    raise MyException(1)
                return func(data, *args, **kwargs)
            return wrap
        return deco

    def error_handler(arg):
        def deco(func):
            @functools.wraps(func)
            def wrap(*args, **kwargs):
                try:
                    result = func(*args, **kwargs)
                except MyException, e:
                    result = e.args[0]
                return result
            return wrap
        return deco
    ```


这两个装饰器作用如同其名字，一个检测输入，另一个做错误处理。

    ```python
    @error_handler(...)
    @check(...)
    def my_func(data):
        # balabala...
        if balabala:
            raise MyException(10)

        return ...
    ```

所以上面这段代码的实际作用是，

1.  首先检测my_func的第一个位置参数data是否合法
2.  如果data不合法，那么就直接在 check 中raise 了异常, 并由 error_handler 捕获
3.  如果data合法，就执行my_func。如果my_func raise了异常，同样由 error_handler 捕获异常的第一个参数作为返回
4.  my_func顺利执行，就把 my_func 的结果返回


本来一起运行良好。但在后的实现中发现需要修改返回方式，不能直接返回。
需要由另一个返回网关统一管理，这个网关需要一个唯一标识码，这个标识码从 my_func 的第一个位置参数 data 可以取出

但现在如果保持这种两个函数的装饰器，不太方便达到这个目的。
因为我不想再在 error_handler 中解析一遍 data, 因为 data 已经在 check 和 my_func 中至少解析两遍了。

我开始用 raise MyException(10, SIGN) 这种方法将这个标识码传递到 error_handler 中。
但发现这样需要修改的地方太多。于是用了下面这种方法：


    ```python
    class Guard(object):
        def __init__(self, args):
            self.args = args

        def __call__(self, func):
            @functools.wraps(func)
            def deco(data, *args, **kwargs):
                try:
                    self._check(data)
                    func_res = func(data, *args, **kwargs)
                except MyException, e:
                    return self._error_response(e.args[0])

                # 这里用统一的网关返回数据
                # return xxx
            return deco


        def _error_response(self, ret_code):
            # 在这里使用 self.SIGN
            pass
                
        def _check(self, data):
            # 在这里得到标识码
            # self.SIGN = parse(data)
            if XXX:
                raise MyException(1)
            if YYY:
                raise MyException(2)

            #...
    ```


其实现在也就是把两个分开的装饰器，合并成了一个。
并且用类来保持状态。（保存标识码）


现在的使用情景就变成了这样：


    ```python
    @Guard(...)
    def my_func(data):
        # balabala...
        if balabala:
            raise MyException(10)

        return ...
    ```



而项目中以前留下的大量 raise MyException 就不用去添加而外的参数了。


并且现在还有一个方便之处，遇到需要特殊 check 的地方，只要用一个子类去继承 Guard 就好。
比如这样：

    ```python
    class NoNeedCheckGuard(Guard):
        def _check(self, data):
            pass



    @NoNeedCheckGuard(...)
    def this_func_no_not_need_check_arguments(data):
        pass
    ```


