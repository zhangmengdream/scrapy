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
  - ![52715540779](http://m.qpic.cn/psb?/V12sfxtB3XMdNx/8yhf3AmcoQYQ8jq*dwM0k4zDah.8vbuogWFNdReC83U!/b/dDABAAAAAAAA&bo=fAJxAQAAAAADByw!&rf=viewer_4)
  - sinter  key[key2]                                          交集部分
  - spop   key                                                     从里面随机删除一个元素，并把这个元素返回回来
  - srandmember   key  member                   从key中随机获取member个元素出来
  - smembers key                                             获取key里面的所有的元素
- 有序集合
  - zadd  zkeys  0  'jango'   5  'scrapy'                          每个元素在设置的时候都有一个分数
  - zrangebyscore  zcourses_set    num1  num2      获取num1、num2两种分数之间的元素
  - zcount  zkey  num1  num2                                     两个分数之间的元素统计有多少个
- type    key                   可以查看某个字段是什么类型
- keys *                           查看所有

### scrapy   redis  搭建分布式爬虫 的使用

python redis的使用

```python
1.在setting中设置
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 300
}
scrapy为我们提供一个基于redis pipeline，将item序列化发送到redis中，这样所有解析出来的item都会放置在redis中，这样当我们启动爬虫的时候，就可以将item的保存做成分布式的

再写spider的时候，不能再继承我们自己的spider了，而是要继承redis的spider

启动一个spider之后，必须先往队列里面放置一个初始化的start_url 
现在所有的url都是放置在redis中了，所以初始的时候，需要往里面push一起始的url，这样才能够开始爬取

redis-cli  lpush  myspider:start_urls http://google.com
```

```python
新建一个项目:
scrapy startproject scrapyRedisTest

将scrapy redis 的源码从github克隆下来 拷贝进项目中

parse里面就可以和之前一样使用，因为RedisSpider 也是继承spider的
```



### 剖析scrapy_redis的源码

```markdown
#所有函数大致浏览

connection.py    用来连接redis的connection文件（用的非常多也是最重要的文件）

defaults.py      default的设置的文件

dupefilter.py	 用来过滤的文件，用来替换scrapy默认的去重器

picklecompat.py  用来做序列化的（比如request、pipeline等）

pipelines.py     将我们的一些item保存到redis中，实现了item的分布式的保存

queue.py         实现了三种队列，这三种队列是做request的队列的，
可以让我们保存数据的时候采用三种模式，（先进先出，先进后出，优先级三种队列   对应这scrapy的三种队列（scrapy也是有三种队列的）） 这里是使用redis的队列来完成的

scheduler.py     url的调度器，也是自己通过redis来实现的

spider.py        读取start_url的时候,使用过redis来读取的（这里重载了scrapy的spider）

utils.py         做了一个py3的兼容
```

### 					  scrapy_redis架构图											



![https://www.biaodianfu.com/wp-content/uploads/2016/12/scrapy-redis.jpg](https://www.biaodianfu.com/wp-content/uploads/2016/12/scrapy-redis.jpg)



```python
schedule的原理是将request放入队列中的
在使用redis分布式的时候就不能放入队列中了，因为队列是存放在内存的

这里是首先将request放入redis中，读取next-request的时候，是从redis的set中或者队列中读取出来的
item进入到itempipeline之后，pipeline会将item序列化放到redis中。item再取出数据通过item processes（保存数据库的过程可以写在这里）

下载那块没有变

去重器：

分布式将我们绝大部分的中间状态都保存在了redis中了

好处：爬虫的启动与暂停就不需要scrapy的启动与暂停了，因为这里所有的重要的状态，都已经放入到了redis中了
```



### connection.py (创建了redis的连接)

```python

这个文件提供了get_redis_from_settings(settings)

里面引入default
from . import defaults 




```

 

### defaults.py （里面有很多默认的配置）

```python
DUPEFILTER_KEY = 'dupefilter:%(timestamp)s'    保留每一个访问过的request的指纹

PIPELINE_KEY = '%(spider)s:items'    每一个spider的item的队列

REDIS_CLS = redis.StrictRedis        使用的是redis这个模块

REDIS_ENCODING = 'utf-8'             链接redis需要指明的编码   

设置一些redis的参数，供redis-py来用的
REDIS_PARAMS = {
    'socket_timeout': 30,
    'socket_connect_timeout': 30,
    'retry_on_timeout': True,
    'encoding': REDIS_ENCODING,
}

SCHEDULER_QUEUE_KEY = '%(spider)s:requests'
request的队列需要的key，也就是这个队列的变量名

SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'
使用 哪一种类型的队列（有三种类型的队列）

SCHEDULER_DUPEFILTER_KEY = '%(spider)s:dupefilter'
保存去重的key（可以理解为变量名）

SCHEDULER_DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'


START_URLS_KEY = '%(name)s:start_urls'
START_URLS_AS_SET = False

```

### dupefilter.py (去重的文件)

```python











```

### picklecompat.py (兼融py2，py3)

```python


```

### pipelines.py  (将item放到redis中)

```python
需要在setting中保存一个pipeline


```

### queue.py （供（scheduler）request的调度来使用的）

```python
class FifoQueue(Base):	
#FifoQueue 是first in  first out  指先进先出的队列（有序队列）放在队尾


    
```



```

### scheduler.py

```python




```







































