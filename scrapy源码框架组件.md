engine

```python
def schedule(): 这个方法调用了schedule的equeue_request(),表示直接将requests放到schedule里面
    
    所以engin拿到request之后，先放到schedule，然后从schedule里面取出request，然后再发送给downloader。

```

HttpRequest

```python
scrapy里面有一个默认的middleware
downloadermiddleware里面的cookies，  里面有一个cookiejar
从request.meta.get('cookiejar')
从request.meta中取出来的
在这里会把cookie放到header中

```

HttpResponse

```python
源码 http ---> response --->  html.py  这里面就是htmlresponse
可以看出htmlresponse继承了textresposne

里面有xpath、css的selector 
```

随机更换useragent

```python
方法1：

在settings中设置:
user_agent_list=[
	"",
	""
]

在 spider页面import进来：
from settings import user_agent_list

import random
random_index = random.randint(0,len(user_agent_list)-1)
random_agent = user_agent_list[random_index]

headers = {
    "HOST":"www.zhihu.com",
    "Referer":"https://www.zhihu.com",
    'User-Agent':random_agent
}

但是这样方法比较麻烦，因为headers不是每次都在循环里面的，需要写在yield上面，才可以实现随机取出一个useragent，比如下面：(这种做法比较简单，但是每次request的时候，都得重复这样的操作，代码比较冗余)

import random
random_index = random.randint(0,len(user_agent_list)-1)
random_agent = user_agent_list[random_index]
self.headers["User-Agent"]=random_agent
yield scrapy.Request(request_url,headers=self.headers,callback...)
--------------------------------------------------------------------------
--------------------------------------------------------------------------

在downloadermiddleware里面处理



    @classmethod
    def from_crawler(cls, crawler):
        o = cls(crawler.settings['USER_AGENT'])
        crawler.signals.connect(o.spider_opened, signal=signals.spider_opened)
        return o
    
这是一个静态的方法，可以通过这个类直接调用这个方法，所有的类只要重载了from_crawler这个函数，都会把当前的crawler给我们传递进来
o = cls(crawler.settings['USER_AGENT']) 这里表示会去setting里面去取useragent，如果取不到，就会使用默认的scrapy。（scrapy很容易会被识别出来）
所以在setting需要写一个，这样写的话就会替换掉scrapy
------------------------------------
USER_AGENT
------------------------------------

   def process_request(self, request, spider):
        if self.user_agent:
            request.headers.setdefault(b'User-Agent', self.user_agent)

这里表示如果要处理，对request进行处理的话，就必须要这样写



如果写了自己的useragentmiddleware，就必须把原始的置为none，或者把我们自己写的优先级数据设置很大，这样的话原始的useragentmiddleware就会先处理，我们的就会修改掉原来的。
--------------------------------------------------------------------------
--------------------------------------------------------------------------
方法2：

import random
class RandomUserAgent(object):

    #这里接收crawler传递进来的值
    # 这里调用super方法，用父类的init方法来初始化他
    def __init__(self,crawler):
        super(RandomUserAgent,self).__init__()
        # 这样的话就可以拿到setting中的 user_agent_list 
        self.user_agent_list = crawler.settings.get("user_agent_list",[])
    @classmethod
    def from_crawler(cls,crawler):
        return cls(crawler)
    def random():
        random_index = random.randint(0,len(user_agent_list)-1)
        random_agent = user_agent_list[random_index]
        return random_agent

    # 这里的crawler传递进来的东西就比较多了，不止是setting，还有crawler的其他信息
    def process_request(self,request,spider):

        request.headers.setdefaule('User-Agent',random())
-----------------------------------------------------------------------------------
补充
    直接通过(这样就可以直接取出setting中的值了) --- 静态的方法
    @classmethod
    def from_settings(cls,settings):
        dbparms = dict(
        useragent = setting['useragent']
        )
        pass
-----------------------------------------------------------------------------------
方法3：
from fake_useragent import UserAgent
class RandomUserAgent(object):

    #这里接收crawler传递进来的值
    # 这里调用super方法，用父类的init方法来初始化他
    def __init__(self,crawler): 
        super(RandomUserAgent,self).__init__()
        self.ua = UserAgent()
        # 这里获取setting中设置的random
        self.ua_type = crawler.settings.get("RANDOM_UA_TYPE","random")
        
    @classmethod
    def from_crawler(cls,crawler):
        return cls(crawler)

    # 这里的crawler传递进来的东西就比较多了，不止是setting，还有crawler的其他信息
    def process_request(self,request,spider):        
        # 这里是为了变成可配置的 ，所以在函数里面调用函数，变成可配置的
        def get_ua():
	        return getattr(self.ua,self.ua_type)
        request.headers.setdefaule('User-Agent',get_ua())
        
为了使代码更加强壮，在setting中设置随机random,或者某个指定浏览器（ie，chrome之类的），变成可配置的
RANDOM_UA_TYPE = 'random'


```















































