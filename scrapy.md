elasticsearch 是最流行的分布式开源搜索引擎，被大量的应用在很多大公司中



spider

source == activate.bat



# navcate

navcate链接不上时，也就是本机ip可以链接，但是其他的不能连接时，需要设置权限

在mysql里面设置权限

mysql提供的一个权限设置的命令

```python
#权限的赋值命令

GRANT ALL PRIVIEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
# GRANT ALL PRIVILEGES 设置所有权限的命令
# *.* 表示所有的表都要设置权限
# root  表示通过root表链接过来
# '%' 代表所有外部ip
```

```python
#刷新权限
flash privileges;
```

此时就可以链接成功navcate了



## 豆瓣源

```python
pip install -i https://pypi.douban.com/simple/ django
豆瓣镜像会加快我们的下载
```



爬虫基础知识：

技术选型：

scrapy



request和beautifulsoup 都是库 

scrapy是框架



爬虫能做什么

1.搜索引擎---百度/Google/垂直领域搜索引擎

2.推荐引擎---今日头条，，根据你的浏览器习惯，猜测你可能感兴趣的内容，然后把数据

推送给我们

3.机器学习的数据样本

4.数据分析（金融数据分析），舆情分析等



正则表达式：

1.在处理字符串的过程中非常常见

2.正则表达式的简单应用

①特殊字符

②dd

```opy
. 代表任意字符
* 代表前面的字符可以重复任意遍
? 非贪婪匹配
() 只会取出括号内匹配的内容
+ 代表加号前面的字符，出现至少一次
| 或
[] 用途： 匹配区间
		 只在某几个值中间取
		 里面的 . * 没有特殊含义了
\s 空格 
\S 只要部位空格都可以
\w [S-Za-z0-9_]
\W 与小写w相反匹配
([\u4E00-\u9FA5])	unicode编码  只要是汉字都可以（匹配一个） 
匹配多个汉字的话 ([\u4E00-\u9FA5]+)
（） 提取子字符串
\d  数值型

```



re下面的函数

```python
re.match(正则表达式，匹配字符串)


# 用或的关系时需要加上（）  （x | $）  这样表示这两个的或
# 在字符串中提取两段时，用group(1)/group(2)来提取 分别代表第一个和第二个
```

字符串编码 

unicode编码

utf8编码



python3中将所有的字符，全部转换成unicode



#### 通过scrapy搭建自己的爬虫









hash解决冲突的方法

bitmap

bloomfilter --- 去重的实现

bitmap





深度优先和广度优先算法（需要掌握算法）

1.深度优先：深度优先的算法是通过--递归实现的

2.广度优先：先访问完兄弟节段，再访问子节点（分层的方式访问，按层次遍历-队列实现的

```python
#深度优先的过程
def depath_tree(tree_node):
    if tree_node is not None：
    	print(tree_node.data)
		if tree_node._left is not None:
            return depth_tree(tree_node._left)
         if tree_node._right is not None:
            return depath_tree(tree_node._rignt)
```



递归函数一定要理解跳出规则，不然的话栈会溢出的



```python
#广度优先的过程
def depath_tree(tree_note):
    if tree_note is not None:
        print(tree_note.data)
       	if tree_note._left is not Null:
            print(tree.note._left.data)
            if tree_not._right is not Null:
                print(tree.note._right.data)
            
```

node.elem  打印a的值





# scrapy

```python
#安装scrapy命令
pip install -i https://pypi.douban.com/simple/ scrapy
#新建scrapy工程命令
scrapy startproject projectname
#创建好了scrapy之后，只是一个scrapy工程框架，里面并没有具体的spider的模板，
scrapy genspider jobbole blog.jobbole.com
# 启动命令
scrapy crawl jobbole
```



```python
settings.py
存放spider的路径
SPIDER_MODULES=['ArticleSpider.spiders']

pipeline 存放与数据相关的地方

middleware 存放我们自己定义的middleware

items 类似jango中的form

spider文件夹，存放我们具体某个网站的爬虫（比如：知乎、伯乐在线的爬虫）

默认并没有为我们创建一个模板，但是提供了一个命令
scrapy genspider name url 
eg: 
    cd AritcleSpider
    scrapy genspider jobbole blog.jobbole.com
    
   
```



```python
@class  选取所有名为class的

```













response里面本身是带有xpath的方法的

```python
# shell环境下调试的命令
scrapy shell http://blog.jobbole.com/113909/

```



strip()方法，去掉回车换行符



replace（）方法替换

replace('.','') 把  . 替换成空格

有空格的时候调用strip()

## xpath的内置函数

```python

contains() 匹配class中的一个（包含）
>>> ("//span[contains(@class,'vote-post-up')]")

extract() 方法查看里面的值
>>> title = response.xpath("//div[@class='entry-header']/h1/text()")
>>> title.extract()[0]
'我曾误删了公司的数据库，但还是活下来了'

extract_first()与extract()[0]
可能这个数组为空
此时取extract()[0]有可能会抛出异常，这时候就需要做异常处理
extract_first()这个就不需要异常处理了，为空的话就会返回null
```





正则中的group(0),(1)  用法？

## css选择器提取元素

```python
#通过css选择器提取字段
response.css()

::text 是css的味蕾选择器取text的值
```



srapy的Request类

```python
from scrapy.http import Request
yield Response(url=url,callback=def)
将提取出来的url转给def来进行数据提取
```

scrapy的parse

```python
from urllib import parse	 #python3的用法
import urlparse 			# python2的用法
parse.urljoin有两个参数(base,url)  完成url的拼接
第一个参数传递url进来的时候，不用传递主域名，直接传递任何一个url，他会主动把这个主域名提取出来
如果传递进来的url有域名，则base里面的域名就起不了作用了
url=parse.urljoin(response.url,post_url),callback
```





yield  urljoin(response.url,post_url)

itemloader





## scrapyitems

```python
为了将爬去的东西进行一个完整的格式化，scrapy为我们提供了一个items类，可以帮我们指定数据类型，就像jango里面的model一样，指定一个字段。
item就像使用一个字典，但是比字典的功能齐全

路由到pipelines里面，在pipelene里面集中处理数据的保存，去重
```



scrapy爬虫的目的就是通过非结构的数据源，提取程结构数据

url可以做成一个md5，这样url就可以变成一个唯一的，且长度固定的一个值

先写一个md5的函数,然后导入到jobbole中，在url字段中使用md5函数







## 通过pipeline将数据保存到mysql中

爬取数据后，跟数据库，或者文件打交道

```python
简单的保存爬取到的数据  --  小规模的项目可以用
scrapy crawl dmoz -o items.json 
```



```python
创建一个保存json的pipeline，保存json文件的第一步是打开json文件
在init初始化的时候设置打开json文件

pipeline会拦截item，可以在这里把item保存在数据库等任何想要保存的地方

保存到mysql之前，首先是数据库的设计

```



数据库和item的关系，相当于jango中model和from之间的关系

先根据item建好表格





```python
pip install -i https://pypi.douban.com/simple/ mysqlclient

sudo apt-get install libmysql
```

## Itemloader



scrapy的itemloader机制，可以让我们的维护工作变得更加简单







scrapy是基于深度优先来完成的





可以在子页面提取具体数据之后（yield下面），加上解析页面的逻辑（parse里面的爬取url逻辑）这样可以深度爬取

## 爬取知乎过程

```python
# 整个爬虫的入口在(整个spider的入口) 所以不管start_url 是什么，我们首先发送的都是这里面的url
# callback=self.login这里指明的返回
def start_requests(self):
	return [scrapy.Request('https://www.zhihu.com/#signin',headers = self.headers,callback=self.login)]

# 这里调用完之后会调用login函数
# login调用到FormRequest之后，返回check_login返回函数
# 这里是模拟登陆，配置登陆需要的参数
def login(self,response):
    response_text = response.text
    match_obj = rw.match('.*name="_xsrf" value="(.*?),response_text,re.DOTALL"')
    xsrf = ''
    if match_obj:
        xsrf = (match_obj.group(1))
    if xsrf:
        post_url = 'https://www.zhihu.com/login/phone_num'
        post_data = {
            '_xsrf':xsrf,
            'phone_num':18715529161,
            'password':'zhangmeng',
        }
        # FormRequest 开启模拟登陆
        return [scrapy.FormRequest(
            	url =post_url,
                formdata = post_data,
                headers = self.headers,
                callback = self.check_login
            )]
    

# 验证服务器的返回数据 判断是否成功
def check_login(self,response):
    text_json = json.load(response.text)
    if 'msg' in text_json and t ext_json['msg'] == '登录成功'：
    	for url in self.start_urls:
            # scrapy会对request的URL去重(RFPDupeFilter)，加上dont_filter则告诉它这个URL不参与去重。
            
            # 爬虫应该有的读取行为在这里执行（知乎完成登陆之后执行的）
            # 这是一个正式的请求，我们没有给他设置callback
            # 下面开始一个真正的爬虫逻辑 --- 深度优先的逻辑 parse
            yeild scrapy.Request(url,dont_filter=True,headers=self.headers)



#所有的request都要设置headers
```









① 用一个pipeline处理所有的item

② 用一个pipeline处理一个item 或者一个网站的item

 

如果有上百个网站就要向，mysql发起上百个连接（非常不合理），所以选择策略①

实际上真正的开发当中，不同的爬虫，或者一个爬虫，爬取的数据，我们需要存储到不同的数据库当中，这时候可以根据不同的数据库建立pipeline



现在例一个 用一个pipeline处理所有的item

（只有sql语句部分才是变化部分）

 



## 更新问题

先爬取了request写入到了数据库里面，再次进行调试，插入数据就会失败（主键冲突，插入了重复的数据），有可能我们第二次爬取数据的时候，内容已经做了更新（点赞数、投票数量，更新时间等有更新）

解决方法：简单的插入逻辑已经满足不了我们的需求，所以这时候，我们可以没有就插入，有就更新里面的某些字段

这里需要用到mysql的一些语法知识

```markdown
# 如果存在会更新，不存在会重新插入
def get_insert_sql(self):
        #插入知乎question表的sql语句  
    insert_sql = """
        insert into zhihu_answer(zhihu_id, url, question_id, author_id, content, parise_num, comments_num, create_time, update_time, crawl_time) 
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        ON DUPLICATE KEY UPDATE content = VALUES(content),comments_num = VALUES(comments_num),update_time = VALUES(update_time)    
    """
```

```markdown
ON 代表如果条件满足
DUPLICATE
VALUES 是会从后面的参数中取值的
这样的话真正的变量传递进来也不需要去变了，会从ON 里面去取
这种用法是mysql特有的
```



爬取知乎过快，或者过于频繁的时候，用一个User-Agent知乎会返回给我们403错误的，实际上就是一种反爬机制

会检查我们的User-Agent，同一个User-Agent在同一个ip下请求过于频繁的话，会返回给我们一个403的错误页面

解决方法：随时更换我们的User-Agent

 



解决方法：反爬中会介绍

防止爬虫被禁止的章节中，介绍验证码破解





debug的快捷键 f6  f8 





1490970181893

t = str(int(time.time() * 1000)) 



























start_url  有多个值的时候，选择哪一个来开始调用爬取的 ？？？？

DLL ？？？







```python
#在一个单调函数中返回多个request 即 item 的例子
import scrapy 
from myproject.items import MyItem

class MySpider(scrapy.Spider):
    name = 'douban'
    allow_domains = "
     www.douban.com
    "
    start_url = ['http://www.example.com/1.html',
                'http://www.example.com/2.html',
                'http://www.example.com/3.html',]
   	def parse(self,response):
        sel = scrapy.Selector(response)
    	for h3 in response.xpath()
        yield MyItem(title = h3)
        
        for url im response.xpath()
        yield scrapy.Request(url,callback=self.parse)
    
```



# 验证码的识别

一、编码实现（tesseract-ocr）  ----  不建议  识别率比较低，效率也比较低



二、在线打码 -- 识别率很高 90%以上（依靠代码识别技术），识别速度比较快，平台提供api，我们自己调用 （平台：云打码）

```python
云打码
开发者账号和用户账号都需要注册

开发者账号：是和云打码合作的一种开发者，




```



三、人工打码 -- 人在识别









##  如何给不同的spider设置不同的setting



如何禁用scrapy的cookie 

```python
setting里面有一个参数
#是否禁用cookie
在这里我们将cookie禁掉，对方就无法通过cookie对我们进行跟踪
设置为False就会禁用我们的cookie，request就不会将我们的cookie带进去

COOKIES_ENABLED = True
有的网站会通过我们的cookie来判断我们是否是爬虫

# 限速（下载延迟）----  需要配置，跟着官方文档的具体设置即可
scrapy默认在每个网页下载之间的空隙是0的，也就是他遇到页面就会直接去下载

setting里面的
DOWNLOAD_DELAY=10 （这个值默认是0  设置成10 代表10秒钟下载一次）
AUTOTHROTTLE_ENABLED = True  设置为True代表开启这个扩展
autothrottle_enabled



```

 如何给不同的spider设置不同的setting

```python
在spider里面设置 --  就是设置自己的setting  比如这里是设置自己的禁用cookie为True
    custom_settings={
        "COOKIES_ENABLED":True
    }


```



## scrapy的进阶开发

##### scrapy的动态网站抓取

```python
Selenium 自动化测试框架
selenium的核心，是使用JavaScript操控浏览器

selenium支持分布式的

selenium实际上是操控浏览器的，中间是有一个drive的
每个浏览器有对应的drive

selenium只是一个接口，他调用的还是一个浏览器，所以需要用浏览器的drive来完成


代码：
from selenium import webdriver
from scrapy.selector import Selector
browser = webdriver.Firefox(executable_path='driver的文件路径')
browser.get('爬取的动态网址')
#将网页的内容展示出来  放进text
text=browser.page_source
t_selector = Selector(text=text)
#通过css匹配出具体的值
print(t_selector.css('.tb-promo-price .tb-rmb-num::text').extract())
#关闭网页，将浏览器退出
browser.close()
```

##### 通过selenium来模拟登陆知乎

```python
from selenium import webdriver
from scrapy.selector import Selector

browser = webdriver.Firefox(executable_path = 'driver的文件路径')
browser.get('https://www.zhihu.com/#singin')
#因为知乎登陆页面默认是手机验证码的 需要手动点击登陆才可以进入账号密码页面，所以需要先设置登陆的点击事件
browser.find_element_by_css_selector(".SignContainer-switch span").click()
#给账号密码输入值
browser.find_element_by_css_selector(".Login-content input[name='username']").send_keys('18715529161')
browser.find_element_by_css_selector(".Login-content input[name='password']").send_keys('zhangmengjie123')
#给登陆设置点击事件
browser.find_element_by_css_selector(".Login-content button.SignFlow-submitButton").click()

```

##### 设置不加载图片有chromedriver （用里面的prefs参数） --- 好处节省时间，加快加载速度

geckodriver如何设置？？

```python
# 设置　不加在图片的　driver
# ChromeOptions 这个类代表他的设置　　用这个类生成一个对象　chrome_opt
chrome_opt = webdriver.ChromeOptions()
# 给这个对象加一个不加载图片的值  设置为２表示不加载图片
prefs = {"profile.managed_default_content_settings.images":2}
chrome_opt.add_experimental_option("prefs",prefs)
browser = webdriver.Chrome(executable_path='/home/test/Desktop/geckodriver',chrome_options=chrome_opt)
browser.get("https://www.taobao.com")
```

##### phantomjs   无页面的浏览器  (phantomjs也可以设置不加载图片)

步骤和chrom请求页面相同 ，不推荐， 

```python
browser = webdriver.phantomJS(executable_path="")
browser.get('')
prine(browser.page_source)
browser.quit()
```

更加推荐chrome，chrome在无界面的情况下也可以使用



##### 如何将selenuim集成到script当中

一、设置一个中间件

```python
class JSPageMiddleware(object):
    # 通过chrom请求动态网页
    def






```

































































