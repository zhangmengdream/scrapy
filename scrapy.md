elasticsearch 是最流行的分布式开源搜索引擎，被大量的应用在很多大公司中



spider

source == activate.bat



# navcate

navcate链接不上时，也就是本机ip可以链接，但是其他的不能连接时，需要设置权限

在mysql里面设置权限

mysql提供的一个权限设置的命令

```python
#权限的赋值命令

GRANT ALL PRIVIEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;吗
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



![52696162931](C:\Users\dream\AppData\Local\Temp\1526961629318.png)

爬虫去重策略

将url经过md5保存到set中，



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

#### spider部分源码

```python
spider源码里面会调用start_requests先遍历start_url，遍历完之后，调用make_request_from_url,返回request，然后调用下载器中间件的process_request，下载完成之后,这样就会调用我们下面的parse函数了。
```

##### windows下第一次运行spider的时候，可能会出现没有win32api安装包的情况

```py 
pip install -i https://pypi.douban.com/simple/ pypiwin32  即可
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

#### 字符串函数

```python
获取的字段中 如果有不想要的字符可以用replace  来将字符中不想要的部分替换成空格 
		   如果有不想要的空格，用strip() 去出空格
		   如果有不想要的字段，可以split() 分割字段取出需要的部分
```

### xpath里面的函数

```python
contains   内置函数   
第一个参数表示取什么的属性值

//span[contains(@class,"vote-piost-up")]
这个表示，我需要找一个span ，这个span的class包含了vote-piost-up这个字符串
```

```python
#需要取封面图的时候，也就是主页面，点击连接时的连接图片也需要传递到数据中的时候，需要用meta，这样就会传递到子页面的response中了
for post_node in post_nodes:
    image_url = post_node.css("img::attr(src)").extract_first("")
    post_url = post_node.css("::attr(href)").extract_first("")
	yield Request(url=parse.urljoin(response.url,post_url),meta={"front_image_url":image_url}, callback=self.parse_detail)

  
下面取字段时：    
front_image_url = response.meta.get('front_image_url','') # 后面的空代表默认值
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

Ubuntu 下的安装
sudo apt-get install libmysqlclient-devsimp
```

#### 将图片保存在本地中

```python
# settings里面设置
ITEM_PIPELINES = {
'scrapy.pipelines.images.ImagesPipeline':1,
}
后面的数字越小就会越早处理这个pipeline

pipeline怎么知道去item中取哪个字段，所以这里需要配置
IMAGES_URLS_FIELD = "字段名"
配置完之后，image回去item里面去找这个字段进行下载，下载图片需要设置图片的保存路径
设置路径需要写一个绝对路径，所以需要用os设置images的路径位置(设置图片下载的路径)

import os
project_dir = os.path.abspath(os.path.dirname(__file__))
IMAGES_STORE = os.path.join(project_dir,'images')
#设置过滤掉一些图片(下面的设置表示下载的图片必须是尺寸大于100*100的)
IMAGES_MIN_HEIGHT = 100
IMAGES_MIN_HEIGHT = 100

#字段名 是我需要取数据的字段名



如果图片的路径如果想要显示出来，用item展示出来
图片已经保存到了本地，怎么能够把本地的图片路径和item记录绑定起来，放到front_image_path 里面

方法: 定义自己的pipeline继承ImagesPipeline，让自己的某些功能能够定制
    ImagesPipeline里面有很多函数可以重载
    
from scrapy.pipelines.images import ImagesPipeline  
class ArticalImagePipeline(ImagesPipeline)：

# 获取图片在本地下载的路径   需要重载ImagesPipeline里面的item_completes这个函数
	def item_completes(self,results,item,info):
        #results里面有我们想要的数据
		for ok, value in results:
            image_file_path = value["path"]
        item["image_path"] = image_file_path
#此时item里面就有了我们获取的图片路径的值，然后将这个item返回，因为pipeline设置的执行顺序是先执行自定义的这个pipeline，然后执行ManhuaspiderPipeline ，所以会把我怕们设置的路径的值带过去
        return item  

** 重要函数
get_media_requests()   传入的必须是列表或者可迭代的对象 获取了url之后，把这个url凑成一个request交给scrapy下载器进行下载
item_completes()
```

#### 插入本地json文件中的两种方法（除了文件名需要修改，其他都是固定格式）

```python
# 方法一：   自定义ｊｓｏｎ文件的导出
class JsonExporterPipeline(object):
    # 调用ｓｃｒａｐｙ提供的ｊｓｏｎ　ｅｘｐｏｒｔ　导出ｊｓｏｎ文件
    def __init__(self):
        self.file = open('articalexport.json','wb')
        self.exporter = JsonItemExporter(self.file,encoding='utf-8',ensure_ascii=False)
        self.exporter.start_exporting()

    def process_item(self,item,spider):
        self.exporter.export_item(item)
        return item

    def close_spider(self,spider):
        # 停止导出
        self.exporter.finish_exporting()
        self.file.close()

# 方法二：
class JsonWithEncodingPipeline(object):
    # 自定义json文件的导出
    def __init__(self):
        self.file = codecs.open('manhua.json','w',encoding='utf-8')

    def process_item(self,item,spider):
        lines = json.dumps(dict(item),ensure_ascii=False)+'\n'
        self.file.write(lines)
        return item
    def spider_closed(self,spider):
        self.file.close()


```

#### 插入mysql数据库的两种方法

```python
方法一：（同步的操作）
class MysqlPipeline(object):
    def __init__(self):
        self.conn = MySQLdb.connect('127.0.0.1','root','test1999','manhua',charset='utf8',use_unicode=True)
        self.cursor = self.conn.cursor()

    def process_item(self,item,spider):
        insert_sql = """
            insert into manhua_info(title, score, statuss, themes, war, image, authorss, brief, image_path, url) 
            VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)
        """
        self.cursor.execute(insert_sql, (item["title"], item["score"], item["statuss"], item["themes"], item["war"], item["image"][0], item["authorss"], item["brief"], item["image_path"], item["url"]))
        self.conn.commit()

方法二： （异步的操作）
# spider的解析速度肯定是超过数据库的入库速度，如果后期爬取的url越来越多，入库速度回跟不上解析速度，这样的话就会堵塞   tw isted给我们提供了mysql插入异步化的操作
需要先在 setting里面配置
MYSQL_HOST = '127.0.0.1'
MYSQL_DBNAME = 'manhua'
MYSQL_USER = 'root'
MYSQL_PASSWORD = 'test1999'

from twisted.enterprise impore adbapi
#adbapi 会将mysql的操作变成异步化的操作
class MysqlTwistedPipeline(object):
    def __init__(self,dbpool):
        self.dbpool = dbpool
    @classmethod
    def from_settings(cls,settings):
        # 这个方法的参数就是settings  会读取settings里面的值
        dbparms = dict(
            host = settings['MYSQL_HOST'],
            db = settings['MYSQL_DBNAME'],
            user = settings['MYSQL_USER'],
            passwd = settings['MYSQL_PASSWORD'],
            charset = 'utf8',
            cursorclass = MySQLdb.cursor.DictCursor,
            use_unicode = True,
		)
        dbpool = adbapi.ConnectionPool("MySQLdb",**dbparms) 
        return cls(dbpool)
        # 这里返回也就是一个实例化的pipeline（实例化的对象）
        
    def process_item(self,item,spider):
        #使用twisted 将mysql插入变成异步执行
        query = self.dbpool.runInteraction(self.do_insert,item)
        #处理异常
        query.addErrback(self.handle_error,item,spider)
        
    def handle_error(self,failure,item,spider):
        #处理异步插入的异常
        print(failure)
    def do_insert(self,cursor,item):
        #执行具体的插入
                insert_sql = """
            insert into manhua_info(title, score, statuss, themes, war, image, authorss, brief, image_path, url) 
            VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)
        """
        #这里的cursor用传递进来的参数cursor就可以了， 也不需要commit 因为会自动帮我们保存 
        cursor.execute(insert_sql, (item["title"], item["score"], item["statuss"], item["themes"], item["war"], item["image"][0], item["authorss"], item["brief"], item["image_path"], item["url"]))

```



## Itemloader

scrapy的itemloader机制，可以让我们的维护工作变得更加简单

itemloader提供的是一个容器，可以配置item提供的某一个字段需要那种规则来解析

```markdown
from scraoy.loader import ItemLoader
#通过itemloader加载item
itemloader = ItemLoader(item=ManhuaItem(),response=response)
itemloader.add_xpath('title','//div[@class="banner_detail_form"]/div[2]/p[1]/text()')
itemloader.add_value('url',response.url)
itemloader.add_xpath('score','//div[@class="banner_detail_form"]/div[2]/p[1]/span/span[1]/text()')
itemloader.add_xpath('statuss','//div[@class="banner_detail_form"]/div[2]/p[3]/span[1]/span/text()')
itemloader.add_xpath('themes','//div[@class="banner_detail_form"]/div[2]/p[3]/span[2]/a/span/text()')
itemloader.add_xpath('war','//div[@class="banner_detail_form"]/div[2]/p[3]/span[3]/text()')
itemloader.add_xpath('image','//div[@class="banner_detail_form"]/div/img/@src')
itemloader.add_xpath('authorss','//div[@class="banner_detail_form"]/div[2]/p[2]/a/text()')
itemloader.add_xpath('brief','//div[@class="banner_detail_form"]/div[2]/p[4]/text()')
# 调用这个方法，才会将以上的规则进行解析
manhua_item = itemloader.load_item()
yield manhua_item

# 有三种比较重要的方法
itemloader.add_css()
itemloader.add_xpath()
itemloader.add_value()

**通过以上过程debug可以发现有两个问题
一、有的字段需要处理函数
这时候我们就需要在items里面定义字段的时候进行修改了

from scrapy.loader.processors import MapCompose,TakeFirst

MapCompose 里面可以连续调用多个函数 ，做多次处理
这里面的Field里面实际上是有两个参数可以自定义的
eg: 
title = scrapy.Field(
   input_processor =  MapCompose()
   )
   input_processor表示当item里面的值传递进来之后，可以对这个item里面的值进行预处理

def add_job(value):
  return value+'-add_job'
  
class ManhuaItem(scrapy.Item):
   title = scrapy.Field(
      input_processor =  MapCompose(add_job)
   )
   url = scrapy.Field()
   score = scrapy.Field()
   statuss = scrapy.Field()
   themes = scrapy.Field()
   war = scrapy.Field()
   image = scrapy.Field()
   image_path = scrapy.Field()
   authorss = scrapy.Field()
   brief = scrapy.Field()

** 二、获取的值都是列表

方法（1）在字段内加函数
from scrapy.loader.processor TakeFirst
class ManhuaItem(scrapy.Item):
   title = scrapy.Field(
      input_processor =  MapCompose(add_job)
      output_processor = TakeFirst()
   )

方法（2）自定义一个itemloader，来取第一个值 ------（重载ItemLoader这个类） --- 推荐
from scrapy,loader import ItemLoader
class ManhuaItemLoader(ItemLoader)
	default_output_processor = TakeFirst()
	
这样在spider页面就用manhuaItemLoader 代替 ItemLoader 来使用
```





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

##### 设置不加载图片有chromedriver （用里面的prefs参数） --- 好处节省时间，加快加载速度 ---  chromedriver里面还有很多设置，自己研究

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
browser = webdriver.PhantomJS(executable_path="")
browser.get('')
prine(browser.page_source)
browser.quit()
```

linux等无界面的系统个中phantomjs的好处就可以体现出来了，，

phantomjs在多进程的情况下渲染是及其不稳定的，所以不推荐

windows下更加推荐chrome，chrome在无界面的情况下也可以使用



### 如何将selenuim集成到script当中

##### 一、设置一个中间件

```python
同步方法：
from selenium import webdriver
from scrapy.http import HtmlResponse
class JSPageMiddleware(object):
    # 通过chrome请求动态网页
    def process_request(self,request,spider):
        if spider == "jobbole":
			brower = webdriver.Chrome(execytable_path='D:/Temp.chromdriver.exe')
			brower.get(request.url)
			import time 
             time.sleep(3)
			return HtmlResponse(url= browser.current_url,body=browser.page_source,encoding='utf-8',request=request)
        
只需要给我们返回的page_source 做一个初始化就可以了 
一遇到HtmlResponse，scrapy就不会给我们的downloader发送了，而是直接返回给我们的spider
记得在setting中启动中间件

--------------------------------------------------------------------------
            
#  每次请求的时候都会打开一个chrome这样会非常慢
解决方法： 初始化的时候定义一个属于类的 browser,之后每次请求的时候用self.browser来请求页面, 这样就可以通过一个browser来请求多个页面，(但是浏览器不会自动关闭，解决方法下面信号部分，添加一个信号)

class JSPageMiddleware(object):
	def __init__（self）:
        self.brower = webdriver.Chrome(execytable_path='D:/Temp.chromdriver.exe')         super(JSPageMiddleware,self).__init__()

    def process_request(self,request,spider):
        if spider.name == "jobbole":
			self.brower.get(request.url)
			import time 
             time.sleep(3)
             print("访问:{0}".format(request.url))
			return HtmlResponse(url= self.browser.current_url,body=self.browser.page_source,encoding='utf-8',request=request)   
          
---------------------------------------------------------------------    
            
既然不是每个页面都需要chrome ，把chrome放入spider里面
当启动多个spider的时候，就会有多个chrome，这样对我们的并发也是有好处的 
所以解决方法是将__init__  放入spider中来做

jobbole.py页面：

from selenium import webdriver
class JobboleSpider(scrapy.spider):
    name = 'jobbole'
      ......
        
	def __init__（self）:
        self.brower = webdriver.Chrome(execytable_path='D:/Temp.chromdriver.exe')         super(JobboleSpider,self).__init__()


middlewares.py页面  (这里的browser调用的方法就是  spider.browser )

class JSPageMiddleware(object):
    def process_request(self,request,spider):
        if spider.name == "jobbole":
			spider.brower.get(request.url)
			import time 
             time.sleep(3)
             print("访问:{0}".format(request.url))
			return HtmlResponse(url= spider.browser.current_url,body=spider.browser.page_source,encoding='utf-8',request=request)  

    
scrapy的信号：scrapy允许我们在__init__ 或者在任何地方都可以加入一个信号量
jobbole.py页面：
from scrapy.xlib.pydispatch import dispatcher
from scrapy import signals
from selenium import webdriver
class JobboleSpider(scrapy.spider):
    name = 'jobbole'
      ......
        
	def __init__（self）:
        self.brower = webdriver.Chrome(execytable_path='D:/Temp.chromdriver.exe')         super(JobboleSpider,self).__init__()
        dispatcher.connect(self.spider_closed，signals.spider_closed)
    def spider_closed(self,spider):
        #当爬虫退出的时候，关闭chrome
        print("spider closed")
        self.browser.quit()
        
做一个信号的映射，当spider close的时候，我们需要做一些什么事情
第一个参数为处理函数即 -- 当某个信号发生的时候，我们用什么函数来处理
第二个参数为发送什么信号量 -- 系统提供了很多信号
							 							（下面会详解信号）
----------------------------------------------------------------------------

异步方法： 需要自己重写一个dowmloader

在github中搜索
scrapy downloader
搜索结果的第一个
flisky/scrapy-phantomjs-downloader

```

###    

#### python提供的无界面环境，可以在

## 无界面的环境下运行chrome

```python
第一步：pip install pyvirtualdisplay

from pyvirtualdisplay import Display
# visible=0 不显示的意思 ,size=(800,600)设置大小
display = Display(visible=0,size=(800,600))
display.start()
browser = webdriver.Chrome()
browser.get()

运行遇上程序的时候，可能会出现错误，需要执行下面的步骤
sudo apt-get install xvfb
pip install xvfbwrapper
```



### 额外的还有的解决方案

```python
scrapy本身也给我们提供了一种解决方案，专门用来针对动态网站的
叫做 scrapy splish
相当于自己运行的一个server，通过http的请求方式，将我们的js拿去执行，性能比phantomjs
chrome都要高，因为他是一个轻量级的，
scrapy splish 好处是支持分布式的，因为他是运行在一个server是，可以针对这个server发起请求，但是稳定性没有chrome高
### 但是更推荐chrome，稳定性比较高
----------------------------------------------------------------------------
selenium grid

与scrapy splish类似，也是支持分布式的，也是启动一个服务，通过api的方式，向网站发起请求
----------------------------------------------------------------------------
splinter
splinter 也是一种操控浏览器的解决方案，用法与selenium很像。
```



### scrapy爬虫的暂停与重启

```markdown

一个爬取到中间之后想要继续爬取， 继续执行与之前相同的命令

** 启动的时候执行命令
crawl spider lagou -s JOBDIR=job_info/001  

要想做到暂停爬虫，就要保存很多中间状态
比如没有做完的requests
比如没有执行完的filter
必须把这些状态都保存下来，才能做到重启与暂停
也包括当前spider的状态，这些状态都必须有保存
这些东西scrapy都已经帮我们做好了，我们需要做的就是提供一个目录，spider的信息会放到这个目录下面来

** scrapy的暂停与重启，在官方文档中明确指明了，不同的spider不能共用同一个目录
同一个spider在不同的run的时候，也不能用同一个目录
（因为在次run会继续上一次的接着爬取，如果想换一个从新爬取就得换一个目录）

scrapy接收结束爬虫的信号是ctrl+c 的命令

linux下的信号 
kill -f main.py
kill -f -9 main.py  强制杀掉，如果用这个命令，无法接收到中断信号，在main文件里就做不到善后的工作

所以需要crawl spider lagou -s JOBDIR=job_info/001  这个命令来运行，这样才能接收到ctrl+c 的信号

--------------------------------------------------------------------------
也可以在setting中设置
JOBBDIR = "job_info/001"
这样设置运行main文件的时候，也会将当前的信息，保存指定在目录之下

--------------------------------------------------------------------------
在spider页面设置
custom_settings = {
	JOBBDIR = "job_info/001"
}

--------------------------------------------------------------------------
建议尽量在黑屏终端使用

一个ctrl+c  会保存状态
两个ctrl+c 会强制终止

运行完会出现三个文件
requests.seen
   保存已经访问过得url
spider.state
   状态信息
requests.queue
	active.json
	p0 --- 需要继续做完的request（当继续跑，跑完的时候p0是会被清除掉的）
	
重新启动 crawl spider lagou -s JOBDIR=job_info/001  
   继续执行与之前相同的命令
```



### scrapy的url去重原理   ？

```python




```





##  telnet

```python
让我们可以连接到远程的端口进行操作

本身是不在localhost上监听的  可以设置远程监听

在源码扩展部分
```



## spidermiddleware

```markdown
# 位于spider和engin之间的

SPIDER_MIDDLEWARES = {
   'manhuaspider.middlewares.ManhuaspiderSpiderMiddleware': 543,
}

默认的ManhuaspiderSpiderMiddleware  
下面简述里面的比较重要的几个函数

 > from_crawler 
 
是会被middle_ware的manage调用的
（ manage管控着我们的spidermiddleware 和 downloadermiddleware
只要写middleware都可以重载from_crawler这个函数 ）
调用这个函数时会发送一个信号， 他有一个处理函数spider_opened 
通过代码可以看出这个函数只是记录了一个日志

** 下面四个函数是spiddermiddleware里面可以重载的四个函数：

> process_spider_input(response,spider):

engin拿到response之后，会发送给spiders 可以在这里加上处理逻辑
    
> process_spider_output(response,result,spider):

spiders解析到request之后，会发送给engin时 可以在这里加上处理逻辑
这个函数里面的逻辑就是有了result之后，yield出去交给engin

> process_spider_exception(response,exception,spider):

当我们call了一个spider，或者调用了这个process_spider_input这个函数raise了exception，就会进入这个函数，可以在这里加入自己想要的逻辑

> process_start_requests(start_requests,spider):

当我们开启了一个start_request的时候就会被调用，与process_spider_output函数类似，不同的是这个函数只能return requests不能return items

```

### spidermiddleware的源码分析

```python
一、depth       在process_spider_output的时候处理
可以监控我们可以爬到多少层，可以在这里设置我们可以爬多少层

会从这部分开始执行：

	if self.stats and 'depth' not in response.meta:
		response.meta['depth'] = 0
   		if self.verbose_stats:
			self.stats.inc_value('request_depth_count/0', spider=spider)
	return (r for r in result or () if _filter(r))

解析：
stats 是指状态收集变量，会记录spider很多中间状态
会判断depth有没有在meta里面 如果没有设置depth为0，如果有会把这个值加1
处理完之后会return，调用_filter方法，这个函数里面会有爬取最大深度的逻辑控制


二、httperror   在process_spider_input的时候处理
帮我们过滤掉一些返回状态不为200的response
当我们yield request的时候，设置一些参数可以使用
例如：
1、
yield Request(url=parse.urljoin(response.url, post_url), meta={'handle_httpstatus_all':True}, callback=self.parse)
 
meta={'handle_httpstatus_all':True}设置了这个之后，当他进入httperror的时候
这个meta会在response，有了这个值，他就不会对我们的状态码进行过滤了，所以即使是404也会帮我们返回过去
2、
yield Request(url=parse.urljoin(response.url, post_url), meta={'handle_httpstatus_list':[404,500,301]},callback=self.parse)

 meta={'handle_httpstatus_list':[404,500,301]}设置了这个之后，在httperror里面就会进入allowed_status,下面的判断语句会显示，如果你的状态码在这个里面，就会返回到spider里面让你自己去处理

3、
在jobbole.py文件中设置
handle_httpstatus_list= [404]

def parse(self,response):
    ........
    
这样的效果如同2，不过这样的优先级就不如2了

```

### scrapy中的状态数据收集

```python
scrapy提供了方便的收集数据的机制，数据以key/value方式存储，也就是dict
(数据收集器一般都是数据类型的)

让spider的统计信息统一管理，让我们操作起来非常简单
常见的数据收集器的使用方法：
通过stats属性来使用数据收集器

stats是类的对象（里面放置了很多spider的数据），他的对象有一些函数：
设置数据: 
stats.set_value('hostname',socket.gethostname())
增加数据：  （调用inc_value数据就会加1）
stats.inc_value('pages_crawled')
当新的值比原来的值大的时候，设置为新的数据：
stats.max_value('max_items_scraped',value)
当新的值比原来的值小的时候，设置为新的数据：
stats.min_value('max_items_scraped',value)
获取数据:
stats.get_value('hostname',socket.gethostname())

# 数据收集器：
一、MemoryStatsCollector   （用的最多的）
是放在内存中的

    def __init__(self, crawler):
        self._dump = crawler.settings.getbool('STATS_DUMP')
        self._stats = {}
STATS_DUMP是可以在setting中设置的
                
二、DummyStatsCollector


# 例子： 
需求、收集伯乐在线的所有404url以及404页面数
spider默认情况下只会处理状态为200到300之间的页面
为了将404可以进行统计需要设置

handle_httpstatus_list= [404]

#所有404url以及404页面数 设置保存这两个数据的变量
def __init__(self):
    #用来保存所有404的页面 
    self.fail_urls=[]
    dispatcher.connect(self.handle_spider_closed, signals.spider_closed)
def parse(self,response):
    if response.status == 404:
        self.fail_urls.append(response.url)
        self.crawler.stats.inc_value("failed_url")
	...................
    
self每一个变量都有一个crawler
inc_value 这个函数就是自动＋1了


debug的时候可以看到stats里面有过很多scrapy的中间件、扩展等给我们默认填充的值，这些值就可以帮我们很好地的了解spider的一些状态
```



### scrapy的信号和扩展

信号是中间件和扩展之间的桥梁、、

scrapy的组件和扩展都是基于信号来设计的

scrapy本身内置了很多信号

middleware是extension的一种

```python
信号提供了一些参数，不过处理函数不用接收所有的参数
信号分发机制仅仅提供处理器接收的参数


延迟的信号处理器（Deferred signal handlers）

# 内置信号：
engine_started 当scrapy引擎启动爬取时，发送该信号（改信号可能在spider_opeded之后被发送，取决于spider的启动方式，所有不要依赖改信号会比spider_opened更早被发送）

engine_stopped 当scrapy引擎停止爬取时，发送该信号

item_scraped   当item被爬取，并通过所有的item pipeline后（没有被dropped，即丢弃），发送该信号

item_dropped   当item被爬取，并通过item pipeline时，有pipeline抛出DropItem异常，丢弃item时发送该信号

spider_closed  当spider被关闭的时候，发送该信号，改信号可以用来释放每个spider在spider_opened时占用的资源

spider_opened  当spider开始爬取的时候，会发生这个信号，如果需要记录开始爬取的时间，可以捕捉这个信号，捕捉到这个信号之后，得到了开始爬取的时间们可以放到数据收集器当中

spider_idle   当spider空闲的时候，这个信号就开始发送
			 rsquests正则等待被下载的时候
			 request被调度的时候
			 items正在 itempipeline中被处理的时候
		 当改信号所有处理器被调用后，如果spider仍然保持空闲状态，引擎将会关闭该spider，当spider被关闭后，spider_close信号将会被发送
        
spider_error  当回调函数产生错误的时候，改信号被发送

request_scheduled 当引擎调度一个request对象，用于下载时，该信号被发送

response_received 当引擎从ENGIN downloader获取到一个新的response的时候，发送该信号

response_downloaded 当一个HTTPResponse被下载时，由downloader发送该信号

以上的信号，可以根据自己的需要插入不同的逻辑


.......
示例： 
需求 ：当closed 的时候将所有的404的url拼接成一个字符串
dispatcher.connect(handle_spider_closed,signals.spider_closed)这句话的设置就是当spider_closed信号发出的时候就会进入handle_spider_closed这个逻辑
stats里面是没有列表的

jobbole.py

handle_httpstatus_list = [404]

def __init__(self):
    #用来保存所有404的页面
    self.fail_urls=[]
    dispatcher.connect(self.handle_spider_closed,signals.spider_closed)
    
def handle_spider_closed(self，spider，reason):
    self.crawler.stats.set_value("failed_urls",",".join(self.fail_urls))

def parse(self,response):
    if response.status == 404:
        self.fail_urls.append(response.url)
        self.crawler.stats.inc_value("failed_url")

```

### 如何开发scrapy的扩展

扩展框架提供的一个机制，是的你能够自定义功能绑定到scrapy

扩展只是正常的类，他们在scrapy启动时被实例化，初始化



> scrapy和扩展的区别：  中间件都是缩减版的扩展
>
> 实际上中间件都是扩展 spidermiddleware和downloader和pipeline都有manage 
>
> manage都是归extension来管，所以理论上来讲他们的都是从extension manage出去的
>
> middleware都是功能首先得，middleware里面的函数都是和信号量进行绑定的
>
> 整个extension的机制是通过信号量来处理的





downloader中间件能够重载的几个函数

- process_request(request,spider)
- process_response(request,response,spider)
- process_exception(request,exception,spider)

spider中间件能够重载的几个函数

+ process_spider_input(response,spider)
+ process_spider_output(response,result,spider)
+ process_spider_exception(response,exception,spider)
+ process_start_requests(start_requests,spider)



```python

加载和激活extension
在settings里面设置extension
EXTENSIONS = {
    'scrapy.extensions.corestats.CoreStats':500,
    'scrapy.telnet.TelnetConsole':500,
}

实现自己的扩展
自己的extension会灵活的多，要求我们自己去做singnal（信号的绑定，绑定我们的处理函数）
这些处理函数会在信号被触发的时候去调用


分析scrapy内部提供的一个extension（源码）

throttle.py  是限速的
telnet.py    
corestats.py  记录了spider一些比较重要的统计信息（里面的整个原理都是通过crawler实现的）

memusage.py   系统的信息
通过信号量绑定一些处理函数（逻辑自便）



```









### scrapy redis的分布式爬虫以及源码解析









































#### twisted异步机制

```python
callback=self.parse
这里只是指定了函数名，并没有做调用，因为scrapy的底层是基于Twisted这个框架来完成的，twisted就会根据我们的函数名来自动调用我们的函数
```























































