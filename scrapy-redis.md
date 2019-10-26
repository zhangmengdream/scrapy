有中文注释。不能使用的时候在上面加

coding=utf-8  就可以了

ensure_ascii=False  这里的意思是数据里如果有中文的话，会默认使用ascii编码，这里设置不启用ascii编码



安装redis

```python
链接 ： https://redis.io/download
里面有步骤

wget http://
 

如果要支持分布式的话，redis.conf 需要修改配置

61行  
bind 127.0.0.1  
# 这个的意思是只支持本地读取
# 注释掉，说明不在绑定本地的了，任何的数据库都可以访问，这样我的远程端才可以连接到master端的redis数据库

129行
daemonize no  

# redis是否作为守护进程来使用
# 这里改为yes就不需要开两个窗口启动redis了 
# 守护的意思就是在后台执行不在前端显示
```











### scrapy redis的分布式爬虫以及源码解析

```py
redis_scrapy并不是框架而是基于scrapy框架之上的组件

用来替换scrapy原来的一些东西，让scrapy拥有了只支持分布式的功能

scrapy本身是有去重的，是在内存中执行的，请求量非常大的时候scrapy的占用量会非常高

将去重的指纹队列放到数据库里，如果我想临时中断他，修改一些东西，再次读取的时候，会读取redis数据库里的请求指纹，之前爬过的请求，就不会再去做了

itempipeline  管道文件，redis也提供了一个模板（在redis数据库里统一存储）

redis是基于内存的，一旦关机，内存的数据会被清空

redis数据库会有三个库
存储数据的
存储请求的队列
存储请求的指纹 --  如果没有过，留下指纹，进入队列，如果有过就不能进入队列

所有的爬虫端共享一个redis数据库

数据用itemprocess这个单独的文件拿出来 

如果不拿出来放在redis数据库里也可以但是这样不安全，数据可能会丢失
```



scrapy-redis策略

```python
假设有四台电脑（python是跨平台的，在什么操作系统中都是一样的）
w10  mac  ubuntu  centos 任何一台电脑都可以作为Master或者Slaver端

Master端 只能有一个（核心服务器）不负责做爬取，但是也可以做爬取，但是爬取的数据就存储在了自己本身的数据库中了，他本身也可以做爬虫端，不过一般在项目里面服务器端是不会做爬取的，因为数据集群一旦做起来的话，网络流量或者磁盘的耗用量是非常高的，所以我们需要尽可能的减少服务器的复杂，所以服务器不做爬取，只做服务器就可以了
做 指纹

Slaver端 （爬虫程序执行端）可以有多个  负责执行爬虫程序，运行过程中提交新的request给Master

Master端负责request的去重和任务分配，他拿到request之后，把请求发给slaver端，slaver拿到新的请求之后交给master端，master端进行指纹去重，去重成功进入队列，然后再发给slaver端进行爬取（发给slaver按照优先级来），有数据的话也会发给master端做数据存储

master有一个redis数据库，负责将未处理的request去重和任务分配，将处理后的request加入待爬取的队列，并且存储爬取的数据
```

![52820126060](C:\Users\dream\AppData\Local\Temp\1528201260600.png)

```python

scrapy_redis默认使用的 任务调度，去重、scrapy、存储，redis都已经帮我们做好了
我们只需要继承法redisSpider 或者rediscrawlspider，指定一个redis_key就可以了

redis_key:
爬虫写好之后，部署好之后，各个爬虫端把程序执行起来，他们不会立即爬数据，他们会等待master端给他们分配任务，这个任务第一个起点就是redis_key指定的。
比如我们有四台机器，这四台机器把爬虫端启动起来了，一开始队列里没有请求，所有爬虫都处于等待状态，一旦master端往redis数据库里push一个请求过去们所有的爬虫就起动了 

缺点是：scrapy_redis调度的任务是request对象，里面信息量比较大（不仅包含url，还有callback函数，headers等信息）可能导致的结果就是会降低爬虫速度，而且会占用redis大量的内存空间，所以如果要保证效率，就需要一定的硬件水平

分布式首先要保证机器成本和人工成本是否是满足的，同一个局域网里搭分布式也是不明智的，因为带宽是统一的（同一个），最好的方法在不同的网段里面搭配各种不一样的请求，这样才会把分布式的性能提升到极致，也就是需要一定的硬件水平

lpush  myspider:start_urls http：//www.domz.org/

lpush 代表启动指令，网数据库里面push一条信息，这个信息代表我的这个指令
myspider:start_urls 是在spider里面配置的redis_key的值
http：//www.domz.org/  这个就是代表start_urls

一旦在master发送这条指令，其他的slaver端都会执行（只要连接的是同一个数据库）



scrapy_redis写的爬虫的执行代码
scrapy  runspider  myspider_redis.py

与单机的区别就是多了一个
redis_key

少了一个start_url

redis分布式的start_url在redis_key 上面push上去

动态的获取域的范围:
def __init__(self,*args,**kwargs):
    domain = kwargs.pop('domain','')
	self.allowed_domains=filter(None,domain.split(','))
    super(Myspider,self).__init__(*args,**kwargs)
    

请求统一处理，数据统一存储,所有的都是统一管理

服务器端只是负责接收数据（甚至可以不需要scrapy，不需要python）
服务器端把redis装好
redis.conf 
设置bind127.0.0.1注释掉

（保证master端和slave端都能够互相ping通）
把数据库服务启动起来
把主机ip和端口号告诉爬虫端的机器
爬虫端的机器在setting里面设置redis的主机ip和端口号
这样他们的数据就指了主机了
然后挨个启动slave端的爬虫机器

然后再redis数据库是输入redis_key(指明第一个要发送的请求)  这样就可以跑起来了
lpush  redis_key http://sss

检查redis数据库服务启动了没有，自己在redis数据库中测试一下，看看能否进入redis-cli能够进入客户端就代表可以启动
```

















### redis 

```python
远程连接一个数据库、
启动：sudo  redis-cli -h 192.168.21.64

这样两个就可以连接远程的redis了

这是各个电脑之间数据库的联通

真正写爬虫的时候，爬虫端是不需要启动redis-server的，master端启动就可以了，只需要爬虫端能够读取redis数据库，也就是能ping通服务端，（爬取的时候我们的组件会帮我们联通redis数据库） 只有slave端能够读取到master端的redis数据库，就表示能够连接成功，可以实施分布式



```



```python
1.在setting中设置
# 使用scrapy_redis里面的去重组件，不使用scrapuy默认的去重
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 使用了scrapy_redis的调度器组件
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 使用了scrapy_redis的管道组件，支持将数据存储到爆redis数据库里，必须启动
ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 300
}
# 允许暂停，redis请求记录不丢失
SCHEDULER_PERSIST=True
如果设置为false，暂定爬取之后就会从头开始爬了

# scrapy_redis 默认的请求队列形式（按照优先级顺序）
SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.SpiderPriorityQueue'

# 这里还可以自己加一些其他的参数
# 指定数据库的主机ip 
REDIS_HOST = '192.168.0.0'
# 指定数据库的端口号
REDIS_PORT = 6379
# 这些如果不写，数据库就默认是本地，127.0.0.1,写的话就支持存储到其他的数据库里

DOWNLOAD_DELAY=1
# 下载延迟
```





开始爬取的时候，一切准备就绪后在redis这端发起一个指令

eg：

lpush  myspider:start_url   http://www,dmoz.org/

(一般不需要在这里push，在spider里面会有redis_key直接爬取)

redis_key = 'myspider:start_urls'

这个格式不是必须的，但是一般都使用这个格式



相当于slaver端开启爬虫没有请求，等待状态，等待master端发一个指令过去，让他执行



在redis中如果爬取了一半想要增加功能然后继续爬取的话，爬取过的网站就不会再继续爬取了，因为指纹已经记录了这个网站（这就是redis分布式的特性）





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
- flushall                         将redis里面的所有数据清空

### scrapy   redis  搭建分布式爬虫 的使用

python redis的使用

```python
1.在setting中设置
# 使用scrapy_redis里面的去重组件，不使用scrapuy默认的去重
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
# 使用了scrapy_redis的调度器组件
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
# 使用了scrapy_redis的管道组件，支持将数据存储到爆redis数据库里，必须启动
ITEM_PIPELINES = {
    'scrapy_redis.pipelines.RedisPipeline': 300
}
# 允许暂停，redis请求记录不丢失
SCHEDULER_PERSIST=True
# scrapy_redis 默认的请求队列形式（按照优先级顺序）
SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.SpiderPriorityQueue'

# 这里还可以自己加一些其他的参数
# 指定数据库的主机ip 
REDIS_HOST = '192.168.0.0'
# 指定数据库的端口号
REDIS_PORT = 6379
# 这些如果不写，数据库就默认是本地，127.0.0.1,写的话就支持存储到其他的数据库里









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





def request_seen(self,request):
   fp = self.request_fingerprint(request)
        # This returns the number of values added, zero if already exists.
        added = self.server.sadd(self.key, fp)
        #sadd 是set的add  key是名称   fp就是request的指纹，加到key里面，如果成功就会返回1，如果失败就会返回0  
        return added == 0
		#  返回0 的话就代表入库失败，入库失败了就代表已经存在了，
         # 也就是request_seen等于true了


 
```

### picklecompat.py (兼融py2，py3)

```python
掉用了pickle

```

### pipelines.py  (将item放到redis中)

```python
需要在setting中保存一个pipeline
# 这个过程是没有必要将这个过程放到redis中，可以在本地downloader之后，可以通过本地的pipeline将他保存下来，这样既下载了这个url，又将这个url的数据解析完成之后，保存到我们的数据库中，所以这里的pipeline可以不设置，不设置的话在本地做保存即可，设置的话可以将item放到我们的redis中，可以完成共享 

（好处）放到redis中，我就可以多启几个进程，甚至可以只写一段代码，不需要爬虫，写一段从redis里面取数据的python文件，然后不停的入库即可



    @classmethod
    def from_settings(cls, settings):
        params = {
            #首先会调用这个函数from_settings---也就是get_redis_from_settings这个函数
            #这个函数会初始化 redis-server
            'server': connection.from_settings(settings),
        }
        if settings.get('REDIS_ITEMS_KEY'):
            params['key'] = settings['REDIS_ITEMS_KEY']
        if settings.get('REDIS_ITEMS_SERIALIZER'):
            params['serialize_func'] = load_object(
                settings['REDIS_ITEMS_SERIALIZER']
            )

        return cls(**params)

    @classmethod
    def from_crawler(cls, crawler):
        return cls.from_settings(crawler.settings)

    def process_item(self, item, spider):
        #deferToThread 异步的对象  这里的意思是将process_item这个函数传到_process_item里面去做，而且_process_item是异步化的，放到线程中去做
        return deferToThread(self._process_item, item, spider)
#这里没有直接把_process_item的逻辑拷贝到process_item里面，是为了效率更高，放到线程中去做，后面源源不断来的item就不会受影响
    
    def _process_item(self, item, spider):
        key = self.item_key(item, spider)
        data = self.serialize(item)
        self.server.rpush(key, data)
        return item
    #        self.server.rpush(key, data) 因为item需要顺序的处理，所以这里调用了对列rpush是放到对列的对尾

```

### queue.py （供（scheduler）request的调度来使用的）

```python
# 有三种queue   FifoQueue    PriorityQueue     LifoQueue
class FifoQueue(Base):	
#FifoQueue 是first in  first out  指先进先出的队列（有序队列）后进来的放在队尾

#代码中，进来的时候 lpush 放在对头
	   #取数据的时候 rpop  从队尾取
	def push(self,request):
        # push的时候，将request放到server里面
        #_encode_request 是指将request做encode
        self.server.lpush(self.key, self._encode_request(request))
        
       """
        def _encode_request(self, request):
        '''Encode a request object'''
        	obj = request_to_dict(request, self.spider)
        	return self.serializer.dumps(obj)
        	
       _encode_request 调用的是 serializer  调用的是picklecompat
       """
        
        
        
class PriorityQueue(Base):	
# 调用的是zcard  有序集合
    def push(self, request):
        data = self._encode_request(request)
        score = -request.priority    # 这里是设置的优先级的 
        # priority  生成一个request的时候，可以赋值的，这个值越大越优先爬取
        self.server.execute_command('ZADD', self.key, score, data)
'''
push 的时候调用的是server.execute_command('ZADD',self.key,score,data )
score 是分数
'''    
 	def pop(self, timeout=0):
        """
		zrange(self.key, 0, 0) 这里就是取第一个数的意思
        """
        pipe = self.server.pipeline()
        pipe.multi()
        pipe.zrange(self.key, 0, 0).zremrangebyrank(self.key, 0, 0)
        results, count = pipe.execute()
        if results:
            return self._decode_request(results[0])
        
    
class LifoQueue(Base):	
# 后进先出

```

### scheduler.py

```python
在spider里面yield一个request之后，就会调用enqueue_request这个函数

### scrapy的 scheduler.py 的源码
def enqueue_request(self, request):
        if not request.dont_filter and self.df.request_seen(request):
            self.df.log(request, self.spider)
            return False
        dqok = self._dqpush(request)
 '''
 _dqpush :  拿到request之后会push request
 
 '''       
        
        if dqok:
            self.stats.inc_value('scheduler/enqueued/disk', spider=self.spider)
        else:
            self._mqpush(request)
            self.stats.inc_value('scheduler/enqueued/memory', spider=self.spider)
        self.stats.inc_value('scheduler/enqueued', spider=self.spider)
        return True

def next_request(self):
        request = self.mqs.pop()
        if request:
            self.stats.inc_value('scheduler/dequeued/memory', spider=self.spider)
        else:
            request = self._dqpop()
            if request:
                self.stats.inc_value('scheduler/dequeued/disk', spider=self.spider)
        if request:
            self.stats.inc_value('scheduler/dequeued', spider=self.spider)
        return request



### scrapy-redis 的 scheduler.py 的源码

# from_settings 做为一个入口
def from_settings


```

### spiders.py

```python
class RedisMixin(object):
    redis_key = None
    redis_batch_size = None
    redis_encoding = None
    server = None

    def start_requests(self):
        """这里重载了start_requests这个方法  就可以调用自己的next_requests """
        return self.next_requests()

    def setup_redis(self, crawler=None):
        """Setup redis connection and idle signal.

        This should be called after the spider has set its crawler object.
        """
        if self.server is not None:
            return

        if crawler is None:
            # We allow optional crawler argument to keep backwards
            # compatibility.
            # XXX: Raise a deprecation warning.
            crawler = getattr(self, 'crawler', None)

        if crawler is None:
            raise ValueError("crawler is required")

        settings = crawler.settings

        if self.redis_key is None:
            self.redis_key = settings.get(
                'REDIS_START_URLS_KEY', defaults.START_URLS_KEY,
            )

        self.redis_key = self.redis_key % {'name': self.name}

        if not self.redis_key.strip():
            raise ValueError("redis_key must not be empty")

        if self.redis_batch_size is None:
            # TODO: Deprecate this setting (REDIS_START_URLS_BATCH_SIZE).
            self.redis_batch_size = settings.getint(
                'REDIS_START_URLS_BATCH_SIZE',
                settings.getint('CONCURRENT_REQUESTS'),
            )

        try:
            self.redis_batch_size = int(self.redis_batch_size)
        except (TypeError, ValueError):
            raise ValueError("redis_batch_size must be an integer")

        if self.redis_encoding is None:
            self.redis_encoding = settings.get('REDIS_ENCODING', defaults.REDIS_ENCODING)

        self.logger.info("Reading start URLs from redis key '%(redis_key)s' "
                         "(batch size: %(redis_batch_size)s, encoding: %(redis_encoding)s",
                         self.__dict__)

        self.server = connection.from_settings(crawler.settings)
        # The idle signal is called when the spider has no requests left,
        # that's when we will schedule new requests from redis queue
        crawler.signals.connect(self.spider_idle, signal=signals.spider_idle)

    def next_requests(self):
        """允许我们获取首个 start_url的时候 是通过redis获取的 ，而不是schedule（最初的时			候schedule是没有数据的 ）
        """
        use_set = self.settings.getbool('REDIS_START_URLS_AS_SET', defaults.START_URLS_AS_SET)
        fetch_one = self.server.spop if use_set else self.server.lpop
        # XXX: Do we need to use a timeout here?
        found = 0
        # TODO: Use redis pipeline execution.
        while found < self.redis_batch_size:
            data = fetch_one(self.redis_key)
            if not data:
                # Queue empty.
                break
            req = self.make_request_from_data(data)
            if req:
                yield req
                found += 1
            else:
                self.logger.debug("Request not made from data: %r", data)

        if found:
            self.logger.debug("Read %s requests from '%s'", found, self.redis_key)

    def make_request_from_data(self, data):
        """

        """
        url = bytes_to_str(data, self.redis_encoding)
        return self.make_requests_from_url(url)

    def schedule_next_requests(self):
        """Schedules a request if available"""
        # TODO: While there is capacity, schedule a batch of redis requests.
        for req in self.next_requests():
            self.crawler.engine.crawl(req, spider=self)

    def spider_idle(self):
        """Schedules a request if available, otherwise waits."""
        # XXX: Handle a sentinel to close the spider.
        self.schedule_next_requests()
        raise DontCloseSpider

        
class RedisCrawlSpider(RedisMixin, CrawlSpider):
"""
基于CrawlSpider的spider
"""
```

### utils.py

```python
为了兼容py2 和 py3 写得
```



### 如何将 bloomfilter  集成到scrapy-redis中



bloomfilter  是最省内存的一种去重策略，他的对bitmap升级的一种方式，因为bitmap当url过多的时候，冲突性很高，很多url可能会映射到同一个地址，。

bloomfilter  是使用了多个hash函数，来对url进行多次的bitmap的映射，降低冲突的概率

bloomfilter的原理，和整体思想，去网上自行百度

7575651

```python

10101001   
表示电压的高和低，电压  高 1  低 0 所以计算机归根结底是由电压来表示的

1byte = 8 bit
'a' = 1byte
'a' = utf8 1byte
'字' = utf8 3byte
set = ('abc')

1kb = 1024 byte
1M = 1024 kb
1G = 1024 M


bloomfilter的算法 是创建一个M位的 BitSet

这里就是bloomfilter的源码，可以直接粘贴使用
https://github.com/liyaopinner/BloomFilter_imooc/blob/master/py_bloomfilter.py

    
#  bloomfilter如何集成到去重器中
dupefilter.py 是去重器 -- 改造他，将bloomfilter集成进来


改造dupefilter.py:
from ScrapyRedisTest.utils.bloomfilter import conn,PyBloomfilter
    
    def __init__(self, server, key, debug=False):
        """bloomfilter的用法是 bf.is_exist()
		所以我们必须要生成一个bf的对象
		但是不可能每次调用request_seen 的时候，都生成bf对象，这样效率会太低
		所以需要在__init__里面生成一个全局的 bf 对象
        """
        self.server = server
        self.key = key
        self.debug = debug
        self.logdupes = True
        #在这里实例化
        self.bf = PyBloomfilter(conn=conn,key=key)

        
        
    def request_seen(self, request):
        """
        判断fp如果存在返回true，如果不存在，把fp放进bf，返回false
        """
        fp = self.request_fingerprint(request)
        if self.bf.is_exist(fp):
            return True
        else:
            self.bf.add(fp)
            return False
        # 下面两段代码就不需要了，以访问的url的集合就由bf来管理了
        #added = self.server.sadd(self.key, fp)
        #return added == 0		
        

push一个初始的start-url进去
lpush jobbole:start_urls http://blog.jobbole.com/all-posts/
命令检测是否改造成功 

```

### 









pipeline里面，如果写成json形式的

```python
import json
class Youyuanpipeline(object):
    def __init__(self):
	    self.filename = open('youyuan,json','w')
    def process_item(self,item,spider):
        content = json.dumps(dict(item),ensure_ascii=False)+',\n'
        self.filename.write(content.encoding('utf-8'))
        return item
    def close_spider(self,spider):
        self.filename.close()
```

json格式不对的情况，可以先挨个写到列表里面，然后再把列表写入json里面

```python
class
```



上传代码给远程服务器做提交

```python
sftp

如果我在ubuntu下面的写的代码，想发送到mac里面
需要先打包
tar -cvf youyuan.tar youyuan
c   打包   
v   显示压缩的过程   
z   压缩
f   待压缩的
x   解包

youyuan.tar   打包过后的包名
youyuan       打包这个文件

tar -cxvf youyuan.tar.gz youyuan
打包和压缩

tar包没有压缩的功能，只有打包的功能

sftp  root@193
sftp  用户名@ip地址
加密码：

youyuan.tar
 
sftp> put  gas.tar
sftp> ls   (远程的意思)
sftp> lls  (本地的意思)

在另一台机器上
tar xvf  gas.tar   解压




下载：scp -r 远程的用户名@远程ip:/文件地址 本地地址
上传：scp -r 本地地址 远程的用户名@远程ip:/文件地址

```



```python
分布式的执行命令：
在 spider下面执行 scrapy runspider yy.py

lpush redis_key  news.163.coms
 

```



```python
class MyMongodb:
    def __init__(self):
        self.conn = None

    def open_spider(self, spider):
        self.conn = client = MongoClient('192.168.137.254', 27017)
        # 初始化油表对象
        self.db = client.zhiliandb

    def process_item(self, item, spider):
        self.db.zhiliansofttong.insert({"salary": item["salary"],
                                'experience': item["experience"],
                                'company': item["company"],
                                "jobvacancy": item["jobvacancy"],
                                "education": item["education"],
                                "describes": item["describes"],
                                "jobtype": item["jobtype"],
                                "address": item["address"],
                                "worktype":item["worktype"],
                                "peoplenum": item["peoplenum"]
                                })

        return item

    def close_spider(self, spider):
        # 关闭mysql连接和游标
        self.conn.close()
```





```python
url = bus_item.get_xpath('//img[@class='']/@src')   get_xpath的意思就是直接取值赋值给url   												只有一个参数
bus_item.add_value('url',response.url)     直接给一个值
```

```python
数据获取的时间，爬虫的名字   经过这个管道就会给他加上这两个字段

class ExamplePipeline(object):
    def process_item(self,item,spider):
        item['crawled'] = datetime.utcnow()
        item['spider'] = spider.name
        return item


```

文件里有中文注释的时候 需要加上

```pu
# -*- coding: utf-8 -*-
```

执行分布式命令

```python
scrapy runspider myspider.py
```



在spider下面执行

rule规则，也可以写css规则

```pyt
rules=[
    Rule(LinkExtractor(
    	restrict_css=('.top','.sub','.cat')),callback='sss'.follow=True
    ),
]
```





.com是商业网站

.org是非盈利机构

lpush  qicha:spider   https://www.qichacha.com/





爬虫端不需要启动redis_server,

dupefilter  请求指纹，也就是哈希值

requests    是所有的请求，是由一堆16进制显示的，可以使用表结构键值 HEX  TABLE





**sudo apt-get install virtualenv**

**sudo apt-get install virtualenvwrapper**

**source /usr/share/virtualenvwrapper/virtualenvwrapper.sh**     注意路径

新开一个终端，先mkvirtualenv「项目名字」

配置workon 环境变量

vi ~/.bashrc

1.接着，我们需要配置下 ~/.bashrc，将 virtualenv 添加进去：

​    即将:

​    export WORKON_HOME=$HOME/.virtualenvs

​    source /usr/local/bin/virtualenvwrapper.sh

​    复制到 ~/.bashrc中,保存退出

2.让 bashrc 生效：

执行source ~/.bashrc命令

****

root@iZbhb13x4ji74cZ:/usr/local/bin# workon env1



/bin/bash





sudo sh rsyncclient.sh

rsyncclient.sh ---------

source ${ABSPATH}/client.conf

\-------------------

报错: source: not found

原因：sh和bash是不同的shell,sh中没有source命令。

解决办法：sudo bash  rsyncclient.sh



cpy2 = soup.find('div',class='aaa')   这个代表找到class为aaa的div  返回的是一个列表

```python
company_full_name = cpy2_content[1].get_text().strip()[len("公司全称:"):] if  cpy2_content[1] else ''

意思是如果cpy2_content[1] 有值就走前面，没有值就为空
```





















