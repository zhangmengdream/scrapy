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

   ```python
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


正是http提供的这些方法，我们才实现了restful 提供的这些接口，elasticasearch是基于restful这个接口来完成的
用GET方法可以获取某个document是可以获取的，但是查询数量过多的时候，效率是比较低的
因为每次建立一个HTTP的连接，让他给我们返回数据，都是需要建立三次握手的（开销比较大，效率比较低）
```

#### 倒排索引

```python
概念：倒排索引源于实际应用中需要根据属性的值来查找记录，这种索引表中的每一项，都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性来确定记录的位置，因而成为倒排索引（inverted index）。带有倒排索引的文件我们称为倒排索引文件，简称倒排文件（inverted file）
```

------

例：

假设有A、B、C三个文件，这三个文件可以当做elasticseacher的文档（这里只是举例子，实际文件的内容不止这么点）

文件A：通过python django搭建网站

文件B：python scrapy爬取网站数据

文件C：scrapy-redis 分布式爬虫

需求：查询有哪些文件包含python这个关键词 

------

解：
正常的思维方式，就是对文档进行遍历,需要遍历文件中的所有的内容，才能确定有没有这个关键词----这种效率是很低的，当文件上亿的时候，每一个关键词都要遍历上亿个文件
	
所以使用倒排，即.当，某个（比如A）文件存储之前，对A的内容全部遍历，遍历完之后进行分析 这里的分析也就是分词，分词都会进行存储，所以倒排索引的结构就是   
													 			

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

- 倒排索引和elasticseracher打分如何打的 -- 需要了解TF-IDF 概念
- TF-IDF 概念是elasticseracher进行打分的很关键的技术

------

目前的搜索引擎中，一般的底层的索引存储，都采用的是倒排索引

倒排索引也是搜索引擎区别于我们已有的关系型数据库或者其余的nosql数据库的核心

------

倒排索引待解决问题（这些问题elasticseacher都已经帮我们完成了）

1. 大小写转换问题，比如python和PYTHON是否应该作为同一个词
2. 词干抽取，looking和look是否可以通过词干抽取为同一个词
3. 分词，如 屏蔽系统  是应该分为 ‘屏蔽’、‘系统’ 还是 ‘屏蔽系统’
4. 倒排索引如果文件过大，如何进行压缩编码

#### elasticseacher索引的操作

（操作是基于kibana操作的 -->Dev Tools --> Console）

es的文档和索引的CRUD操作（CRUD增删改查）

一、索引的初始化

索引可以理解为关系数据库中的数据库

增：（完成操作后在elasticseacher-head的插件里面就可以发现拉钩的索引了）

```python
PUT lagou
{
    "settings":{
        "index":{
            "number_of_shards":5,
            "number_of_replicas":1
        }
    }
}
-------------------------------------------------------------------------
PUT lagou : 用的HTTP的PUT方法，lagou为索引的名称
settings ：
index:  number_of_shards 指创建索引分片的数量（默认分片的数量是5个，不写的话也是为5）
    	number_of_replicas  指创建索引副本的数量（默认副本的数量1个，不写的话也是为1）
-------------------------------------------------------------------------        
        
运行完之后，表示创建好了索引：
{
    "acknowledged":true,
    "shards_acknowledged":true
}

以上完成之后，在head的插件里面就可以发现拉钩的索引了
一旦创建好了索引，shards（分片）的数量是不可以修改的
replicas(副本)的数量是可以修改的

往索引里面放一个文档：
保存文档：
（保存文档的时候，必须要指明一个type，即使没有事先建立）index--lagou，type--job，id--1
 不指明id的时候，会自动生成一个id

PUT lagou/job/1
{
    "title":"python分布式爬虫开发"，
    "salary_min"":15000,
    "city":"北京"，
    "company":{
        "name":"百度"，
        "company_addr":"北京市软件园"
    }，
    "publish_date":"2017-4-16",
    "comments":15
}
```

查：

```python
获取settings的信息：
GET lagou/_settings  运行即可获取settings的信息

GET _all/_settings   获取所有索引的settings

GET .kibana,lagou/_settings   获取某一些索引的settings的信息(注意kibana,lagou之间不可以有空格)

GET _settings		获取所有索引的settings

获取索引信息
GET _all			获取所有的索引

GET lagou			获取拉钩的所有信息

GET lagou/job/1      文档的获取信息

GET lagou/job/1?_source=title，city     获取某些字段的信息，这里的source是关键词，值获取指定字段的值

GET lagou/job/1?_source		     获取所有的信息
GET lagou/job/1?_source=title    只获取title的信息
```

改：

```python
更新拉钩副本的数量为2（切片数量不能修改，运行修改的话会报错）
PUT lagou/_settings
{
  "number_of_replicas":2   
}

修改文章 ：修改需要指明id，修改的方式是一种覆盖的方式（完全覆盖），
比如这里，和原来部分相比，去掉了city值，运行结果也没有了city
PUT lagou/job/1
{
 	"title":"python分布式爬虫开发"，
    "salary_min"":15000,
    "company":{
        "name":"百度"，
        "company_addr":"北京市软件园"
    }，
    "publish_date":"2017-4-16",
    "comments":15
}

post方法改值:(这里的doc指明我要修改哪些字段)
    
POST lagou/job.1/_update
{
    "doc":{
       "comments":20     
    }
}

修改有两种方式，1：覆盖的方式。2：指明修改哪些字段。 推荐的是第二种方式
```



删：

```python
DELETE lagou/job/1      删除id
DELETE lagou/job        删除文档
DELETE lagou	        删除索引

elasticseacher 5 不支持删除文档的操作了
```

#### elasticseacher的批量操作

批量获取

```python
用GET方法可以获取某个document是可以获取的，但是查询数量过多的时候，效率是比较低的
因为每次建立一个HTTP的连接，让他给我们返回数据，都是需要建立三次握手的（开销比较大，效率比较低）elasticseacher提供了mget，可以一次性查询多条命令

1、传递两个get的请求体查询，两个index可以不一致

GET _mget
{
    "docs":[
        {"_index":"testdb",
         "_type":"job1",
         "_id":1
        },
        {"_index":"testdb",
         "_type":"job2",
         "_id":2
        }
    ]
}



2、查询同一个index下面的不同的type的方式   testdb为一个index
GET testdb/_mget
{
    "docs":[
        {"_type":"job1",
         "_id":1
        },
        {"_type":"job2",
         "_id":2
        }
    ]
}

3、查询同一个type里面不同的id
一、
GET testdb/job1/_mget
{
    "docs":[
        {"_id":1
        },
        {"_id":2
        }
    ]
}
二、(简写方式)
GET testdb/job1/_mget
{
    "ids":[1,2]
}
```



批量的bulk操作(可以执行增、删、改的操作)

> 批量导入可以合并多个操作，比如index，delete，update，create等。
>
> 也可以帮助从一个索引导入到另一个索引。

标准格式：

```python
{"index":{"index":"test","type":"type1","_id":"1"}}
{"field1":"value1"}

{"delete":{"_index":"test","_type":"type1","_id":"2"}}

{"create":{"_index":"test","_type":"type1","_id":"3"}}
{"field1":"value2"}

{"update":{"_id":"1","_type":"type1","_index":"index1"}}
{"doc":{"field2":"value2"}}s
```

例：

```python
POST _bulk
{"index":{"index":"lagou","type":"job","_id":"1"}}
{"title":"python分布式爬虫开发","salary_min":15000,"city":"北京","company":{"name":"百度","company_addr":"北京市软件园"},"publish_date":"2017-4-16","comments":15}

执行就可以插入了
```

注意：只能做成一行，不能换行



#### es映射

 概念：创建索引的时候，可以预先定义字段的类型，以及相关属性

 Elasticseacher会根据JSON源数据的基础类型，猜测你想要的字段映射。将输入的数据变成可搜索的索引项。Mapping就是我们自己定义的字段的数据类型，同时告诉elasticsearch如何索引数据以及是否可以被搜索



作用：会让索引建立的更加细致完善

类型：静态映射，动态映射

string类型：text，keyword（string类型在es5已经开始废弃）

数字类型：long，integer，short，byte，double，float

日期类型：date

bool类型：boolean（true，false，yes，no）

binary类型：binary(二进制,二进制的数据是不会被检索的)

复杂类型：object，nested

geo类型（地理位置）：geo-point（通过经纬度标识）,geo-shape(通过多个点标识)

专业类型:ip（ip地址），competion（做搜索建议的）

```python
复杂类型：object，nested

PUT lagou/job/1
{
 	"title":"python分布式爬虫开发"，
    "salary_min"":15000,
    "company":{
        "name":"百度"，
        "company_addr":"北京市软件园"
    }，
    "publish_date":"2017-4-16",
    "comments":15
}

object：company就是object(一个对象)

    "company":{
        "name":"百度"，
        "company_addr":"北京市软件园"
    }

nested：(把对象放到数组里面就是一个nested)

    "company":{
        "name":"百度"，
        "company_addr":"北京市软件园"
        "emplyee":[{
            {
                "name":"百度"，
                "age":20
            },
             {
                "name":"谷歌"，
                "age":18
            }
        }]
    }，
```





es里面一旦设置了某个列的类型，那么这个类型就不可以修改了，如果非要修改，需要将索引删除，修改之后重新导入



#### elasticsearch的简单搜索（查询）

elasticsearch是功能非常强大的搜索银枪，使用它的目的就是为了快速的查询到想要的数据

查询分类：

基本查询：使用elasticsearch内置查询条件进行查询

组合查询：把多个查询组合在一起进行复合查询

过滤：查询同时，通过filter条件在不影响打分的情况下筛选数据（只起到筛选的作用，并不参与打分）



#### 基本查询：



match查询:查询的时候会将闯入的字符串进行分词，会使用title

```py
GET 
```

term查询：传递进来的值，不会做任何处理（不会分词，会做全量查询）



term查询和match的区别：term是不会对查询的输入条件做解析的，match虽然会做解析，但是对keyword类型，也是不会分析的



terms查询：（可以传递一个数组，只要有一个词，都会返回结果）

```python
GET lagou/_search
{
    "query":{
        "terms":{
            "title":{"工程师","django","系统"}
        }
    }
}

"title":{"工程师","django","系统"}  
里面的三个词，只要有一个词，都会返回回来
```

控制查询的返回数量：(可以做分页)

```python
GET lagou/_search
{
    "query":{
        "match":{
            "title":"python"
        }
    },
    "from":1,
    "size":2
}

from:指从第几个开始  0代表从第一条开始
size:表示取几个
```

match_all  查询

```python
GET lagou/_search
{
    "query":{
        "match_all":{}
    }
}

会将所有数据全部返回回来
```

match_phrase 查询

短语查询:会将输入的查询条件分词，分成一个词条，词的数组 （必须满足里面的所有词才会返回结果）

```python
GET lagou/_search
{
    "query":{
        "match_phrase":{
            "title":{
                "query":"python系统",
                "slop":6
            }
        }
    }
}

slop:6  指  python  系统 两个词之间的最小距离，
```

multi_match 查询

比如可以指定多个字段，查询title和desc这两个字段里包含python的关键词文档

```python
GET lagou/_search
{
    "query":{
        "multi_match":{
            "query":"python",
            "fields":["title^3","desc"]
        }
    }
}


multi_match 指我们可以指定多个字段
查询title和desc这两个字段里任何一个包含python，都会返回一个结果

"title^3"：代表设置权重，title里面出现的权重是desc里面出现权重的3倍（会影响排序）

```

```python
指定返回的字段

GET lagou/_search
{
    "stored_field":["title","company_name"],
    "query":{
        "match":{
            "title","python"
        }
    }
}

#stored_field 指明返回哪些字段

```

通过sort把结果排序

```python
GET lagou/_search
{
    "query":{
        "match_all":{}
    },
    "sort":[{
        "comments":{
            "order":"desc"
        }
    }]
}

先做query查询，这里代表对comment进行sort，desc代表降序  asc代表升序
```

range范围查询

```python
GET lagou/_search
{
    "query":{
        "range":{
            "comments":{
                "gte":10,
                "lte":20,
                "boost":2.0
            }
        }
    }
}

大于等于10，小于等于20，
"boost":2.0代表：权重
```

对时间做range

```python
GET lagou/_search
{
    "query":{
        "range":{
            "add_time":{
                "gte":"2017-04-01",
                "lte":"now"
            }
        }
    }
}

now:代表当前时间
```

wildcard查询：模糊查询

```python
GET lagou/_search
{
    "query":{
        "wildcard":{
            "title":{
                "value":"pyth*n","boost":2.0
            }
        }
    }
}

*代表通配符

```

#### 组合查询

bool查询

es5之后，老版本的filtered已经被bool替换
bool包括 must  should  must_not  filter来完成

```python
格式如下：

bool:{
    "filter":[],
    "must":[],
    "should":[],
    "must_not":[],
}

查询条件只有一个的时候，可以用{}字典来表示，多个的时候用[]
```

建立测试数据（用bulk操作添加大量数据）

```python
POST lagou/testjob/_bulk
{"index":{"_id":1}}
{"salary":10,"title":"Python"}
{"index":{"_id":2}}
{"salary":20,"title":"Scrapy"}
{"index":{"_id":3}}
{"salary":30,"title":"Django"}
{"index":{"_id":4}}
{"salary":30,"title":"Elasticsearch"}
```

简单的过滤查询

```python
查询薪资为20k的工作

GET lagou/testjob/_search
{
    "query":{
        "bool":{
            "must":{
                "match_all":{}
            },
            "filter":{
                "term":{
                    "salary":20
                }
            }
        }
    }
}

对应的sql语句
select * from testjob where salary=20

int类型是不会分词的

```

指定多个值

```python
GET lagou/testjob/_search
{
    "query":{
        "bool":{
            "must":{
                "match_all":{}
            },
            "filter":{
                "terms":{
                    "salary":[10,20]
                }
            }
        }
    }
}
```



```python
在倒排索引时，title是string类型的字符串，所以在做数据的索引的时候会先做分词，大写字母会转换成小写的

term会原样查询，所以查询的时候就会查询不到了
match可以查到

```

查看分析器解析的结果（运行查看结果）

```python
GET _analyze
{
    "analyzer":"ik_max_word",
    "text":"pyrhon网络开发工程师"
}

#ik_max_word 和 ik_smart 都是分析器ik_smart是以最小的分词分出来
（可在github上查看）
```

bool过滤查询，可以做组合过滤查询

```python
GET lagou/testjob/_search
{
    "query":{
        "bool":{
            "should":[
                {"term":{"salary":20}},
                {"term":{"title":"python"}}
            ],
            "must_not":{
                "term":{"price":30}
            }
        }
    }
}

对应的sql语句
select * from testjob where (salary=20 OR title=Python) AND (salary!=30)

多个term查询的时候用数组，单个用字典
```

嵌套查询(bool里面嵌套bool)

```python
GET lagou/testjob/_search
{
    "query":{
        "bool"{
            "should"[
                {"term":{"title":"python"}},
                {"bool":{
                    "must":[
                        {"trem":{"title":"django"}},
                        {"trem":{"salary":30}}
                    ]
                }}]
        }
    }
}

对应的sql语句
select * from testjob where title="python" or (title="django" AND salary=30)
```

过滤空和非空

建立测试数据

```python
POST lagou/testjob2/_bulk
{"index":{"_id":"1"}}
{"tags":["search"]}
{"index":{"_id":"2"}}
{"tags":["search","python"]}
{"index":{"_id":"3"}}
{"other_field":["some data"]}
{"index":{"_id":"4"}}
{"tags":null}
{"index":{"_id":"5"}}
{"tags":["search",null]}
```

处理null空值的方法

```python
GET lagou/testjob2/_search
{
    "query":{
        "bool":{
            "filter":{
                "exists":{
                    "field":"tags"
                }   
            }
        }
    }
}


对应的sql语句
select tags from testjob2 where tags is not NULL
```

查询不存在的

```python
就是上面的值取反

GET lagou/testjob2/_search
{
    "query":{
        "bool":{
            "must_not":{
                "exists":{
                    "field":"tags"
                }   
            }
        }
    }
}

为null的值，和没有这个字段的值，都会返回出来
```

### 如何通过scrapy将数据插入到elasticsearch中

新建一个pipeline将数据写入到elasticsearch中

```python
pipeline.py

class ElasticseacherPipeline(object):
    #将数据写入到ｅｓ中
    def process_item(self,item,spider):
        # 将ｉｔｅｍ转换为ｅｓ的数据
        # article = ArticalType()
        # article.title = item['title']
        # article.url = item['url']
        # article.score = item['score']
        # article.statuss = item['statuss']
        # article.themes = item['themes']
        # article.war = item['war']
        # article.image = item['image']
        # article.image_path = item['image_path']
        # article.authorss = item['authorss']
        # article.brief = item['brief']
        # 
        # article.save()
        item.save_to_es()
        return item
  


items.py


class ManhuaItem(scrapy.Item):

    def save_to_es(self):
        article = ArticalType()
        article.title = self['title']
        article.url = self['url']
        article.score = self['score']
        article.statuss = self['statuss']
        article.themes = self['themes']
        article.war = self['war']
        article.image = self['image']
        article.image_path = self['image_path']
        article.authorss = self['authorss']
        article.brief = self['brief']

        article.save()
        return
    
在setting中配置pipeline
```



#### 通过jango和elasticsearch打造搜索引擎网站

一.智能提示(elasticsearch提供智能提示)

```python

```











