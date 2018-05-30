确保同一个局域网内（电脑和手机在同一个网段的）

在手机上配置ip

用fiddle抓包，所以代理的算口设置设置fiddle的端口8888

设置好之后，手机的通信，就会被fiddle拦截到

会抓取到json的文件，传输数据用的，抓到json的文件，就可以拿到想要的数据了

拿到图片链接就可以发起请求，下载图片了



json取字段值很好取了



域名只需要写主域名就可以了

app的前面需要加上capi



offset需要拼接路径的时候，写两个值，在start_url里面写字符串拼接



保存图片的历经写一个imgurl字段



python的文件转换成json用的是dumps

json的转换成python文件用的是loads





```python
import  json
from douyu.items import douyuitem
class DouyuSpider(scrapy.spider):
    name = 'douyumeinv'
    allowed_domains=['capi.douyucdn.cn']
	offset=0
    url='https://capi.douyucdn.cn/api/v1/getVerticalRoom?limit=20&offset='
    start_urls=[url+str(offset)]
    def parse
        #把json格式的数据转换成python格式data段是列表，列表里的每段是一个字典
        data = json.loads(response.text)["data"]
        for each in data:
            item=DouyuItem()
            item['nickname']=each['nickname']
            ......
            yield item
    # 这样是处理一页的数据
    # 接着处理下一页的数据
        self.offset=20
        yield scrapy.Requests(self.url+str(self.offset),callback=self.parse)
        
        
pipeline.py

图片下载出来
需要一个useragent
在fiddle里面获取一个手机浏览器的useragent

在setting里面设置，默认有这个配置，打开，把手机端的useragent写进去即可
pipeline打开，里面有imagepipeline配置进去（专门处理图片的）
设置下载图片保存的位置


images_urls_field='图片字段'
images_store='图片保存的路径'



在程序里面直接使用自己定义的setting里面的变量
from scrapy.utild.project import get_project_settings
from pipelines.images import ImagesPipeline

class ImagesPiprline(ImagesPipeline):
    # get_project_settings这个方法就可以获取setting里面设置的变量值
    IMAGES_STORE=get_Project_settings().get('IMAGES_STORE')
    
    #这个方法，获取图片的链接，发送图片的亲求
    # 发送scrapy.requests请求,发送完这个请求之后，会通过item_completed这个函数做处理
    def get_media_requests(self,item,info):
        image_url=item['imagelink']
        yield scrapy.Request(image_url)
    def item_competed(self,results,item,info):
        image_path = [x["path"] for ok,x in result if ok]
        
        #网上下载的图片默认的名称是他的hash值
        #给从网上下载的图片重命名
        # os.rename就是重命名的函数
        os.rename(self.IMAGES_STORE+'/'+'image_path[0]',\
                  self.IMAGES_STORE+'/'+item['nickname']+'.jpg')
            
        item['imagePath'] = self.IMAGE_STORE+'/'+item['nickname']
        
        return item
scrapy里面有一个图片处理的方法
```



pipeline源码里面有很多管道文件，其中有一个image文件

里面有很多方法



重载下面这两个方法用来保存图片

get_media_requests()

item_completed()





