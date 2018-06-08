spider 可以在运行crawl的时候  通过-a  传递参数

```python
scrapy crawl myspider -a category=electronics
```

spider在构造器中获取参数

```python
class myspider(scrapy.Spider):
    name='myspider'
    def __init__(self,category=None,*args，**kwargs):
        super(MySpider,self).__init__(*args,**kwargs)
        self.start_urls=['http://www.example.com/categories/%s'%category]
        .......
        
```









### 中间突然断开

scrapy自带多线程机制，但是当程序跑到一半时，断掉的话，会不知道哪些页面是爬过的哪些是没有爬过的，这时候有以下几种解决方案

```python
for i in range(totalpages):
    url='***page='+str(i)
    yield Request(url=url,callback=self.next)
```

1. 把所有爬过的url保存到一个文档中，然后再次启动的时候，每次爬取要和文档中的url列表对比，如果相同则不再爬取

2. 在scrapy再次启动的爬取的时候，和数据库里面的数据作对比，如果相同则不存取

3. 利用Request中的优先级（priority）

   当request中加入priority属性的时候，每次scrapy从请求队列中去请求的时候就会判断优先级，先取出优先级高的去访问。

   这样就能保证顺序性，保证顺序性之后，我们要记录已经爬取的页数，由于发生请求下载页面存储数据这几个动作是顺序执行的，也就是说程序发送了这个请求不代表此时已经爬去到这一页了，只有当收到response的时候，才能确定我们已经爬取到了数据，这时我们才能记录爬取的位置

   ```python
   for i in range(totalpages):
       url = '***page'+str(i)
       priority=totalpages-i
       yield Request(url=url,priority=priority,callback='self.next')

   ```

   ```python
   f = open('filename','r+') # 程序开始我们要读取上次程序断掉的时候爬取的位置
   page = int(f.read())-20   # 这里减20
   f.close()
    ......
    ......

   def first(self,response):
       for i in range(self.page,totalpages):   # 这里循环到上次断掉的page开始
           url = '***page='+str(i)
           priority = totalpages-i             # 优先级从低到高
           yield Request(url=url,         
                         meta={'page':i},      # 把page传递到下一个函数
                         priority=priority,    # 设置优先级
                         callback=self.next    # 回调下一个函数
                        )

   def next(self,response):
   	f = open('filename','wb')               # 请求成功，把当前的page保存    
   	f.write(response.meta['page'])
   	f.close()
       
   减20是因为，虽然请求是按照优先级高低顺序发送的，但是也避免不了小范围的乱序，一方面是因为scrapy多线程，另一方面也和爬取网站的服务器响应速度有关，这个值也要是情况而定
   减20可以尽最大可能保证数据的完整性
   ```

   ​













```python

```

































