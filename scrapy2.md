### scrapy redis的分布式爬虫以及源码解析

###### 主要通过分析开源的scrapy resid来了解什么是分布式爬虫，以及如何将scrapy变成一个分布式的爬虫

> 分布式爬虫的好处

1. 充分利用多台机器的带宽来加速我们的爬取

   (任何单台服务器的带宽都是有限的)

2. 充分利用多台机器的ip加速爬取

   (① ip被禁止的可能性很小，② 将爬取分散到多台机器上去，加快了爬取的速度)



> scrapy本身是不支持分布式爬虫的，将scrapy变成一个分布式的爬虫

需求：

request队列集中管理 

去重集中管理

```python
这个集中管理不能放到scrapy中进行管理，因为scrapy并没有提供一种机制，可以让外部访问去重的set，所以上面的两个对列必须放到第三方的组件中去做，也就是redis
redis实际上就是内存数据库，也就是将所有的数据都放到内存当中，不像sql数据库是将数据放到文件中，做缓存的处理

```

![](http://m.qpic.cn/psb?/V12sfxtB3XMdNx/GqM7wzeCpOLxJVSPYRoOTDIoA6qZhltKDKl1Zs*Rdqg!/b/dPMAAAAAAAAA&bo=gASIAgAAAAADByw!&rf=viewer_4)

### Redis的基本概念

redis是一个key-value的存储系统，是一个内存数据库，会将所有的数据放到内存当中

类似于将python中的dict，从内存中单独成一个服务，可以让很多程序都去使用它，使用的时候就想python中操作dict一样很方便

redis也是支持数据持久化的，可以将数据保存到文件系统中

对于线程来说是线程安全的，也就是可以多个线程同时操作一个list，所以让我们做分布式非常的简单

>  redis的使用：

### redis的安装 

```python
linux下：
sudo apt-get install redis-server
redis是一个数据库，我们要对它进行操作就必须要安装一个客户端，需要安装一个redis_connect端，才能够进行操作

虚拟机安装redis连接
https://blog.csdn.net/zhangxing52077/article/details/72866226

开启命令：
(artical_spider) test@Ubuntu:/usr/local/redis-3.2.5/src$ ./redis-server 
(artical_spider) test@Ubuntu:/usr/local/redis-3.2.5/src$ ./redis-cli 
-----------------------------------------------------------------------------    
windows下：
 .................
```

##### redis数据类型

>字符串
>
>散列、哈希
>
>列表
>
>集合
>
>可排序集合

- 字符串的命令（如何设置和获取内存数据库里 面的字符串）
  - set   mykey   myvalue                  设置变量
  - get  mykey                                     获取变量
  - getrange   mykey   start  end     从字符串中获取子串
  - strlen  mykey                                查询长度
  - incr/decr  mykey           加1减1 （这里的mykey必须是int，或者是一个可以转化为int的字符串）
  - append   mykey    （str）   这样会给字符串进行一个join的操作


- 散列、哈希的命令
  - hset  myhash  mykey   myvalue                    设置变量
  - hget  myhash  mykey                                      获取变量
  - hexists myhash exkey                                     检查exkey、是否存在   
  - hdel  myhash  mykey                                      将hash里面的mykey删除
  - hval   myhash                                                   获取myhash里面所有的值


- 列表
  - lpush/rpush                                                      从左/右边传入，类似栈先进先出
  - blpop/brpop  key1[key2]  timeout                从左/右边删除一个元素timeout是  值如果没有元素的                                                                                                                            话会等设置的几秒钟、timeout的参数必须要传
  - lpop/rpop                                                          从左/右边删除一个元素，没有的话直接返回空
  - lrange  mylist  0 10                                           查看列表的开始到结束
  - llen  key                                                              查看长度
  - lindex key  index                                               取第几个元素的意思
- 集合 （不重复的）
  - sadd  myset  key                               设置一个值，如果再次set的相同的值的话，则会插不进去
  - scard  myset                                      获取set里面有多少元素
  - sdiff   key[key2]              让两个set做减法（前面的set减去和后面set的交集剩下的值，要注意顺序）
  - ![52715540779](C:\Users\dream\AppData\Local\Temp\1527155407791.png)
  - sinter  key[key2]                                          交集部分
  - spop   key                                                     从里面随机删除一个元素，并把这个元素返回回来
  - s
- 有序集合

























































