Tags: python redis

今天在工作中发现，某些有顺序的hash记录，取出来却不能保证顺序。
后来发现是自己的疏忽，并不是redis的问题

## Redis的hash顺序是按照存入hash的先后保存的。只是Python的Hash无法保证顺序

这点让我疏忽了, 所以在存入的时候我用了 hmset.

# 如何存

看下面两种情况：

    ```python
    r = redis.Redis()
    r.hmset('test', {100:1, 104:2, 101:3, 89: 4})

    for index, value in enumerate([100, 104, 101, 89]):
        r.hset('test1', value, index+1)
    ```

然后到 redis-cli 中看看这两个key的情况：

![redis-cli](http://i1297.photobucket.com/albums/ag23/yueyoum/xx_zps76ffb3fe.png)

就如同我们上面所说，这其实是python字典无法保证顺序导致。


# 如何取

既然不能用 hmset, 那自然也不能用 hgetall 
得用 hkeys / hvals 来取

![redis hash](http://i1297.photobucket.com/albums/ag23/yueyoum/xx_zps1714d7d0.png)

