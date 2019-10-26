namedtuple模块

from collections import namedtuple

这个模块的功能是将这个元祖变成一个类（节省空间，映射）

```python
User = namedtuple("User",["name","age","height"])
user_tuple = ("bobby",29,175)
user = User(*user_tuple,"master")
print(user.age,user.height,user.height)


namedtuple 源码  实质上就是声明了一个类，这个类继承tuple。

因为是支持iter对象，所以 list和tuple都可以，他们都说iter对象，dict也是可迭代对象，所以也是可以的
user_list = [1,2,3,4]
x = User._make(user_list)
print(x.name,......)

namedtuple 的好处  1.节省空间   2.使用tuple的特性，可以拆包（**other）

```

##### defaultdict（是使用c语言实现的，所以性能是非常高的）

```python
from collections import defaultdict

# 例1 ： 计算users里面重复的元素对应的个数
users = ["bobby1","bobby1"]

user_dict = {}
for user in users:
    # 这不操作的意思是，如果那个值不存在就会设置默认值(第一个参数是key，第二个参数是默认值)
    # 这里的做法是，先默认为0  在进行加1  的操作，就不会有什么问题了
    # setdefault 比 判断更加有效，因为这个少做了一次查询的操作
    user_dict.setdefault(user,0)
    # setdefault 完成类似这步的操作
    # user_dict[user] = 1
    user_dict[user] +=1
    

    
# 参数为传递一个可调用的对象    这里的list是个类，是个可调用的对象
# 此时 bobby这个key不存在，这个函数是可以在传入的key不存在的时候，调用传递进来的对象，给传递进来的key设置可以默认的空值（list --> []   int --> 0 ）
default_dict = defaultdict(list)
default_dict['bobby']
pass


所以例1可以写为  (使代码的严谨性变高了，很多初始化 )

default_dict = defaultdict(int)
users =  ["bobby1","bobby2","bobby2","bobby3","bobby3","bobby3"]
for user in users:
    default_dict[user] += 1


```

##### deque(c语言写的源码，性能很高)

```python
from collections import deque

# 双端队列
 deque 是线程安全的






```

##### Counter  统计 (Counter是dict的子类)

```python
from collections import Counter
users =  ["bobby1","bobby2","bobby2","bobby3","bobby3","bobby3"]
user_counter = Counter(users)
print(user_counter)
# 统计结果  从大到小进行排列
Counter({'bobby3':3,'bobby2':2,'bobby1':1})
# 对字符串中字母出现的次数进行统计
counter_test = Counter('asuidhfiuogjd')
# 进行合并统计
counter_test.update('sdjofijgi')
# 统计出出现次数最多的前n个元素  top -> n
print(counter_test.most_common(2))
print(counter_test)
```

##### OrderedDict  可排序dict

```python
from collections import OrderedDict

python3里面的dict 默认是有顺序的  先添加的放在前面（python3下面 dict和OrderDict都是有序的）
python2下面的dict是无序的（使用OrderDict）

pop（key）
popitem()
```











买糖  礼物  礼金



装修好房子    定好日子

一家人一块去我家，



















































































