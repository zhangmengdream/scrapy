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

基于girhub 开源的专门用来随机切换user-agent的

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





##### 



















