Tags: linux

我刚开始的时候用的是Ubuntu, 记得Ubuntu自带一个工具可以设置系统全局代理。
但我现在一直用Xfce4，虽然早已发现Xfce4没有提供工具来设置全局代理，
但也一直没在意，因为毕竟常用的需要联网的软件/程序都是可以设定代理的。


但最近github访问一直不稳定，刚才常规姿势又上不去了。

sublime text2 的 Package Control 是通过github来安装plugin的，
sublime只是一个编辑器，因此不可能提供设置代理的选项。
于是就找Xfce4中的系统全局代理设置方法。

google了一下，很容易就搜到了：
在 /etc/environment 中添加以下内容，就可以为系统制定全局的 http 代理

    ```bash
    http_proxy="http://127.0.0.1:8118"
    ```


上面是我自己的情况，我在本地的8118端口上开的有 http 代理。
你只要将其替换成你自己的host:port就行

