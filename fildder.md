fiddler工作原理

```python
客户端 ---------------------> fiddler ---------------------> web服务器
      <---------------------代理服务器 <---------------------
    
fiddler是以代理web服务器的形式工作的，它使用代理地址127.0.0.1，端口8888
fiddler是http和https都可以抓包的

```

fiddle有断点功能

1. 向上的断点 （请求）
2. 向下的断点 （响应）



AllProcesses 后面 数字前面 点击箭头朝上为上行的断点 ，点击箭头朝下为下行的断点

capturing ： 捕获（进行时）（点击捕获，再点击，停止捕获）





cache是保存在内存之中的

cookie是保存在客户端硬盘之中的，可以存放一些被加密的信息，重要的信息

session是放在服务器内存中的

token是开发人员自己定制的验证身份的信息，可以放在任意位置（可放在请求头，

请求体，cookie中等。。。）








































