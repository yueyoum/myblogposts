Tags: codebattle, erlang, unity3d

从玩法设计，到建模贴图；从学习Unity3d，到debug Server，
折腾了一个月， 把自己一直想做的东西终于折腾出来了。

AI 对战， 你写的AI 和 别人的AI 进行 对战。

先看演示示例：

[youtube][http://www.youtube.com/watch?v=V6PkjlUXV6w&feature=youtu.be] 
[youku][http://v.youku.com/v_show/id_XNTk0OTk2Mjg0.html]


CodeBattle 这个项目包含四个部分。

*   [server][https://github.com/yueyoum/codebattle-server]
*   [client][https://github.com/yueyoum/codebattle-client]
*   [proto][https://github.com/yueyoum/codebattle-proto]
*   [ai][https://github.com/yueyoum/codebattle-ai]

client 是一个unity3d的项目，它用来建立房间，以及显示AI的操作.

proto ， client 和 server 之间的数据交互 使用了 google protobuf 来序列化和反序列化数据.

ai 这个就是此项目的重头戏， 你根据数据交互格式，以及游戏规则写自己的AI，来和别人的AI对战。


**大家主要看的是 proto 和 ai 这两个repo**
proto 定义的数据格式， ai 里面有游戏规则，以及示例ai。

#### 如何开始：
在 client repo 中找到 编译好的 client 下载链接， 下载解压后运行。
如果你使用 我提供的server，那么 默认的 ip 和 port 不用更改，直接 create room。
你就会到达一个新场景，顶部是room id， 但场景中没有一个 marine。

这需要你的AI 加入这个room， 才会为你的AI创建marines， 当两个AI都加入同一个房间后，
对战就开始了。

刚开始你没有自己的AI，可以运行 ai 这个repo 中的示例AI，来感受一下整个流程。


当然 ，你也可以设计出新的玩法，然后 checkout server repo， 修改后在你本地运行。
client 中也包含了 unity3d 的项目文件， 你也可以 checkout 后 自行修改，添加自己想要的功能