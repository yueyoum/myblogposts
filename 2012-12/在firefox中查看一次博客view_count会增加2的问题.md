Tags: firefox

## 问题情况

在开发本博客的时候，我一直用的google chrome，
webkit内核的浏览器自带的开发者工具太爽了。
所以虽然我的主浏览器是firefox，但是在开发网页的时候还是会用chrome

今天用firefox看已经发表的文章时，发现一个问题：
这篇博客每载入一次，view_count计数会增加2，而不是理所应当的1。

在 chrome 和 opera 都确定了 **只有** firefox才会引发此问题.


## 解决

**httpfox** 是我必安的firefox插件，同样，这次也打开httpfox，
查看当打开一篇blog时，到底都有哪些请求。

果然在httpfox中找到了对blog的两次请求

*   一次是用户手动点击，打开一篇blog的请求
*   一次是disqus发送的请求。（本blog使用disqus的服务）

在 debug 的时候，terminal中的输出也一清二楚：

![terminal][debug_in_terminal]


事情到了这里似乎已经很明了了，只要在blog入口判断 **Referer** ，
如果是disqus的，就不计view_count。


代码片段


    ```python
    from functools import wraps
    from bottle import request
    
    def forbid(**kwargs):
        lower_key_kwargs = {}
        for k, v in kwargs.items():
            lower_key_kwargs[k.lower()] = v
            
        def deco(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                if 'referer' in lower_key_kwargs:
                    if request.get_header('Referer', '').startswith(
                        lower_key_kwargs['referer']):
                        return ''
                return func(*args, **kwargs)
            return wrapper
        return deco
        
        
    # 在view函数上使用此装饰器即可，这里只做了referer的判断
    # 这样对应的view，如果referer是 http://disqus.com 开头，
    # 就直接返回空字符串，不执行view函数
    
    @app.get('/blog/<title>')
    @forbid(referer='http://disqus.com')
    @jinja_view('post.html')
    def show_post(title):
        pass
    ```



目前我是这么做的。但 **还不明白** 为什么只有firefox出现了这样的问题，
chrome 和 opera 都没有disqus的那次请求。

所以，有知道的同学请留言告诉我吧。



[debug_in_terminal]: http://i1297.photobucket.com/albums/ag23/yueyoum/22_shadowed_zps89b1c1c6.png "screenshot"

