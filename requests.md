# requests



### 发送请求

```python
import requests
r = requests.get('https://www.baidu.com')

这时r就是一个response的对象了
有多种类型
get post put delete head options 几种方式的请求都可以发送，有的请求需要参数的话
就传递进去data={'key':'value'}

例如：
r = requests.get('https://www.baidu.com')
r = requests.post('https://www.baidu.com',data={'key':'value'})
r = requests.put('https://www.baidu.com',data={'key':'value'})
r = requests.delete('https://www.baidu.com')
r = requests.head('https://www.baidu.com')
r = requests.options('https://www.baidu.com')
```

### 传递url参数

```python
如果传递的参数比较多，可以构建一个字典，用params关键字参数传递进去
payload = {'key1':'value1','key2':'value2'}
r = requests.get('https://www.baidu.com',params=payload)

此时的r.url就会时下面的样式
https://www.baidu.com?key2=value2&key1=value1
    
注意：字典里面值为None的键时不会被传递到url里面的，还可以传递进去一个键对应一个列表
payload={'key1':'value','key2':['value2','value3']}
r = requests.get('https://www.baidu.com',params=payload)

此时传递进去之后返回的值为
https://www.baidu.com?key1=value&key2=value2&key2=value3
```

### 响应内容

```python
import requests
r = requests.get('https://api.github.com/events')
r.text
此时返回的就是响应内容，即html页面的内容

requests会字典解码来子服务器的内容，大多数的unicode字符集都会被无缝的解码

请求发出后，requests会基于http头部对响应的编码做出有根据的推测。当访问r.text的时候，requests就会使用其推测的本文编码。可以用r.encoding来检测时使用了什么编码

>>> r.encoding
'utf-8'
>>> r.encoding='ISO-8859-1'

这里表示原来的编码是'utf-8' 把编码改为了'ISO-8859-1'

如果出现乱码，可以使用r.content 来找到正确的编码，然后使用r.encoding='正确的编码'这样的方式来改为正确的编码，这样就可以正常的展示我想要的东西了

```

### 二进制响应内容

```python
r.content 代表以字节的方式访问请求响应体，对与非文本请求

以请求方式返回的二进制数据，创建一张图片：
from PIL import Image
from io import BytesIO

i = Image.open(BytesIO(r.content))
```

### json响应内容

```python
json的响应内容requests中都会有一个json的解码器
r = requests.get('www.baidu.com')
r.json()

注意：成功调用了json（）并不代表返回就一定成功了，有了可能返回错误的数据是一个json格式
所以需要配检查code是否成功
r.status_code()
或者
r.raise_for_status()
```

### 原始相应内容  ？？？？（运行出错）

```python
极少数情况下如果你想获取来自服务器的原始套接字响应，可以访问r.raw
r = requests.get('https://www.baidu.com',stream=True)
r.raw
r.raw.read(10)

一般情况下可以将文本流保存到文件中
with open(stream,'wb') as fp:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)


```

### 定制请求头

```python
如果想为请求添加http头部，只需要传递dict给headers参数就可以了
url='https://www.baidu.com'
headers={'useragent':'my-app/0.0.1'}
requests.get(url=url,headers=headers)

但是requests不会基于指定headers的具体情况改变自己的行为，只不过在最后的请求中，所有的header信息会被传递进去
```

### 更加复杂的post请求

```python
传递data参数

payload = {'key1':'value1','key2':'value2'}
r = requests.post('https://www.baidu.com',data=payload)
r.text

还可以为data传入元祖列表
payload = (('key1','value1'),('key2','value2'))
r = requests.post('https://www.baidu.com',data=payload)
r.text

如果传入的数据是一个string，那么数据会被直接发布出去
import json
payload = ('key1','value1')
r = requests.post('https://www.baidu.com',data=json.dumps(payload))
r.text

还可以直接用json传递参数

payload = ('key1','value1')
r = requests.post('https://www.baidu.com',json=payload)
r.text
```

### post一个多部分编码的文件

```python
上传多部分编码
url = 'http://httpbin.org/post'
files = {'file':open('report.xls','rb')}

r.requests.post(url,files=files)
r.text

{
    ...
    "files":{
        "file":"<censored...binary...data>"
    }
    ...
}
```

可以显示的设置文件名，文件类型和请求头

```python
url = 'http://httpbin.org/post'
files = {'file':('report.xls',open('report.xls','rb'), 'application/vnd.ms-excel', {'Expires': '0'})}

r.requests.post(url,files=files)
r.text

{
    ...
    "files":{
        "file":"<censored...binary...data>"
    }
    ...
}
```

可以发送作为文件来接收的字符串

```python
url = 'http://httpbin.org/post'
files = {'file':('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

r.requests.post(url,files=files)
r.text

{
    ...
    "files":{
        "file":"<censored...binary...data>"
    }
    ...
}
```

### 响应状态码

```python
>>> r.requests.post(url,files=files)
>>> r.status_code
200
-------------------------------------------------------------------
还可以查询
>>> r.status_code=requests.codes.ok
True
-------------------------------------------------------------------
如果发生了错误请求 4.. 5.. 的状态的错误响应的时候，可以通过Response.raise_for_Status()来抛出异常

>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404
>>> bad_r.raise.for_status()

Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
-------------------------------------------------------------------
如果状态码为200的时候，调用raise_strtus_code会返回none
 
```

### 响应头

```python
>>> r.headers
{'Connection': 'Keep-Alive', 'Content-Length': '17931', 'Content-Type': 'text/html', 'Date': 'Tue, 29 May 2018 08:37:48 GMT', 'Etag': '"54d97485-460b"', 'Server': 'bfe/1.0.8.18'}

可以展示一个字典形式的服务器响应头，根据RFC2616.HTTP头部是大小写不敏感的

所以
>>> r.headers['content-type']
'text/html'
>>> r.headers['Content-Type']
'text/html'

接收者可以合并多个相同名称的header阑尾，把他们合为一个field-name：field-value配对，将每个后续的栏位值一次追加到合并的栏位值中，用户都会隔开即可，这样做不会改变信息的语义
```

### Cookie  后面设置cookie那块不太理解？？

```python
如果某个响应中包含一些cookie，可以访问他们
>>> url = 'https://github.com/zhangmengdream/scrapy'
>>> r = requests.get(url)
>>> r.cookies['example_cookie_name']
example_cookie_name

也可以把自己的cookie设置上去
>>> url = 'https://github.com/zhangmengdream/scrapy'
>>> cookies=dict(cookies_are='working')
>>> r.requests.get(url,cookies=cookies)
>>> r.text

cookie的返回对象为RequestsCookiejar，他的行为和字典类似，
但是接口更为完整，适合跨域名跨路径使用，还可以把cookiejar传递到requests中使用
>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie','yum',domain='httpbin.org',path='/cookies')
>>> jar.set('gross_cookie','blech',domain='httpbin.org',path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url,cookies=jar)
>>> r.text
'{"cookies":{"tasty_cookie":"yum"}}\n'

```

### 重定向与请求历史

```python
默认情况下，除了HEAD,Requests会字动处理所有重定向
可以使用响应对象的history方法来追踪重定向

Response.history 是一个response对象的列表，为了完成请求而创建了这些对象
这个对象列表按照从老到最近的请求顺序
例如： 一个重定向的例子

>>> r=requests.get('http://github.com')
>>> r.url
'https://github.com'
>>>r.status_code
200
>>>r.history
[<Response [301]>]

可以使用 allow_redirects=False参数，来禁止重定向

如果使用了HEAD,也可以启动重定向
可以使用 allow_redirects=True参数
```

### 超时

```python
可以设置 timeout=1，代表一秒为超时时间

>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)

        
timeout仅仅对链接过程有效，与响应体的下载无关，timeout并不是整个下载响应的时间限制，
```

### 错误异常

```python
遇到网络问题（如DNS查询失败，拒绝连接等）requests会抛出ConnextionError异常（连接异常）
如果HTTP请求返回了不成功的状态码，Response.raise_for_status(),回抛出HTTPError异常
请求超时抛出Timeout异常
请求超过了设定的最大重定向次数，抛出TooManyRedirects异常
所有Requests显示抛出的异常都继承字requests.exceptions.RequestRxception
```

# Requests高级用法



### 会话对象

```python
会话对象能够让你跨请求保持某些参数。他也会在同一个session实例发出所有请求之间保持cookie，期间使用urllib3的connection pooling功能，所有如果你想同一主机发送多个请求，低层的TCP链接将会被重用，从而带来显著的性能提升

跨请求保持一些cookie(第一次请求得到的cookie第二次请求页面的时候会带过来)
>>> s=requests.session()
>>> s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
>>> r = s.get('http://httpbin.org/cookies')

>>> r.text
'{"cookies": {"sessioncookie": "123456789"}}'

```







































































































































