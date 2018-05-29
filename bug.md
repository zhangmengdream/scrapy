##  request和response

```markdown
# request：

cookie是可以自动设置的，不过自动登录之后，scrapy就会自动将cookie加入到request当中.

scrapy会有一个默认的middleware
# 参数：
1. cookie：  可以使一个dict，也可以是一个list
2. priority: 这个参数是用来影响scheduler的优先调度（比如一个request后来，下一次调度的时候，如果设置的priority比较高，会优先调度这个request）
3. dont_filter: 表名这个request不应该被过滤（当设置为True的时候，这个request是会被过滤掉的） --  当你在同一时刻，发送多个request的时候，希望不要被过滤掉，就可以设置他为false
4. errback 如果在处理请求时出现异常，将调用该函数,做一下后续的处理。这包括了404错误的页面。



```

```markdown
# response
返回的参数
url
status
headers
body
flags
request --- 之前yield出去的request  可以知道当时是对哪个request做的解释

```



## 如何随机的切换user-Agent

```markdown
user-Agent 用户代理
可以让服务器能够识别用户使用的操作系统，以及版本，CPU类型，浏览器版本，浏览器的渲染引擎，浏览器语言等等

# 如何随机的切换user-Agent



```



## DOWNLOADER_MIDDLEWARES

```markdown
和pipeline一样，需要自己配置class  后面的数字代表执行的顺序





```





### fake-useragent

基于github 开源的专门用来随机切换user-agent的

第一步 安装：

pip install fake-useragent

用法：

```python
>>> from fake_useragent import UserAgent
>>> a = UserAgent()

# 他会随机切换不同的版本
>>> a.ie
'Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; Trident/5.0)'
>>> a.firefox
'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:21.0) Gecko/20130401 Firefox/21.0'


# 直接random，会在不同的浏览器之间随机的切换，切换系统
>>> a.random
'Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; InfoPath.1; SV1; .NET CLR 3.8.36217; WOW64; en-US)'

```

#### 代码块：

```python
from fake_useragent import UserAgent
class RandomUserAgentMiddlware(object):
    # 随机更换user-agent

    def __init__(self,crawler):
        super(RandomUserAgentMiddlware,self).__init__()
        self.ua = UserAgent()
    @classmethod
    def from_crawler(cls,crawler):
        return cls(crawler)

    def process_request(self,request,spider):
        request.headers.setdefault('User_Agent',self.ua.random)




```







### ip的变化方案，以及如何设置ip代理



西刺免费代理IP里面提供ip，自己写一个爬虫爬取里面的ip，爬取后放入数据库，或者文件当中，就有了ip代理的数据源



HTTP 和 HTTPS 是ip代理的一种类型

```python
将ip爬取出来，存入数据库， ，
写一个类，在从数据库动态获取ip，，检测ip是否可用，用百度网站请求检测，将不可用的ip删除。  
引用类，在middleware里来设置ip动态代理

代码：
import requests
from scrapy import selector 

class ip_dali(response):
    
    def 




```



# scrapy-proxies

```python
proxies   代理







```







限速很重要 最稳定的ip还是自己的ip



tor洋葱网络，可以层层包装，完成匿名访问，黑客用的比较多，比较安全，比网上搜索的ip代理，甚至收费的ip代理都要稳定一些









怎么限速？？？





收费的Ip代理

pip install crawlera



```pu
通过selenium来模拟登陆知乎的时候，需要先设置点击事件，才可以真正进入登陆页面
```

模拟登陆成功后，就可以用browser来爬取其他页面的内容，怎么做？



### 已解决：

##### 当一个字段有多个值得时候，怎么存储  ------  拼接成字符串

scrapy的url去重原理   ？？？？？



### 问题：



当请求频率过快的时候 中间需要填写验证码，怎么做 ？？？？？？？？？

怎么爬app  --------    如何用Fiddler对Android应用进行抓包 ？？？？？？？？

怎么爬视频 ？？？？？？？？？ 

多进程多线程爬虫 ？？？？？



缓存？？？？？？

如何将selenium集成到scrapy中   写一个中间件  返回httpresponse对象

如何将scrapy_redis集成到scrapy中

如何将 bloomfilter  集成到scrapy-redis中，改源码，在源码中添加 。。。。。

。。。。。。。。。。

如何将elasticasearch集成到scrapy中



selenium获取的数据为什么打印不出html？？？







今日头条，百度等，怎么用的爬虫？   elasticsearch

爬虫算是开发吗









(1)采用scrapy-redis分布式实现。

(2)攻破403反爬技术，所以必须通过Fiddler抓取二次访问的请求报头



## 















|      |      |
| ---- | ---- |
|      |      |





1.需求分析、技术方案选型，项目的架构设计、开发环境搭建；  

2.爬虫模块：对目标站点页面解析，进行深度抓取；

  3.中间件：构造请求和响应，处理特殊页面。

  4.管道模块：对关键目标信息进行持久化存储。

  5.爬取策略制定，友好抓取数据，防止反爬。  







腾讯招聘网  开发环境：Ubuntu、scrapy框架、scrapy-redis分布式组件、Redis  

项目简介：这个项目是对找工作方面信息的抓取，采用scrapy-redis分布式  实现。分布式使用Redis做为缓存数据库，利用Redis的高并发和I/O读写来  实现高速下载。抓取过程中发现职位均为JS动态加载的，导致信息抓取不完  整，故采用开启下载中间件，利用selenium + PhamtonJS的方式发送请求，  待数据加载完毕后返回提取。

  自主爬取的网站：天气网，腾讯空间，职友集  





增  insert

删 truncket  delete  ？？？？

查 select

改 uodate modify  ？？？？

语句

ajax加载出来的数据都是json吗，都是需要在控制台获取吗

## 





信号管理是用dispatch模块



当取出多种url，这些url的匹配规则不一样，怎么给他们分别传递到解析子页面的函数中，

放入队列还是scrapy本身有方法？？



如果有的链接里面的内容也是不一样的，有时是一样的，也就是对应的xpath也有不同的时候，应该怎么办？？？？？



爬取的页面中如果有表格，怎么存储，关联表格吗？？？

















