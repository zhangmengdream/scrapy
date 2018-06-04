1148646628@qq.com     AA199110711aa



有时候主目录在pycharm中设置成根目录可以导入文件，在黑屏中却会显示出错，这时候是因为黑屏并不知道主目录的位置，需要在setting中设置

setting是scrapy 的主入口，我们可以将pythonpath的操作放到setting中操作

```python
import os
import sys
# sys.path.insert(0,"这里写python path")
BASE_DIR=os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0,os.path.join(BASE_DIR,'ArticleSpider'))
#这样会把我们的python path加入到目录中，这样就可以找到util了

这样路径就可以生效了
```



## CrawlSpider源码剖析



```pthon
CrawlSpider是个  类

Spider里面有start_request  会遍历start_urls

CrawlSpider继承Spider 所以入口也就start_request
start_urls的回调函数是parse函数

注意：在文件中不能再重载parse函数了（也就是不能写这个函数了），因为CrawlSpider里面有这个函数，不能覆盖，否则很多规则就不能用了

_parse_response()，这个函数是parse返回的函数，也是CrawlSpider的核心函数
返回里面有参数
return _parse_response(response,self.parse_start_url,cb_kwargs={},follow=True)

def _parse_response()  这个函数会检测是否有callback  也就是第二个参数 parse_start_url

parse_start_url() 这个函数可以重载，功能和Spider重载parse效果一样




```

```python

def _parse_response(self, response, callback, cb_kwargs, follow=True):
    # 这一步 有callback就处理，没有就不处理
    if callback:
        cb_res = callback(response, **cb_kwargs) or ()
        #这步就是将result的返回直接返回回来了
        cb_res = self.process_results(response, cb_res)
        for requests_or_item in iterate_spider_output(cb_res):
            yield requests_or_item
            
	#这一步是crawlSpider的核心
    #follow和self._follow_links默认都为True，所以默认会往下执行
    if follow and self._follow_links:
        for request_or_item in self._requests_to_follow(response):
            yield request_or_item
            
            

注1：def set_crawler(self, crawler):
        super(CrawlSpider, self).set_crawler(crawler)
        self._follow_links = crawler.settings.getbool('CRAWLSPIDER_FOLLOW_LINKS', True)
在settings里面设置这个参数，如果设置为False 则rule 就没有作用了，就不会调用rule的规则了
这个值默认是True，所以会跟踪
self._follow_links = crawler.settings.getbool('CRAWLSPIDER_FOLLOW_LINKS', True)   





注2：def _requests_to_follow(self, response):
    #这里判断传进来的类型是不是response
        if not isinstance(response, HtmlResponse):
            return
        #seen是函数内部的变量，所以这个seen是针对当前的response做的，会记录response里面提取出哪些url
        #然后通过这个set去重
        seen = set()
        #enumerate(self._rules)  把_rules变成一个可迭代的对象
        for n, rule in enumerate(self._rules):
            # 这里是判断有没有在seen、里面，因为seen会去重
            links = [lnk for lnk in rule.link_extractor.extract_links(response)
                     if lnk not in seen]
            if links and rule.process_links:
                links = rule.process_links(links)
            for link in links:
               
                seen.add(link)
                r = self._build_request(n, link)
                yield rule.process_request(r)
                |
                |
                V
            
   def _build_request(self, rule, link):
       #将link进行request
        r = Request(url=link.url, callback=self._response_downloaded)
        r.meta.update(rule=rule, link_text=link.text)
        return r

```

```python
	这两个函数都是可以重载处理的 
    def parse_start_url(self, response):
        return []
# 重载这个函数，和之前的parse功能是一样的，在crawl_spider中parse函数不能重载，已经被占用了
    
    
	#这个函数处理parse_start_url 函数的return
    def process_results(self, response, results):
        return results
```

## rule，LinkExtractor的一些参数用法

```python
    rules = (
        Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )
    
    
LinkExtractor：
参数
allow=r'www.lagou.com/jobs/'  代表符合这个规则的就提取
deny=r'www.lagou.com/jobs/'   代表符合这个规则的就不提取

allowed_domains=['www.lagou.com']  指这个域名下的我才做处理，如果不是这个下面的域名，就直接抛弃
deny_domains=['www.lagou.com']  指这个域名下的我不做处理，如果不是这个下面的域名，就处理 

restrict_xpaths=()  可以进一步取限定url
tags  = ('a','area')  默认从中和两个标签中寻找
attrs = ('href')      获取href
```

## 如何用，让他来爬取拉钩的全站

```python


response.css('.name')extract()


```

#### Middleware是介于request和response之间的钩子框架

他可以全局修改request和response

middleware 在setting中配置的 数字越小处理越优先，自己设置的要数字大一些，这样才不会被覆盖

```python
下载器中间件middleware里面需要自己去写的几个函数：
一、process_request(request,spider)
	当每个request通过下载器中间件时，该方法被调用
    返回： None 	   或者
    	  Response   或者
    	  Request    或者
    	  raise（IgnoreRequest）
         
二、process_response(request,response,spider)
	会在每个response返回时被调用
    返回： Response   或者
    	  Request    或者
    	  raise（IgnoreRequest）
            
三、process_exception(request,exception,spider)




```



 







seo ？？？





