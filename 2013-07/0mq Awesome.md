Tags: zeromq

以前就知道 [zeromq][1]，但没研究过，只是在项目里简单使用了 **REQ**, **REP** 模式。

这个模式和传统的socket很相似，一个bind，一个connect，
只不过 zeromq 很自由，不管是 server， 还是client 都可以bind或者connect，
当然，只能是一方bind。

使用起来很方便，我也不写hello world的例子了，网上很多。
现在讲一讲我是如果使用它构建了一个容易伸缩的网络架构。

## 实际问题

我们的前端是一个web server, 一般没必要解耦的操作直接在web中完成了。
但有个任务，用户可能会频繁的请求，而这部分和web本身的关系不大，于是我们将其拆开
作为一个独立的服务，web通过zeromq来请求这个服务。所以最开始的结构就是这样：

	web REQ <----> REP service

1.	**service** 设置socket类型为 **REP**, 然后bind在某个地址，等待请求
2.	**web** 设置socket类型为 **REQ** , connect到 service的地址上。
3.	**web** 收到相应请求，就组装数据，发送给 service，然后web 调用socket.recv()，block住。
4.	**service** 计算完成后，返回数据，等待下个请求
5.	**web** recv获得数据，处理后再返回client

这是最典型的请求应答模型，为了提高 **service** 的处理速度，相应的办法也很多，比如：

*	`apache`, `gunicorn`, `uwsgi` 这样的 Master, slave 进程结构。

	Master不负责计算，只负责管理slave进程。

*	`twisted`, `tornado` 的异步IO

	但异步IO毕竟还是单进程，而且callback不直观

*	`erlang`, `gevent` 这种的协程模型


线程就别说了，小心锁把你的小JJ锁住了


## 用zeromq来解决

这几天学习了下zeromq后，发现有两个模型很适合这种场景，而且使用简单

1.	一个REQ, 多个REP， 这样zeromq会自动负载均衡

	看上去很完美，但这种方式有一个最大问题，就是如果我后面新开 service 进程
	来进一步提高处理速度，那么必须想办法通知 web，req 要多连接一个新开的 service。

	如果不是用外部存储，保存开启的 service 情况，那么就 **必须** 重启web。
	修改代码，添加上新的REQ 地址

	如果使用像redis这样的外部存储，那么还必须设计一套规范：

	service开启的时候如何将自己的IP:PORT存入redis，
	web在每次要请求service之前，如何从redis中读取service的状态。

	并且这种方式还有个问题就是：每开启一个新的service，就必须要占用了一个端口，
	总之，这种方法实现简单，但 **可维护性**, **可扩展性** 都不是太好。所以放弃

2.	PUSH, PULL 模型

	[这篇文章][2] 解释的比较清楚，并且网上也有PUSH PULL的例子，
	自己再做做试验，很快就能理解。


我就直接说我的处理方式吧，我使用了 PUSH/PULL 模型

![PUSHPULL][3]


client就是上面说的web， 以前直接把 worker 作为 service 在跑，
现在这些 worker都位于 route 的后面， web直接链接的是 route。

这种架构对于web而言是透明的，它根本不知道也不用关心自己链接的是谁。

route 要bind三个端口：

*	用于web链接的端口， REQ
*	PUSH数据给worker的端口，PUSH
*	接收worker PUSH出来的数据端口 PULL

worker 就要connect两个端口:

*	worker的PUSH端口，PULL
*	worker的PULL端口，PUSH


处理流程大致如下：

1.	web发送请求
2.	route收到后，将其通过PUSH端口 push 出去
3.	worker得到数据，进行处理后，自己再将其PUSH 出去，
4.	route 会pull得到 worker处理完毕的数据，最好将其通天REQ端口发回client



#### 好处

*	容易扩展, 只要启动新的worker的就可以。不用新指定端口，不用重启其他服务。
*	容错性好，一个worker挂了，请求会自动分发到其他worker，继续工作
*	容易部署，使用 [supervisord][4]



[1]: http://www.zeromq.org/
[2]: https://learning-0mq-with-pyzmq.readthedocs.org/en/latest/pyzmq/patterns/pushpull.html
[3]: http://i1297.photobucket.com/albums/ag23/yueyoum/tu_zps35847949.png
[4]: http://supervisord.org/