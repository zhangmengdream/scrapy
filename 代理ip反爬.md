### flask+redis维护一个代理池

```python
redis主要是维护这个池，提供池的队列存储，flask用来实现代理池的接口
可以通过flask从代理池拿出一个代理
代理池的要求：
1.多站抓取，异步检测（从各大免费ip网站抓取，通过异步请求的方式检测代理的可用性，）
2.定时筛选，持续更新
3.提供接口，易于获取（用python的flask）

代理池架构
维护一个代理的队列

这个队列可以用python的数据结构来存储，也可以用数据库来存储

获取器：从各大网站平台上把ip获取下来，临时存储进一个数据结构
过滤器：把这些代理进行筛选，把这些代理拿下来之后，就测试一下这些代理，用来请求网站，查看返回值
把过滤后的代理，留下来的可用的代理放入代理队列
定时检测器：时间长了之后，代理队列里面的一些代理可能会被封，不可用，就需要定是检测的装置，定是检测器，定时的从里面拿出一些代理，然后用过滤器里面的方法，把可用的留下，不可用的剔除掉，把可用的代理重新放回代理队列，做定时的检测，保证代理队列持续更新

API：需要实现一个接口，通过接口的形式，拿到代理队列里面的代理的内容
然后通过请求接口，从网页里面获取到代理的ip和端口，（为了方便提取实现的）
```



![img](C:\Users\dream\Desktop\捕获.PNG)

#### 代理池的实现

```python
如果检测到代理池里的内容太少了，就会启动爬取的操作，也就是从网站上抓取

多进程的方式
1. 一个进程是检测代理池的代理（定时检测器valid_froxy--参数是时间的参数，意思是隔多少秒进行一个定时检测）
conn = RedisClient()   redis连接的对象
# 利用io http这个库,实现异步检测的方法
#aiohttp 是一个异步请求库，可用利用他来做一些异步的请求检测之类的，所以这样请求库可以做一个非常高效的异步检测
#从代理的一端取出代理进行检测，另一端进行放入队列
async def test_single_proxy(self,proxy):
    async with aiohttp.Client
    
 2. 另一个进程是（从各大网站获取代理）把可用的代理放入队列



---------------------------------------------------------
git上的
Germey/ProxyPool
```







```python
spider.py

from urllib.parse import urlencode
import requests
import pymongo
from requests.exceptions import ConnectionError
from pyquery import PyQuery as pq
#把配置里面的所有变量引入过来
from config import *
#mongodb的连接信息
client = pymongo.MongoClient('MONGO_URI')
#db名称直接指定为微信
db = client['MONDO_DB']
base_url = 'http://weixin.sougou.com/weixin?'
#需要加入cookies，因为需要登陆之后才能看到登录之后的搜索信息
headers = {
    'Cookies':'',
    'Host':'',
    'User-Agent':''
}
# key_word='风景'

#一个生成代理的接口，用这个接口就可以拿到一个代理,为None说明这个代理没有拿到
# proxy_pool_url = 'http://127.0.0.1:5000/get'
#代理默认设置为空 ，也就是默认不使用代理的，用本机ip爬取，如果出现错误，再使用代理爬取
proxy = None
def get_proxy():
   try:
       response = requests.get(proxy_pool_url)
       if response.status_code == 200:
	   		return response.text
        return None
    except ConnectionError:
        return None
    
def get_html(url):
    print('Crawling',url)
    print('Trying Count',count)
    global proxy
    if count >= MAX_COUNT:
        print('Tried Too Many Counts')
        return None
    try:
        if proxy:
            proxies = {
                'http':'http://'+proxy
            }
        #拦截302的操作，  requests就会默认的处理跳转的  
        #allow_redirects 这个参数的意思是不让他自动处理这个跳转
        #这样我们才能正常拿到302的状态码
        #如果有代理就用代理的请求，如果没有就用正常的请求
        response = requests.get(url,allow_redirects=False,headers=headers,proxies=proxies)
        if response.status_code == 200:
            return response.text
        if response.status_code == 302:
            print('302')
            #说明ip被封了 需要设置代理
            proxy = get_proxy()
            if proxy:
                print('Using Proxy',proxy)
                return get_html(url)
            else:
                print('Get Proxy Failed')
                return None
    #捕捉异常，如果没有正常请求成功，重新调用这个方法，进行失败后的重试(重新调用自己)
    except ConnectionError:
        return get_html(url)
		
def get_index(keyword,page):
#构建get请求的参数
	data = {
        'query':'keyword',	
        'type':2,
        'page':page
        }
    #用urlencode 组建成get请求参数的样子
    queries = urlencode(data)
    url = base_url+queries
    html=get_html(url)
def parse_index(html):
    doc = pq(html)
    items = doc('.new-box .new-list li .txt-box h3 a').items()
    for items.attr('href')
def get_detail(url):
    try:
        response = request.get(url)
        if response.status_code==200:
            return response.text
        return None
    except ConnectionError:
        return None
#解析源代码的方法
def parse_detail(html):
    try:
        doc = pq(html)
        title = doc('.rich_media_title').text()
        content = doc('.rich_media_content').text()
        date = date.doc('#').text
        return {
            'title':title,
            'content':content,
            'date':date,
            'nickname':nickname
        }
    #如果出现错误直接返回None
    except XMLSyntaxError:
        return None
#保存到mongodb
def save_to_mongo(data):
    # 调用插入的方法，进行去重，第一个参数是需要查询匹配的目标基准点
    if db['articles'].update({'title':data['title']},{'$set':data},True)
    #如果查询到则进行插入，如果查询不到设为不成功
    	print('Save to Monge',data['title'])
    else:
    	print('Save to Monge Failed',data['title'])
    
    
#分页循环
def main():
    for page in range(1,101):
        html = get_index(key_word,page)
        if html:
            article_urls = parse_index(html)
            for article_url in article_urls:
                article_html = get_detail(article_url)
                if article_html:
				  article_data = parse_detail(article_html)
                    print(article_data)
                    if article_data:
                        save_to_mongo(article_data)
if __name__=='__main__'：
    get_index('风景'，1)
    
    
# config.py里面配置一些常用信息
PROXY_POOL_URL = 'http：//127.0.0.1:5000/get'
KEYWORD = '风景'
MONGO_URI = 'localhost'
MONGO_DB = 'weixin'
MAX_COUNT = 5

#代理ip是我们自己维护的ip，这些ip可能是不太稳定的，，效果可能不太好
#自己买ip 需要改写get_proxy这个方法就好了
```



#### cookie池的要求redis+flask维护cookie池

自动登陆更新

定时验证筛选 : 如果cookie不能请求正常页面，就把cookie从cookie池里取出拿掉

提供外部接口 : 

![img](C:\Users\dream\Desktop\gitscrapy\cookie池架构.PNG)



#### cookie池的实现























































