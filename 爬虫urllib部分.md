ping   www.baidu.com  -t         (-t是一直ping的意思)

会返回一个ip

直接在浏览器输入这个ip就是百度首页

这样就是没有经过DNS服务商，直接访问了web服务器



每个服务器都有自己的ip地址，可以通过这个ip地址根据这个域名的站点显示出来

http主要是用于请求的，获取与html相关的

```python
网页三大特征：
1.每个网页都偶有自己唯一的url（统一资源定位符）来进行定位
2.网页都使用HTML(超文本标记语言)来被描述页面信息
3.网页都使用HTTP、HTTPS(超文本传输协议)来传输HTML数据

爬虫的设计思路：
1.首先确定需要爬取的网页的url地址
2.通过http/HTTPS协议来获取对应的HTML页面
3.提取HTML页面里有用的数据：
	a:如果是需要的数据就保存起来
	b:如果是页面里其他的URL，就继续执行第二步
四、为什么选择python做爬虫？
可以做爬虫的语言哟很多，php，java，c/c++,python等

php 对多线程的支持不好，异步不好，而且对多线程异步支持不够好，并发处理能力很弱
爬虫是工具性程序，对速度和效率要求比较高

java 的网络爬虫生态圈也很完善，是python爬虫最大的对手，但是java语言本身很笨重，代码量很大。重构成本比较高，任何修改都会导致代码的大量变动，爬虫经常修改部分代码

c/c++ 运行效率和性能几乎最强，但是学习成本很高。代码成型比较慢。能用c/c++ 做爬虫是能力的表现，但是不是正确的选择

python 语法优美，代码简介，开发效率高，支持模块多，相关的http请求模块和HTML解析模块非常丰富。 还有强大 的scrapy框架一起成熟高效的scrapoy-redis分布式策略，而且调用其他接口也非常的方便。（胶水语言）

```



```python
1.python的基本语法知识

2.如何抓取HTML页面
 http请求的处理，urllib，urllib2,requests
 处理后的请求可以模拟浏览器，发送请求，获取服务器响应的文件

3.解析服务器响应的内容
 re,xpath,bs4(BeaytifulSoup4),jsonpath,pyquery等
 使用某种描述性语言来给我们需要提取的数据，定义一个匹配规则，符合这个规则的数据就会被匹配。

4.如何采集动态HTML、验证码的处理
通用的动态页面采集。Selenium+phantomjs
Selenium、自动化测试工具，网页写出的页面，需要对各个浏览器进行适配，所以就会用这个自动化测试工具加载浏览器让浏览器去跑，如果测试之后能通过，这个页面就可以通过了。
phantomjs、（无界面的浏览器）可以模拟浏览器的功能，但是不是一个真正的浏览器：模拟真实浏览器加载js、ajax等非静态的页面数据
机器识别，图像识别系统
Tesseract：谷歌维护的机器学习库，机器图像识别系统，识别文本信息。可以把验证码放进去，识别字符串。可以处理简单的验证码，复杂的验证码可以手动数据（打码平台）

5.scrapy框架  pyspider
    高定制高性能（异步网络框架twisted），所有数据下载速度非常快，提供了数据存储、数据下载。提取规则等组件

6.分布式策略：
	1.是否需要分布式
	2.人员成本机器成本
    scrapy-redis  在scrapy基础上添加了一套，以redis（基于内存存储的数据库）数据库为核心的一套组件。让scrapy框架支持分布式的功能，主要在redis里做请求指纹去重，请求分配，数据临时存储
    
7.爬虫、反爬虫、反反爬虫 之间的斗争：    
	其实爬虫做到最后，最头疼的不是复杂的页面，也不是晦涩的数据
    
    UserAgent、代理、验证码、动态数据加载、加密数据
    加密数据（把数据加密也得让浏览器可以正常显示，数据加密的秘钥其实就隐藏在某个js脚本里，找出来比较费劲，但是有这个思路）
    数据的加载是否值得费劲做反爬虫
    机器成本+人力成本>数据加载，就不反了，一般做到封ip就结束了。
    面子的战争。。。
    
    只要是真是用户可以浏览的网页数据，爬虫就一定能爬下来
```

#### 根据爬虫的使用场景分为两种爬虫 --- 通用爬虫、聚焦爬虫

```python
一、通用爬虫：
搜索引擎用的爬虫系统

1、目标就是尽可能把互联网上所有网页下载下来，放到本地服务器里形成备份，再对这些网页做相关处理（提取关键字、去掉广告），最后提供一个用户检索接口

2、抓取流程： 首先选取一部分已有的url，把这些url放到带爬取的队列里
		 从队列里面取出这些URL，然后解析DNS得到主机IP，（一个域名一定对应一个ip地址，但是一个ip地址不一定有域名），然后去这个ip对应的服务器里下载HTML页面,保存到搜索引擎的本地服务器里，之后把这些爬过的url放入已爬取队列
    	  分析这些网页内容，找出网页里其他的URL连接，继续执行第二部，直到爬取条件结束

3、搜索引擎如何获取一个新网站的URL：
	① 主动向搜索引擎提交网址
     http://zhanzhang.baidu.com/linksubmit/url
     做个人站点的时候，可以往这里面添加
	② 在其他网站里，设置网站的外链
	③ 搜索引擎会和DNS服务商进行合作，可以快速收录新的网站。
    
4.通用爬虫并不是万物皆可爬，也要遵守规则：
Robots协议：协议会指明通用爬虫可以爬取网页的权限
Robots.txt 并不是所有爬虫都遵守的，一般只有大型搜索引擎爬虫才会遵守
	个人的爬虫就不管了
    就是一个建议，可以不遵守、
    
5.通用爬虫工作流程：  爬取网页   存储数据  内容处理  提供检索、排名服务
6.搜索引擎排名：
 ①.PageRank值：根据网站的流量（点击量、浏览器、人气）统计，流量越高，排名越靠前，网站也越值钱。
 ②.竞价排名：谁给钱多，谁排名就高。

7.通用爬虫的缺点：
 ① 只能提供和文本相关的内容（HTML,Word，PDF）等，但是不能提供多媒体（音乐、图片、视频）和二进制文件，程序脚本等。
 ② 提供的结果千篇一律，不能针对不同背景领域的人，提供不同的搜索结果
 ③ 不能理解人类语义上的检索（为了解决这个问题所以有了聚焦爬虫）
-----------------------------------------------------------
很基本的在网址里输入www.baidu.com
后台发生了什么？

从协议层的角度来说：从http的角度来理解（网络有七层）http，https，ftp这些协议都是建立在tcp，ip这些协议基础上的，所以我们在说http的时候默认tcp，ip是连接成功的

网址里输入www.baidu.com，首先会把url地址发送到dns服务商做解析，解析回来的是ip（DNS就是把域名解析成ip的一种技术）

eg:
    ping   www.baidu.com  -t         (-t是一直ping的意思)

会返回一个ip

直接在浏览器输入这个ip就是百度首页
这样就是没有经过DNS服务商，直接访问了web服务器
每个服务器都有自己的ip地址，可以通过这个ip地址根据这个域名的站点显示出来

直接输入ip地址，没有访问DNS服务商，直接访问了web服务器
通过域名解析输入www.baidu.com，这样是先通过DNS解析，解析完成之后返回一个ip地址给浏览器，浏览器再根据这个地址发送到指定的服务器上面，每个服务器都有自己的ip地址

http主要是用于请求的 ，获取与html相关的
-----------------------------------------------------------
聚焦爬虫：
爬虫程序员写的针对某种内容爬虫
面向主题爬虫：面向需求爬虫，会针对某种特定的内容爬取信息，而且会保证信息和需求尽可能相关
```



百度快照、

百度快照的好处，就是某个网页被封住了或者被删除了，在百度快照里面可能会有备份，点开百度快照，会显示是百度什么时候爬下来的网站

搜索引擎不能爬网页的图片，只能爬文本内容，



### HTTP和HTTPS

```python
HTTP协议：超文本传输协议，是一种发布和接收HTML页面的方法
HTTPS 就是在HTTP的基础上加了一个安全套接层ssl，  
SSL主要用于web的安全传输协议，在传输层对网络连接进行加密，保证在internet上数据传输的安全
 HTTP  的端口是80
 HTTPS 的端口是443
```

浏览器发送HTTP请求的过程：

1. 当用户在浏览器输入url并回车，浏览器会向HTTP服务器发送HTTP请求，HTTP请求主要分为Get和Post两种
2. 我们在浏览器输入URL  http://www.baidu.com 的时候，浏览器会发送一个request请求去获取http://www.baidu.com 的html文件，服务器把Response文件对象发送回给浏览器
3. 浏览器分析response中的HTML，发现其中引用了很多其他文件，比如image、css、js文件。浏览器会自动再次发送request去获取图片，css文件或者js文件。
4. 当所有的文件都下载成功后，网页会根据HTML语法结构，完整的显示出来了

- 基本格式：

```python
scheme://host[:port#]/path/.../[?query-string][#anchor]
              
scheme:协议（如http,https,ftp）
host：服务器的ip地址或者域名
port#：服务器的端口（如果是走协议默认端口,缺省端口80）
path:访问资源的路径
query-string： 参数，发送给http服务器的数据
anchor锚点：指的是页面上指定的位置（跳转到网页的指定锚点位置）（比如我打开一个网站直接跳到评论区，就是因为有锚点）
              
```

### 客户端的HTTP请求

```python
headers的内容  两部分
1 协议参数
GET https://www.baidu.com/  HTTP/1.1
请求类型    请求地址 		协议版本号
2 请求头的信息
HOST：www.baidu.com     主机
connection:keep-alive   长连接（意思就是不关闭这个浏览器，这个链接会一直保存）   
cache-control:max-age=0 获取最大值的参数
Upgr-ade-Insecure-Requests:1  升级一个不安全的请求，如果你的访问的是一个https的url地址，但是你访问的是http的，可能会报不安全，这里是升级你的http为https
useragent: 你的浏览器的版本地址
Accept:  可以接受的文件内容的类型
Accept-Ecoding：gzip，deflate，sdch，br  有这个说明可以对网络传输的东西进行压缩，这样传输会更快一些 （写爬虫的时候尽量不要写这一行，因为可能会是乱码，还要做响应的解压缩的操作） 不用写
Accept-Language:  支持的语言（q代表权重，默认1为权重最高）不用写

cookie   存储本地浏览器的用户信息
```



## urllib2

urllib2之所以诞生是创始人认为urllib太过于笨重

```python
import urllib2
ua_headers = {
    'User-Agent':'.....'
}
request = urllib2.Request('http://www.baidu.com',headers = ua_headers)
#向指定的url地址发送请求，并返回服务器响应的类文件对象
response = urllib2.urlopen(request)
#服务器返回的类文件对象，支持python文件对象的操作方法
#read()方法就是读取文件里的全部内容，返回字符串
html = response.read()
#打印响应内容
print(html)
#返回http的响应吗，返回
print(response.getcode())
# 返回响应的url 防止重定向问题
print(response.geturl())
# 返回服务器响应的http报头信息
print(response.info())

urllib2默认是User-Agent:Python-urllib/2.7
User-Agent:是爬虫和反爬虫斗争的第一步，养成良好的习惯，发送请求的时候带着user-agent
有data数据为post请求，没有data数据为get请求
wd={"wd":"参数"}
urllib.urlencode(wd) 
wd=
用到encode的时候就用到urllib  其他的时候就用urllib2

#这样可以转换成原来的字符串
urllib.unquote("")  
```





```python
def loadPage():
# 作用：根据url发送请求，获取服务响应文件
# 
```

ajax方式加载的页面，数据来源一定是json

拿到了json，就是拿到了网页的数据

ajax的加载也有get和post两种

如果是get会在url上面显示出来，



```python

```



```python
import urllib2
#构建HTTPHandler处理器对象，支持处理http请求
#在HTTPHandler增加参数debugleven=1  设置为自动打开debug log模式
#程序在执行的时候，会打印收发包的信息2
http_handler = urllib2.HTTPHandler(debugleven=1)
#调用build_opener()方法构建一个自定义的opener对象，参数是构建的处理器对象
opener = urllib2.build_opener(http_handler)

request = urllib2.Request('http://www.baidu.com')

response = opener.open(request)
print(response.read())

通过相关的handler处理器来构建处理器对象，支持某些特定的功能

通过urllib2.build_opener()构建一个自定义的open的对象，可以用来调用open方法发送请求

```



代理处理器

```python
每个网站都会有流量统计，这个流量统计会记录所有访客的信息，必须 ua  ip   频率等

import urllib2
#代理开关，表示是否启用代理
proxyswitch=True
#
httpproxy_handler = urllib2.ProxyHandler({'http':'125.88.67.54:80'})
# 如果没有代理，也得传入一个字典
nullproxy_handler = urllib2.ProxyHandler({})

if proxyswitch:
    opener = urllib2.bulid_opener(httpprocy_handler)
else:
    opener = urllib2.bulid_opener(nullproxy_handler)
    #构建了一个全局的opener，之后所有的请求都可以用urlopen()方式去发送，也附带handler的功能
urllib2.install_opener(
opener.install_opener(opener)
)


```



```python
import urllib2
import os
#在环境变量里设置了才可以取出来
#获取系统环境变量的授权代理的账户和密码
#os.environ.get操作系统环境变量的方法，获取指定的系统的环境变量
proxyuser = os.environ.get('proxyuser')
proxypasswd = os.environ.get('proxypasswd')

authproxy_handler = 
# ma_mo_hacker 为用户名   sffqry为密码
#urllib2.ProxyHandler({"http":"ma_mo_hacker:sffqry@114.213.122.34:16816"})
urllib2.ProxyHandler({'http':proxyuser+':'+proxypasswd+"@114.213.122.34:16816"})
#构建一个自定义的opener
opener = urllib2.build_opener(authproxy_handler)
#构建请求
request = urllib2.Request('http://www.baidu.com')
#获取响应
response = opener.open(request)
#打印内容
print(response.read())
```



python是解释性语言，不是编译性语言，编译性语言把源码编译后就编译成可执行程序了，这样是没法找到你的源码的，除非反编译，反编译的代码也不一定完整

解释性语言：编译成就是代码，执行就是可执行程序



```
HTTPPassworkMgrWithDefaultRealm（）
这个类会创建一个密码管理对象，用来保存和HTTP请求相关的授权信息
ProxyBasucAuthHandler（） 授权代理处理器
HTTPBasicAuthHandler()    验证web客户端的授权处理器

```





```
 1 import urllib2
  2 
  3 test = 'test'
  4 
  5 password = '123456'
  6 
  7 webserver = '192.168.21.52'
  8 #构建一个密码管理对象，可以用来保存和HTTP请求相关的授权账户信息
  9 passwordMgr = urllib2.HTTPPasswordMgrWithDefaultRealm()
 10 
 11 #添加授权账户信息，第一个参数是realm如果没有指定就写None,后面三个分别为站点i
    p,账户和密码
 12 passwordMgr.add_password(None,webserver,test,password)
	httpauth_handler = urllib2.HTTPBasicAuthHandler(passwordMgr)
	
```



cookielib  httpCookieProcessor

cookielib 就是用来保存cookie的

```python
cookielib库里面有四个对象 一般用CookieJar，和MozillaCookieJar 类

CookieJar         父类
只能存储http的cookie值，不能和本地交互的

下面三个类是可以和本地交互的
FileCookieJar     子类


MozillaCookieJar  FileCookieJar的子类


LWPCoolkieJar     FileCookieJar的子类



```



```python
import urllib
import urllib2
import cookielib
cookie = cookielib.CookieJar()

#通过HTTPCookieProcessor()处理器类构件一个处理器对象，用来处理cookie
#参数就是构建的cookieJar()对象
cookie_handler = urllib2.HTTPCookieProcessor(cookie)
#构造一个自定义的opener
opener.urllib2.build_opener(cookie_handler)
#通过自定义的opener的addheaders的参数，可以添加HTTP报头参数
opener.addheaders = [('User-Agent','sxxxxxxxxxxxxxxxxxxxxx')]
url='http://www.renren.com/PLogin.do'
#需要登陆的账户名 密码
data = {'email':'','passwork':''}

#通过urlencode()编码转换
data = urllib.urlencode(data)

#第一次post请求，发送登陆需要的参数获取cookie
request = urllib.Request(url,data=data)

#发送第一次post请求，发送登陆需要的参数，获取cookie（如果登陆成功的话）
response = opener.open(request)

#第二次可以是get请求，这个请求将保存生成的coolie一并发到web服务器，服务器会验证cookie通过
response_deng = opener.open('http://www.renren/com/884928/profile')
#获取登陆后才能访问的页面信息
print(response_deng.read())

r'http://www.baidu.com'
在这里r的作用是为了防止字符串被转义字符所影响，原样输出
u'xxxxxx'
代表unicode字符串


解析库  请求库




? 匹配前一个字符1或0次
+ 匹配前一个字符1或无限次
* 匹配前一个字符0或无限次


re.I  表示忽略大小写
re.S  全文匹配

pattern=re.complie(r'([a-z]+) ([a-z]+)'，re.I)

pattern.match('Hellow world hellow pYRHON')
group(0)  代表所有的字符串，全部打印出来
group(1)  代表打印第一个字符串
group(2)  代表打印第二个字符串

search  从任意位置查找。匹配一次
pattern = re.complie(r'\d+')
pattern.search(r'shaubs229jd32')

findall 匹配全文，返回列表
```



```python
http://temp.163.com/special/00804KVA/cm_yaowen_02.js?callback=data_callback
```

```python
????
w   w+   wb   wb+
a   a+   ab   ab+

```

## json

```python
json只有{} 或[]  或两者
json.dumps() 序列化的时候默认使用ascii编码
添加参数 ensure_ascii=False  禁用ascii编码，按utf-8编码

#识别编码格式
>>> chardet.detect(json.dumps(dictStr))
{'confidence':1.0,'encoding':'ascii'}
#意思是100%的相似度是ascii的编码

>>> chardet.detect(json.dumps(dictStr，ensure_ascii=False))
{'confidence':0.99,'encoding':'utf-8'}
#意思是99%的相似度是utf-8的编码

jsonPath
是一种信息抽取类库，是从JSON文档中抽取指定信息的工具，提供多种语言实现版本

jsonPath对json相当于xpath对xml
安装方法： 在 https://pypi.python.org/pypi/jsonPath  下载
    解压后执行  python setup.py install
    
官方文档： https://goessner.net/articles/JsonPath
    

```

```python
取json的数据
import urllib2
import json
import jsonpath
url = 'sssss.json'
headers = {}
request = urllib2.urlopen(request)
# 取出json文件的内容,返回的格式是字符串
html = response.read()
# 把json形式的字符串转换为python形式的Unicode字符串
unicodestr = json.loads(html)
# python形式的列表
# $..name 匹配格式  表示匹配name的值
city_list = jsonpath.jsonpath(unicodestr,"$..name")
for item in citylist:
    print item

# 正常情况下，只有英文的时候，是不会做任何编码格式的
# 只有有中文的情况下，会转成ascii编码格式，ensure_ascii 默认为true
# dumps()默认中文为ascii编码格式
# 禁用acsii编码格式，返回的是unicode字符串方便使用。所以写入的时候需要encode
array = json.dumps(city_list,ensure_ascii=False)
with open ('lahoucity.json','w') as f:
    f.write(array.encode('uth-8'))
```

```python
json.dump()
将python内置类型，序列化为json对象后写入文件
json.load()
将json形式的字符串转换为python的
```

```python
不同的web服务器会根据不同的浏览器，发送不同的页面，正常情况下ie为标准格式
所以我们用xpath做解析的时候，请求头就写成ie的useragent，如果写其他的
```

## xpath 

xpath的模糊查询

```python
//div[contains(@id,'qiushi_tag_')]
# 这样就可以匹配出div的id里面有qiushi_tag_的部分，
```

```python
import urllib2
from lxml import etree
import json

url=''
headers = {}
request = urllib2.Request(url,headers=headers)
html = urllib2.urlopen(request).read()
#响应范湖的是字符串，解析为HTML DOM模式 
text = etree.HTML(html)
# 模糊查询返回的所有段子的节点位置,contents() 模糊查询方法，第一个参数是要匹配的标签，第二个参数是标签名的部分内容
items={}
node_list = text.xpath("//div[contains(@id,'qiushi_tag_')]")
for node in node_list:
    username = node.xpath('./div/a/@title')[0]
    image = node.xpath('.//div[@class="thumb"]//@src')[0]
    #取出标签下的内容 段子内容
    content = node.xpath('.//div[@class="content"]/span')[0].text
    #取出标签里包含的内容 点赞
    zan = node.xpath('.//i')[0].text
    #取出标签里包含的内容 评论
    comments = node.xpath('.//i')[1].text
    items = {'username':username,
             'image':image,
             'content':content,
             'zan':zan,
             'comments':comments,
            }
    #将unicode转码，加上换行
    with open('qiushi.json','a') as f:
        f.write(json.dumps(items,ensure_ascii=False).encode('utf-8')+'\n')
```

## 多线程机制

CPU  进程  线程  锁 队列

CPU：计算机计算的基本单位，是用来执行任务的，cpu里面可能包含多个进程，一个进程里可能包含多个线程，线程之间需要执行任务的话，必须要加锁（阻塞）

父线程和子线程之间的关系，只要父线程结束，不论子线程是否结束，都会一起结束

队列相当于内存里面的堆，先进先出

创建线程的时候，可以通过队列的方式保存数据。



首先创建多个线程，重写父线程，

线程启动的时候，对应线程里自定义启动的run方法，线程启动start方法，线程启动的时候，功能就在run里面实现



调用解析线程去做这件事，每个解析线程都会解析这个页面，比如创建三个解析线程，分别对应解析html里面的源码，这样速度就很快了

什么时候使用多线程，什么时候使用多进程

多进程一般用于密集的数据计算（大批的数据处理）因为他是单独的一个任务

一个进程一次只能执行一个任务

线程适合密集型的io任务

爬数据存数据就是io  ，使用多线程，不用多进程，使用网络阻塞

网络性能的瓶颈就在io这块



多线程爬虫的逻辑

首先有一个页码的队列，每一个线程爬页面的时候，给他一个页码让他组合起来去爬

比如第一次为 1 第二次为 2，直到页码队列为空的时候，采集就结束了，采集完之后，调用解析线程，解析每个数据队列里面的html源码，直到数据队列为空，说明解析的队列也结束了

这个程序就结束了。

页码的

1. 计算机的核心是cpu，cpu承担了所有的计算任务

2. 一个cpu核心一次只能执行一个任务 ，多个cpu核心同时就可以执行多个任务

3. 一个cpu同时（一次）只能执行一个进程，其他进程处于非运行状态

4. 进程里面包含的执行单元叫线程:, 一个进程可以包含多个线程

5. 一个进程的内存空间是共享的，每个进程里的线程都可以使用这个共享空间

   一个线程在使用这个共享的时候，其他线程必须等他结束

6. 通过"锁"实现（锁的作用，防止多个线程同时使用一个内存空间的）

   先使用的线程会空间上锁，其他线程就在门口等待，打开锁之后再进去

##### 什么是进程：表示程序的一次执行

##### 什么是线程：CPU运算的基本调度单位

##### GIL: 全程解释器锁， python里的执行通行证，而且只有一个，拿到通行证的线程就可以进入CPU执行任务，没有就等待。

##### python的多线程适用于 : 大量密集的I/O处理

##### python的多进程适用于 : 大量的密集并行计算

多进程的速度是远大于多线程的

##### scrapy 专门的异步网络框架 很多协程在做处理



设备管理器---->处理器

intel(R) Core(TM)i7-4870HQ CPU @ 2.50GHz

intel的 Core有个超线程技术，一个核心可以做两个线程  

Q 代表4核心 H 代表焊接上去的，电压会高一点，性能会稍微好一点 U是低电压版，功率很低，CPU主频也低，消耗会少一点，一般用于超极本，或计算量不大的电脑，带K的一般是台式机电脑的，代表CPU可以超频，带M的指移动端的，X是当前intel最顶尖的，旗舰版的CPU

intel(R) Core(TM)i7-8550U CPU @ 1.80GHz

```python
#线程库
import threading
#队列
from Queue import Queue
#解析库
from lxml import etree
#请求
import requests
#json文件处理
import json

class Thread_crawl(threading.Thread):
    def __init__(self,thread_name,page_queue,data_queue)
    # 调用父类的初始化
    # 如果有一个父类，下面两种方法都可以，但是如果代码重构，有多个父类继承的话，建议使用下面的
    #threading.Thread.__init__(self)
    super(Thread_crawl,self).__init__()
    #线程名
    self.thread_name = thread_name
    #页码队列
    self.page_queue = page_queue
    #数据队列
    self.data_queue = data_queue
    #请求报头
    self.headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36"}
    def run(self):
        #
        while not CRAWL_EXIT:
            try:
            #取出一个数字，先进先出
            #有个可选参数block，默认值是True
            #1.如果队列为空，block为True的话，并不会结束程序，就会进入阻塞状态，直到队列有新的数据
            #2.如果队列为空，block为false的话，就会弹出一个queue.empty()异常
                page = self.page_queue.get(False)
                url = "https://www.qiushibaike.com/8hr/page/"+str(page)+"/"
			   content = requests.get(url,headers=self.headers)
                self.dataQueue.put(content)
                # html = etree.HTML(content)
        	except:
                pass   
            
# 做一个全局变量，如果队列为空的话，就做一个标记，让他退出
CRAWL_EXIT = False
PARSE_EXIT = False
    
def main():
    #页码的队列，表示10个页码
    page_queue = Queue(10)
    #放入了1-10的数字，先进先出
    for i in range(1,11):
        pageQueue.put(i)
        
    #采集结果（每一页的html源码）的数据队列，参数为空表示不限制，
    data_queue = Queue()
    
    #三个采集线程的名字
    crawl_list = ['采集线程1号','采集线程2号','采集线程3号']  
    
    #存储三个采集线程
    threadcrawl=[]
    for thread_name in crawl_list:
        #Thread_crawl 是一个类，他会创建一个对象thread, 这个线程的对象执行的时候会调用 start 方法
        thread = Thread_crawl(thread_name,page_queue,data_queue)
        thread.start()
        
        threadcrawl.append(thread)
        
    #等待pagequeue队列为空，也就是等待之前的操作执行完毕
    while not page_queue.empty():
        pass
    # 如果page_queue为空，采集线程退出循环
    global ERAWL_EXIT
    ERAWL_EXIT = True   
    print 'page_queue为空'
    
    for thread in threadcrawl:
        thread.join()
        print('1')
    
if__name__=='__main__':
    main()
```



分布式是加分项但不是必须的



https://weibo.cn/pub/ 微博登录接口

phantomjs是需要下载的

http://phantomjs.org.download.html  下载之后可以放在环境变量里面  

```python
driver.get('http://www.baidu.com/')
#获取页面名为wrapper的id标签的文本内容
data = driver.find_element_by_id('wrapper').text
print(data)  #打印数据内容
print(driver.title) #打印标题，百度一下，你就知道
#生成当前页面快照并保存
driver.save_screenshot('baidu.png')
#id='kw'是百度收缩输入框，输入字符串 ‘长城’
driver.find_element_by_id('kw').send_keys(u'长 城')
#id='su'是百度搜索按钮 click()是模拟点击
driver.find_element_by_id('su').click()
driver.save_screenshot('baidu.png')  #获取新的百度快照
print(driver.page_source)  #打印渲染后的源代码
print(driver.get_cookies()) # 获取当前页面的cookie

ctrl+a 全选输入框内容
driver.find_element_by_id('kw').send_keys(Keys,CONTROL,'a')

ctrl+x 剪切输入框内容
driver.find_element_by_id('kw').send_keys(Keys,CONTROL,'x')

ctrl+a 输入框重新输入内容
driver.find_element_by_id('kw').send_keys('itcast')

模拟Enter键回车
driver.find_element_by_id('su').send_keys(Keys.RETURN)

清除输入框的内容
driver.find_element_by_id('su').clear()

生成新的页面快照
driver.save_screenshot('itcast.png')

获取当前url
print(driver.current_url)

关闭当前页面，如果只有一个页面，关闭浏览器
driver.close()

#弹窗处理
当出发了某个事件后，页面出现了弹窗提示，处理这个提示，或者获取提示信息的方法
alert = driver.switch_to_alert()

#页面切换
一个浏览器肯定会有很多窗口，所以我们肯定要有方法来实现窗口的切换，切换窗口的方法
alert = 
# 等于-1说明没找到这个元素，不等于-1说明
if driver.page_source.find('shark-pager-disable-next')!=-1:
    break
    
#执行js语句
# 向下滚动10000像素  js='document.body.scrollTop=10000'
driver.execute_script(js语句)
time.sleep(10)
```



自动化测试工具，用无界面浏览器爬取动态页面，分页的ajax加载技术的都可以爬取



采集线程用到网络io  解析线程是要往本地写，就是磁盘io



unicode

前面加个u就可以了

print u'房间名'+.....

### 自动化测试模块抓取

```python
#测试模块里面确实用的比较怪异，首先不需要对象
#只需要导入unittest这个模块 然后写main就可以执行
import unittest
from selenium import webdriver
from bs4 import BeautifulSoup
class douyu(unittest.TestCase):
    #初始化方法参数是固定的，必须是setUp()
    def setUp(self):
        self.driver=webdriver.PhantomJS()
        self.num=0
        self.count = 0

    #测试方法必须有test字样开头
    def testDouyu(self):
        self.driver.get('https://www.douyu.com/directory/all')
       	while True:
            soup = bs(self.driver.page_source,'lxml')
            names = soup.find_all('h3',{'class':'ellipsis'})
            number = soup.find_all('span',{'class':'dy-num fr'})
            
	#zip(names,numbers)将name和number这两个列表合并为一个元祖：[（1,2）（3,4）...]
	#
            for name,number in zip(names,numbers):
            	print(u'观众人数：'+number.get_text().strip()+u'\t房间名称：'+ name.get-text().strip())
                self.count+=int(number.get_text().stript())
                self.num+=1
            #如果在页面源码里找到下一页为隐藏的标签就退出程序
            if self.driver.page_source.find('shark-pager-disable-next')!=-1:
                break
            #如果没有退出就抑制剂点击下一页
            self.driver.find_element_by_class_name('shark-pager-next').click()
    #测试结束执行的方法
    def tearDown(self):
        #退出phantomjs()浏览器
        # 统计有多少人做直播
        print('当前网站直播人数'+str(self.num))
        print('当前网站观众人数' +str(self.count))
        #统计当前有多少人在看直播（观众人数累加即可）
        
        self.driver.quit()
        
if __name__=='__main__':
    # 启动测试模块
    unittest.main()
```

### 识别图片文字（python使用机器学习库）

```python
pip install pytesseract
from pytesseract import *
from PIL import Image
#打开图片
image = Image.open('fonts_test.png')
text = image_to_string(image)
print(text)
```































