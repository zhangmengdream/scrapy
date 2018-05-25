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
- flushall                         将redis里面的所有数据清空

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

### elasticsearch搭建搜索引擎 

```python
elasticsearch 是基于java开发的
			 是一个RESTful
			 是一个分布式的系统
        	  是一个全文搜索引擎
下面关系型数据库的缺点部分，elasticsearch都可以完成
```



 **搜索我们可以用like之类的语句取数据库做查询，但是关系型数据库有以下缺点**

> 关系数据搜索缺点

1. 无法打分
   - 因为无法打分，所有无法对搜索出来的结果进行排序
2. 无分布式
   - 传统的数据库要做分布式是比较麻烦的，对程序员的要求比较高
3. 无法解析搜索请求
   - 比较复杂的情况（字数比较多的时候，无法解析）无法在数据库中搜索到我想要比配的关键词的时候，实际上是没有结果的 
4. 效率低
   - 当数据过多的时候，无法满足
5. 分词
   - 中文单个字很难有自己的意思，所有在分析请求的时候，需要分析
   - 比如：关系  数据  搜索  缺点    这样分词



##### nosql数据库(not only sql)

可以理解为文档数据库，与关系型数据库差别很大



![](http://m.qpic.cn/psb?/V12sfxtB3XMdNx/EmLDockelnzG.3hIj4r26oQEamO2qXk4L4OTtctYhcs!/b/dDEBAAAAAAAA&bo=QAMGAgAAAAARB3c!&rf=viewer_4)



对于关系型数据库，这样一段json，就需要四张表的关联来完成

nosql只需要把他们保存成一条记录，也可以简单的理解为，直接保存成json

不管这个json的结构如何，都是会保存到nosql数据库的

mongodb  redis  elasticsearch  都是属于nosql（nosql的类型非常多）数据库



elasticsearch 在某些情况下（比如update不频繁的时候）可以当作mongodb来使用

elasticsearch 在update情况下，比mongodb慢很多，mongodb的更新低于mysql

mongodb的插入和查询大多数情况下优于mysql

elasticsearch 主要用于搜索引擎 （既有数据的存储，又有数据的分析）

#### elasticsearch安装

elasticsearch是基于Java开发的，所以安装elasticsearch之前必须要安装java

 安装java：https://blog.csdn.net/ycbbeibei/article/details/78963950

java -version 查看java的版本



1. 安装elasticsearch-rtf（不建议安装官网上的，因为elasticsearch依赖的包比较多，建议GitHub上下载安装）

   在github上搜索elasticsearch-rtf

   启动elasticsearch-rtf  

   - unzip  解压
   - 在bin目录下运行./elasticsearch

2. head插件和kibana的安装

   ​	head插件相当于navcate，是基于浏览器的插件，可以完成类似navcate的功能

   ``` python
   在github上搜索elasticsearch-head
   git clone git://github.com/mobz/elasticsearch-head.git
   cd elasticsearch-head
   npm install
   npm run start
   # npm 是nodejs里面提出来的一种包管理器 类似于python里面的pip
   # 需要先将npm安装好

   npm有一个中央仓库，不过这个仓库实在国外的安装会非常慢
   所以有个cnpm 是淘宝npm的镜像
   http://npm.taobao.org/
   运行下面这条命令即可（可用cnpm）
   npm install -g cnpm --registry=https://registry.npm.taobao.org
       
   运行了npm install（或者cnpm install）就会生成node_modules(存放第三方包的)

   ```

```python
在elastiscsearch官网中下载kibana

下载完成之后，打开bin文件，里面会有两个文件
kibana           (这个是运行kibana的文件)
kibana-plugin    (这个是可以为kibana安装插件的)

kibana 运行这条命令，就会出现界面端口，从浏览器打开界面
```

#### elasticsearch概念

1. 集群：一个或者多个节点组织在一起
2. 节点：一个节点是集群当中的一个服务器，由一个名字来标识，默认是一个随机的漫画角色的名字
3. 分片：将索引划分为多份的能力，允许水平分割和扩展容量，多个分片响应请求，提高性能和吞吐量
4. 副本：数据的备份，创建分片的一份或多份的能力，在一个节点失败其余节点可以顶上

| elasticseacher | mysql  |
| :------------: | :----: |
|  index(索引)   | 数据库 |
|  type（类型）  |   表   |
| document(文档) |   行   |
|     field      |   列   |

```python
elasticseacher为了达到自己搜索的目的，数据是自己保存的，不是一个中间库，，
是一个集合了数据保存以及数据分析这样的一个搜索引擎，
elasticseacher也是要自己存储数据的
index(索引)做名词来使用的时候，代表某一个数据库
对某一个document做一个索引，这时候index(索引)就代表是一个动词
```

#### HTTP方法

|  方法  |                             描述                             |
| :----: | :----------------------------------------------------------: |
|  GET   |              请求指定的页面信息，并返回实体主体              |
|  POST  | 向指定资源提交数据进行处理请求，数据被包含在请求体中，post请求可能会导致新的资源的建立和、或已有资源的修改 |
|  PUT   |            向服务器传送的数据取代指定的文档的内容            |
| DELETE |                   请求服务器删除指定的页面                   |

```python
这里介绍最常用的四种方法，另外还有方法
HEAD,OPTIONS,TRACE,CONNECT

DELETE
请求服务器删除指定的页面（用post也可以完成，但是delete方法，含义更加明确）
POST不同的参数时，服务器会根据post过来的参数做相应的操作（包括删除）

```

#### 倒排索引

```python
概念：倒排索引源于实际应用中需要根据属性的值来查找记录，这种索引表中的每一项，都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性来确定记录的位置，因而成为倒排索引（inverted index）。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件（inverted file）
```

--------------------------------------------------------------------------------------------------------------

例：

假设有A、B、C三个文件，这三个文件可以当做elasticseacher的文档（这里只是举例子，实际文件的内容不止这么点）

文件A：通过python django搭建网站

文件B：python scrapy爬取网站数据

文件C：scrapy-redis 分布式爬虫

需求：查询有哪些文件包含python这个关键词 

--------------------------------------------------------------------------------------------------------------

解：
正常的思维方式，就是对文档进行遍历,需要遍历文件中的所有的内容，才能确定有没有这个关键词----这种效率是很低的，当文件上亿的时候，每一个关键词都要遍历上亿个文件
	
所以使用倒排，即.当，某个（比如A）文件存储之前，对A的内容全部遍历，遍历完之后进行分析 这里的分析也就是分词，分词都会进行存储，所以倒排索引的结构就是  |
															          V				

​				        	**python写各大聊天系统的屏蔽脏话功能原理**

| 关键词   | 文章         |
| :------- | :----------- |
| python   | 文章1，文章3 |
| 聊天     | 文章2        |
| 系统     | 文章3，文章4 |
| 屏蔽脏话 | 文章5        |
| 功能原理 | 文章6，文章7 |

左边是key，也就是关键词，右边是关键词出现的文章，关键词也就是分词之后出现的关键词

python在文章1中如果出现30次，在文章2中出现那1次，也要把出现的次数据记录下来，这样才能对文本进行打分，在某个文章中这个词出现的频率越高，这个词的权重也就越高

倒排索引还有更加精确的存储结构

| 关键词   | 倒排列表（这里是一个列表，这里的例子只写了一个元素，可以有多个元素） |
| -------- | ------------------------------------------------------------ |
| python   | （文章1，<2,10>,2）指在文章1的第2,10，位置出现过这个关键词，2代表频率，下面的一样 |
| 聊天     | （文章2，<12,25,100>,3）                                     |
| 系统     | （文章3，<10>,1）                                            |
| 屏蔽脏话 | （文章5，<50,60>,2）                                         |
| 功能原理 | （文章6，<56,57,58>,3）                                      |

有了倒排索引之后，再想查某个关键词在那些文章中出现过就显得很简单了

* 倒排索引和elasticseracher打分如何打的 -- 需要了解TF-IDF 概念
* TF-IDF 概念是elasticseracher进行打分的很关键的技术

--------------------------------------------------------------------------------------------------------------

目前的搜索引擎中，一般的底层的索引存储，都采用的是倒排索引

倒排索引也是搜索引擎区别于我们已有的关系型数据库或者其余的nosql数据库的核心

-----------------------------------------------------------------------------------------------------------------------------------

倒排索引待解决问题（这些问题elasticseacher都已经帮我们完成了）

1. 大小写转换问题，比如python和PYTHON是否应该作为同一个词
2. 词干抽取，looking和look是否可以通过词干抽取为同一个词
3. 分词，如 屏蔽系统  是应该分为 ‘屏蔽’、‘系统’ 还是 ‘屏蔽系统’
4. 倒排索引如果文件过大，如何进行压缩编码

#### elasticseacher索引的操作





















































































































































