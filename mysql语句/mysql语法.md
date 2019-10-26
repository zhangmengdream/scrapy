sql的innnodb和myisam



mysql的存储是以文件的形式存储的 

ENGINE=InnoDB 表的存储引擎为innodb

把表的存储引擎改为myisam

alter table 表名 engin=



查看表的信息，里面包含表的搜索引擎，也可以查看到

- show create table  userinfo ;



### MyISAM

```python

myisam表是独立于操作系统的，这就说明可以轻松的将其从windows服务器移植到linux服务器
每当我们建立一个myisam，引擎的表是，就会在本地建立三个文件，例如表名为info
三个文件分别为

info.frm   存储表框架结构
info.MYD   存储数据
info.MYI   存储索引

MyISAM表无法处理事务，这就意味着有实物处理需要的表，不能使用MyISAM存储引擎，MyISAM存储引擎特别适合以下的情况：
1. 选择密集型的表。MyISAM存储引擎在筛选大量数据时，非常迅速，这是他最突出的优点
2. 插入密集型的表。 MyISAM的并发插入特性允许同时选择和插入数据。例如：MyISAM存储引擎很适合管理邮件或者web服务器日志数据
```



### innodb

```python
健壮的事物存储引擎，为用户操作非常大的数据，提供了强大的解决方案
。。Innodb是默认的存储引擎，例如表名为user
innodb会生成两个文件

user.frm    存储表的框架结构
user.ibd    存储表的数据和索引


适合的情况
1. 更新密集的表。Innodb存储引擎特别适合多重并发的更新请求.
2. 事物,innodb存储引擎是支持事物的标准mysql存储引擎
3. 自动灾难恢复, callback
4. 外键约束.mysql中支持外键的存储引擎只有innodb
5. 支持自动增加列auto_increment属性
```



### myiusam和innodb的区别

```python
innodb和myisam是mysql非常重要的两个存储引擎
innodb支持事务处理，比较安全（处理过程中，全部成功才会提交，只要有时间的就会失败，不提交）
myisam 支持大量的搜索，常用于博客，微博等，引擎执行效率更高
innodb 会产生两个文件 myisam会产生三个文件
```

### 事物处理

```python
alter table name engine=innodb   把引擎改为innodb

查询是否为手动提交
select @@autocommit (1为自动提交0为手动提交)

先改成手动提交（默认是自动提交）
set autocommit=0

开始
begin；

代码块

执行提交或者回滚
commit work;    提交
callback work;  回滚

```

### 对于表结构的操作

```python
给表添加新的字段
alter table user add  字段名  约束条件
```



### 聚合函数

```python

```



### 多表联查

```python
两个关联表




```

user表：

![52768631478](C:\Users\dream\AppData\Local\Temp\1527686314784.png)

goods表：

![52768623330](C:\Users\dream\AppData\Local\Temp\1527686233305.png)

goods表的gid是自增id，uid是user表的自增id，这个uid就是goods表的外键，通过这个字段进行关联



```python
比如查询 张三 都买了哪些商品：查询李四的id:

SELECT * FROM goods where uid in (SELECT id from user WHERE username='李四')
这样还不算是多表联查，因为并没有用到uid，这只是一个子查询

以上可以看出这两个表是有关联的，通过uid来关联
```

### 隐式内连接查询

其实也是内连接，隐式的因为没有使用join 

```python
SELECT * from goods,`user` where goods.uid=`user`.id and `user`.username='张三'

SELECT * from goods,`user` where goods.uid=`user`.id and uid=1

这两语句结果相同，得到user表和goods表的所有张三的数据
```

如果只获取某个表的某些数据，需要指定

```python
select user.id,user.username,goods.gid from user,goods where goods.uid=user.id and uid=1
```

### 显式内连接查询 INNER JOIN

```python
SELECT * from user inner join goods on user.id=goods.uid and user.id=1

on代表条件
A表  inner join  B表  on  条件  inner join  C表  on  条件 ...
```

三表联查的话，在on条件之后，再加 inner join  表名 on 条件  就可以了

关键的是关联条件，外键 

#### 左关联left join

以左表为主表，右表为辅表，左表的所有数据都拿出来，右边的有关联的拿出来，没有关联的置为空。

右关联与左关联相反









![52768735271](C:\Users\dream\AppData\Local\Temp\1527687352710.png)







### select数据的查询

主体结构：

- select    字段名 (或)*   from   表名

查询给某个字段起别名

- select   id i  from  表名；

- select  id  i,  username  u  from  表名；

  ​

### where条件

一、比较运算符

1.  `>`

   select   *   from   表名  where>20    (下面的可以套用这个例子)

2.   <

3. `>=`

4.  <=

5.  =

6.  !=   / <>

二、逻辑运算符

- and  并且
- or  或
- between and  在...之间
- in   在...里
- not  in  不在...里面

三、子查询

条件还是一条sql语句

select  *  from  userinfo where id in (select id from userinfo  where age>30)

四、order by  排序

order  by  字段名  asc/desc    升序/降序

select * from userinfo where age>18 order by id  desc; 

order by 默认是升序，order by 不论是商品，帖子，都是按照时间倒叙查的，都是最后发的在最前面，之前发的在最后面（就像淘宝评论一样）

五、is  not  is  **用来查询某个字段是否为空**

###### 只是用来查询数据是否为空时使用

先设置一个空的字段，便于查询    insert  into  a(username)  values(null)

查询username为空的数据，

select *  from  a  where  username=null;    （查询不到数据）

(空是一个特殊值，查询空等于空，根本查询不到数据， 这里就需要用到is)

select  *  from  a  where  username is  null;    查询name是空的数据

select  *  from  a  where  username  is  not  null;  查询name不是空的数据

六、limit   offset

limit   5  取出5条数据   等同于  limit  0,5   （从开头位置取出5条数据）

- select * from userinfo  limit 2;

条件组合式的查询

- select * from userinfo where age=30 order by id desc limit 2;

查询年龄等于30 的id最大的2条数据

##### group  by  分组

select  count(*) from userinfo

统计男和女 分别有多少人

通过分组，相同的放在一组

select  count(*) from  userinfo  group by sex;

select sex,count(*) as total from  userinfo group by sex;

统计每个班级的男和女分别有多少人

select   sex,chassid,count(*)   as  num   from userinfo  group  by  sex,chassid

group  by  sex,chassid(这里代表给谁分组)



**having**条件相当于你的where

按照班级性别来分组，查询每班人数大于2人的数据

select  sex,chassid,count(*)  as num  from  userinfo  group  by  sex,chassid  having  num>2;

按照班级性别来分组，查询python1708的数据

select sex,count(*),chassid  from userinfo group by chassid,sex having chassid='python1708' and count(*)>2;

##### 模糊查询like

主体结构：

like '%字符%'  包含  （包含关系）

select  *  from  userinfo  where  name   like  '%三%'

'%三%'

'三%'   表示以三开头

like  '字符%'   以某个字符开始的数据

like  '%字符'   以某个字符结束的数据

like  '%字符%'   包含某个字符的数据



### delete删除

主体结构：

delete  from 表名  [where  条件]

delete from 表名  where  id=1;

删除的时候，如果没有条件，表示删除所有

不过删除后，自增的记录仍然在，也就是上次id到哪个位置，下面再次插入值的时候，也会从哪里开始记录

用truncate  表名；  清空表，自增记录也删除，重新添加数据的时候，会重新从1开始。

##### 清空表的方式：

1. truncate 表名；

2. delete from 表名；

   alter   table  test   auto_increment=1;    把test表的自增设置为1.

注意：

1. 删除的时候，如果没有条件  删除所有的数据
2. 如果想要清空表的话 truncake 表名
3. 删除后的数据，自增依然记录当前的数据的位置
4. 如果想delent之后，插入数据从1开始，就需要设置auto_increment=1  回归设置
5. alter  table  userinfo  auto_increment=1;

DISTINCT 去除重复的值

select  distinct  age  from  test;

查询取出重复的值后的age字段的值

#### update修改

update  表名 set  字段名1=值1[字段名2=值2]

不加条件的话，全都修改了

注意：当修改的时候，没有where条件的时候，所有的数据都修改。

update  表名  set   age=age+2 ;



### 聚合函数

1. count() 统计个数
2. max()  最大值
3. min()  最小值
4. sum()  求和
5. avg()   求平均数







### 数据库的导入导出

































select * from user,goods where goods.uid=user.id and uid=2 ORDER BY goods.num LIMIT 1







sql 修改字段名的操作？？？

















数据结构需要掌握，

堆			 

栈     	 	 特点：先进后出

队列   		 特点：先进先出 （排队）

列表（链表）  特点：和顺序表一一对应，（顺序表，也可以当成是一根线，是有节点的）

​		链表相当于一根铁链，是环环相扣的



拿到数据是为了操作的，也就是增删查改这些操作



别人问你数据结构，其实就是问他的特点

如何去优化：

存储的数据，最频繁的操作就是查，之后显示出来

​	数据的数据结构，num  string  字典 集合 ...  每一种数据类型都有各自的特点，对应形式存储



怎么判断用那种数据结构，针对不同的数据类型？ 



如何优化排序，根据用户的点记录，还是点赞率还是采纳率，综合给一个百分比，来排序



shart  快排 ？？？

如果需要逆序的话，先快排，再reverse（从打到小逆序）

如果面试的时候，被问到排序，应该说选择，冒泡，快排  这个顺序  

选择：

冒泡：

一个一个挨着比，还是从前比到后的区别。



猜数字的小游戏：

0-100 之间的数，我随机生成一个数，以什么样的方式，能够最快的找到随机生成的这个数

（二分查找）

（二叉树 ） 红黑树



问的目的，不是怎么去快排，而是如何去排序，也就是排序的原理（掌握原理，写两个例子熟悉）



面试时，遇到不会的可以扯一些和数据相关的东西

比如问到数据结构时，可以扯一些数据相关的，慢慢的往自己的方面靠拢



为什么我在网址上 输入  www.baidu.com    就能够跳转到百度的首页

把这里搞清楚http就基本可以了

端口

ip

如何加密解密

硬件 路由器  

如何通过一条网线去上网

dns域名解析，怎么取解析



（http和https的区别  实现原理？？？）









爬虫主要就是爬取数据



爬阿里里面的时候

里面有一个动态的 生成码，这个动态的生成码，如果你按照正常的看都看不懂，是因为阿里用了自己的字体  自己设计的字体，非常生僻的字体，

所以这时候我们就需要把它的原生html爬去下来  ，看js来寻找漏洞，破解

随机生成六位数，这六位数有一定的规律，把规律找到就个很容易爬去了



视频类的比如爱奇艺，在爱奇艺上下载的电影，其他的播放器不能看，因为做了加密

爱奇艺有的电影需要加会员才可以看



如果把它的加密给破解掉，就可以不需要vip在线解析，把想看的电影，对应的目录爬取下来，在线解析



爬下来的数据是二进制的存储在什么数据库：

mongodb  redis 都可以

文本型和字典型的



爬去的视频，肯定是以二进制的形式保存起来，可以保存在redis，也可以保存在mongodb

，然后把我想要的数据，展示出来



爬去数据很简单，最重要的是，爬到的数据可以用

解密出来  加密的方式有md5/等 

支付宝的加密就是md5



挨个分，挨个存储

数据清洗的方面了



爬虫的价值，就是爬出我们有用的数据，把有用的数据清晰出来



比如，京东的爬去淘宝的信息，爬出一些一星的差评的用户信息，给他们做一些推广，把用户买的差评的东西，用京东推广过去，用户就有可能感觉京东比较靠谱，来京东购买东西

所以一星用户就是有价值的信息



用分布式解决大数据的存储，爬虫是跟大数据相关的，比如爬取了海量的数据之后，这个数据能做什么

这时候，就需要进行大量的计算，计算我们要筛选出哪些东西来，筛选出这些东西之后，就可以针对的做一些二次操作，

分布式的理念就是，相当于淘宝在双十一的时候，在零点那一刻，他的访问量亿级以上



两千万用户的话，大概需要多少服务器去支撑

并发是怎么实现的：  高并发，分布式，多台服务器，就是硬件。

分布式的原理是，比如我们去买东西，把某一个单独的商品放到一块，手机， 电脑， 连衣裙  等分别存放在不同的服务器上。归总再分开。

特点： 即使某一台服务器挂了，也不会影响其他的服务器正常的操作

分布式的优点，可以并发的去处理海量的数据

分布式即使预估的并发数据小了一点，即使某一台服务器挂了，也不会去影响其他的服务器



反爬虫 

图片验证码 ， 拉的图片  倒排字等     精通      掉



挑一到两个深了说。











https://blog.csdn.net/shakespeare001/article/details/51360732  时间复杂度？  一点要看











```
SELECT (SELECT name FROM TA WHERE id = TB.id1) AS id1,
       (SELECT name FROM TA WHERE id = TB.id2) AS id2
FROM TB
```





```oy
select (select name from user where id=tb.id1) as id1,(select name from user where id=tb.id2) as id2 from tb;
```



```python
select c.name,d.name from (select a.id1 id1,a.id2 id2,a.name name from TA a,TB b where a.id1=b.id1) temp c,TA d where c.id2=d.id2

```































比如秒杀的时候，抢到的时候还有货，要付款的时候，就显示没有货了。单项管道，有个判断条件，

比如商品只有一个数量为1，如果数量为零了，立马通道关掉，相当于，进都进不去，图片就换成灰色的，就没有点击事件了。



十几个人同时抢一张票的时候，此时设置了一个宽的通道十几个人都可以进去，一个人付款成功后，从通道出去了，其他的通道就封闭了，就完成不了购票了

































































为什么一个字段可以设置两个索引？？





















