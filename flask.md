flask是轻量级的因为flask仅包含Werkzeug(路由模块)和jinja2（模板引擎），这两个也是flask框架的核心

其他的功能都得需要用第三方扩展

![55653716705](C:\Users\dream\AppData\Local\Temp\1556537167052.png)



![55653722923](C:\Users\dream\AppData\Local\Temp\1556537229233.png)



http://flask.pocoo.org/extensions/



```python
<> 定义路由的参数  <>内需要起名字
@app.route('/orders/<int:order_id>')
def get_order_id(order_id):
    # 需要在视图函数的（）内填入参数名，name后面的代码才能去使用
    return 'order_id'%order_id


处理int:  用来限定类型（默认为str）
	  (大致原理是将参数强转为int，如果成功，则可以进行路由匹配，否则就无法匹配该路由)

```

jinja2模板引擎

```python

列表的使用

{{ mylist }}
{{ mylist.1 }}
{{ mylist[1] }}

for循环使用快捷键，，先写for然后按住tab键进行补全

# 过滤器（另外还有很多）

{{ url_str | upper }}  #字符串变大写
{{ url_str | reverse }}  #字符串反转

# 过滤器的链式调用
{{ url_str | upper|reverse|lower|reverse }}


```

web表单

```python
#目的：实现简单的登录的逻辑处理
1. 路由需要有get和post两种请求方式 ---需要判断请求方式
2. 获取请求的参数
3. 判断参数是否填写 并且 密码是否相同
4. 如果判断都没有问题，就返回一个success
5. 给模板传递消息  flash--> 需要对内容加密，因此需要设置secret_key,作为加密消息的混淆 （模板中需要遍历消息）-----flash(u'密码不一致')	 如果出现编码问题，可以再前面加个u

app.secrrt_key = 'itemdjposjfps'

@app.route('/',methods=['GET','POST'])
def index():
    # request:请求对象 --> 获取请求方式，数据
    # 1. 判断请求方式
    if request.method == 'POST':
        # 2 .获取请求参数
         username = request.form.get('name')
		password = request.form.get('password')        
		password2 = request.form.get('password2')

		# 3 判断参数是否填写
		if not all([username,password,password2]):
            # print('参数不完整')
            flash(u'参数不完整')	 
         # 4  密码是否相同
         elif password != password2:
                # print('密码不一致')
                flash(u'密码不一致')	
         else:
            return 'success'
    return render_templatr('index.html')
    
模板中（flash消息的用法）：
<form method='post'>
	...
</form>

# get_flashed_messages 是内置的获取flash消息的函数，所以使用的时候需要加上（）
# 使用遍历获取闪现的消息
{% for messagr in get_flashed_messages() %}
	{{ message }}
{% endfor %}

```



wtf扩展 

```python
#可以进行表单验证
1.安装扩展 Flask_WTF
from flask_wtf import FlaskFrom
需要继承FlaskFrom才可以使用


form.username.label 取的是用户名，form.username 取的是框  
{{ form.username.label  }} {{ form.username }}


表单的验证函数
我们没有CSRF_token
{{ form.csrf_token() }}

验证参数，WTF可以一句话就实现所有验证
if login_form.validate_on_submit():
    # 如果能够走到这里说明校验成功，可以提交
    return 'success'
else:
    # 如果验证不通过  闪现一个消息
    flash('参数有误')


```

```python
数据库的模型，需要继承db.model
db = SQLAlchemy(app)

class Role(db.Model):
    # 定义表名
    __tablename__ = 'roles'
    
    # 定义字段
    # db.Column 表示是一个字段
    # db.Foreignkey（roles.id） 代表是外键 db.Foreignkey（表名.id）
	id = db.Column(db.Integer,primary_key =True)
    name = db.column(db.Strint(16),unique=True)
    role_id = db.column(db.Integer,db.Foreignkey(roles.id))
    
```



```
和在单模块中创建蓝本不同，当在子包中创建蓝本时，为了方便其他模块导入蓝本对象，这是蓝本对象在蓝本子包的构造文件中创建。而且因为蓝本在构造文件中定义，为了把路由、错误处理器、请求处理函数等和蓝本对象关联起来，需要在构造文件中导入这些模块，为了避免循环依赖，在构造文件的底部添加这些语句


from flask import Blueprint
front_bp = Blueprint('front',__name__)
from myapp.bluefrints.front import views,errors


程序中还可以添加： 自定义命令，shell上下文处理器，模板上下文处理器，错误处理函数 等 
```



建立模型

```
用户模型作为关系的中心，其他数据木星大多和它简历一对多的关系






```

































