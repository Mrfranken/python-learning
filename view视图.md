- # 视图

  [TOC]

- 视图接受Web请求并且返回Web响应

- 视图就是一个python函数，被定义在views.py中

- 响应可以是一张网页的HTML内容，一个重定向，一个404错误等等

- 响应处理过程如下图：
  ![image](https://note.youdao.com/yws/api/personal/file/E63A6062766E46EFB43C575210A903AA?method=download&shareKey=7dd4dbc6b2be090ad17f5f762b6494bb)
---

### 视图的url解析包含两部分
- 在settings中指定 ROOT_URLCONF 的值
    - e.g.例如现在我们项目的名称是test3则 ROOT_URLCONF = 'test3.urls'
- 在应用中创建urls.py文件，定义本应用中的urlconf
    - urlpatterns是一个url()实例的列表
    - 一个url()对象包括： 正则表达式、视图函数、名称name
```python
from django.conf.urls import include, url
urlpatterns = [
    url(r'^', include('booktest.urls', namespace='booktest')),
] #使用include指定项目主url使用的urls匹配文件
```
- 匹配过程：先与主URLconf匹配，成功后再用剩余的部分与应用中的URLconf匹配
```python
请求http://www.itcast.cn/booktest/1/
在test3这个项目urls.py中的配置：
url(r'^booktest/', include('booktest.urls', namespace='booktest')),
在booktest应用urls.py中的配置：
url(r'^([0-9]+)/$', views.detail, name='detail'),
匹配部分是：/booktest/1/
匹配过程：在urls.py中与“booktest/”成功，再用“1/”与booktest应用的urls匹配
```
- 可以在url中传递参数，在url匹配过程中可以将url中的参数解析出来
    - 正则表达式非命名组，通过位置参数传递给视图：
     ```
     url(r'^([0-9]+)/$', views.detail, name='detail'),
     ```
    - 正则表达式命名组，通过关键字参数传递给视图，本例中关键字参数为id
     ```
     url(r'^(?P<id>[0-9]+)/$', views.detail, name='detail'), #这里views中的detail函数可以接受名字为id的参数
     ```
### HttpReqeust对象
- 服务器接收到http协议的请求后，会根据报文创建HttpRequest对象
- 视图函数的第一个参数是HttpRequest对象
- 在django.http模块中定义了HttpRequest对象的API

1. #### 属性
- 下面除非特别说明，属性都是只读的
- path：一个字符串，表示请求的页面的完整路径，不包含域名
- method：一个字符串，表示请求使用的HTTP方法，常用值包括：'GET'、'POST'
- encoding：一个字符串，表示提交的数据的编码方式
- 如果为None则表示使用浏览器的默认设置，一般为utf-8
- 这个属性是可写的，可以通过修改它来修改访问表单数据使用的编码，接下来对属性的任何访问将使用新的encoding值
- GET：一个类似于字典的对象，包含get请求方式的所有参数
- POST：一个类似于字典的对象，包含post请求方式的所有参数
- FILES：一个类似于字典的对象，包含所有的上传文件
- COOKIES：一个标准的Python字典，包含所有的cookie，键和值都为字符串
- session：一个既可读又可写的类似于字典的对象，表示当前的会话，只有当Django启用会话的支持时才可用，详细内容见“状态保持”

2. #### 方法
- is_ajax()：如果请求是通过XMLHttpRequest发起的，则返回True

#### QueryDict对象
- 定义在django.http.QueryDict
- request对象的属性GET、POST都是QueryDict类型的对象
- 与python字典不同，QueryDict类型的对象用来处理同一个键带有多个值的情况
- 方法get()：根据键获取值
    - 只能获取键的一个值
    - 如果一个键同时拥有多个值，获取最后一个值
```python
dict.get('键',default)
或简写为
dict['键']
```
- 方法getlist()：根据键获取值
    - 将键的值以列表返回，可以获取一个键的多个值
```python
dict.getlist('键',default)
```

#### POST属性
- QueryDict类型的对象
- 包含post请求方式的所有参数
- 与form表单中的控件对应
- 问：表单中哪些控件会被提交？
- 答：控件要有name属性，则name属性的值为键，value属性的值为键，构成键值对提交
    - 对于checkbox控件，name属性一样为一组，当控件被选中后会被提交，存在一键多值的情况
- 键是开发人员定下来的，值是可变的

在templates文件夹中定义这样一个模板：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/booktest/posttest2/" method="post">
    {% csrf_token %} <!-- 注意这里在做post时需要加上，具体原因不明-->
    用户名：<input type="text" name="username"><br>
    密码： <input type="password" name="password"><br>
    性别:
    <input type="radio" name="gender" value="男" id="male">
    <label for="male">男</label>
    <input type="radio" name="gender" value="女" id="female">
    <label for="female">女</label><br>
    爱好:
    <input type="checkbox" name="hobby" value='0' id="sing">
    <label for="sing">唱歌</label>
    <input type="checkbox" name="hobby" value='1' id="run">
    <label for="run">跑步</label>
    <input type="checkbox" name="hobby" value='2' id="swim">
    <label for="swim">游泳</label>
    <br>
    <input type="submit" value="提交">
</form>
</body>
</html>
```
在views.py中定义视图函数，这个函数用来处理从/booktest/posttest2/这个url传递过来的post请求
```python
def postTest2(request):
    uname=request.POST['uname']
    upwd=request.POST['upwd']
    ugender=request.POST['ugender']
    uhobby=request.POST.getlist('uhobby') #可以看到post请求也有getlist方法
    context={'uname':uname,'upwd':upwd,'ugender':ugender,'uhobby':uhobby}
    return render(request,'booktest/postTest2.html',context)
```

### HttpResponse对象
- 在django.http模块中定义了HttpResponse对象的API
- HttpRequest对象由Django自动创建，HttpResponse对象由程序员创建
- 可以在视图函数中不调用模板，直接返回数据

1. #### 属性
- content：表示返回的内容，字符串类型
- charset：表示response采用的编码字符集，字符串类型
- status_code：响应的HTTP响应状态码
- content-type：指定输出的MIME类型

2. #### 方法
- init ：使用页内容实例化HttpResponse对象
- write(content)：以文件的方式写
- flush()：以文件的方式输出缓存区
- set_cookie(key, value='', max_age=None, expires=None)：设置Cookie
    - key、value都是字符串类型
    - max_age是一个整数，表示在指定秒数后过期
    - expires是一个datetime或timedelta对象，会话将在这个指定的日期/时间过期，注意datetime和timedelta值只有在使用PickleSerializer时才可序列化
    - max_age与expires二选一
    - 如果不指定过期时间，则两个星期后过期

```python
#cookies练习
def cookietest(request):
    resp = HttpResponse()  #使用HttpResponse构建实例
    cookie = request.COOKIES #使用request去接受cookie
    if 't1' in cookie:
        resp.write(cookie['t1']) #假设在得到的请求中有cookie则在resp中返回cookie的value
    resp.set_cookie('t1', 'thisiscookie', 120) #t1=thisiscookie这个cookie将在120s后过期
    return resp
```

#### 子类HttpResponseRedirect
- 重定向，服务器端跳转
- 构造函数的第一个参数用来指定重定向的地址

```python
在view.py中
from django.shortcuts import render, redirect
from django.http import HttpResponse, HttpResponseRedirect

#这个函数将把从/booktest/redirect1/来的请求重定向到/booktest/redirect2/，即redtest2
def redtest1(request):
    return HttpResponseRedirect('/booktest/redirect2/')

def redtest2(request):
    return HttpResponse('转向来的页面')
```

#### 简写函数 render
- render(request, template_name[, context])
- 结合一个给定的模板和一个给定的上下文字典，并返回一个渲染后的HttpResponse对象
- request：该request用于生成response
- template_name：要使用的模板的完整名称
- context：添加到模板上下文的一个字典，视图将在渲染模板之前调用它
```python
from django.shortcuts import render
def index(request):
    return render(request, 'booktest/index.html', {'h1': 'hello'})
```

### 状态保持
- http协议是无状态的：每次请求都是一次新的请求，不会记得之前通信的状态
- 客户端与服务器端的一次通信，就是一次会话
- 实现状态保持的方式：在客户端或服务器端存储与会话有关的数据
- 存储方式包括cookie、session，会话一般指session对象
- 使用cookie，所有数据存储在客户端，注意不要存储敏感信息
- 推荐使用sesison方式，所有数据存储在服务器端，在客户端cookie中存储session_id
- 状态保持的目的是在一段时间内跟踪请求者的状态，可以实现跨页面访问当前请求者的数据
- 注意：不同的请求者之间不会共享这个数据，与请求者一一对应

#### 启用session
- 使用django-admin startproject创建的项目默认启用
- 在settings.py文件中
```
项INSTALLED_APPS列表中添加：
'django.contrib.sessions',

项MIDDLEWARE_CLASSES列表中添加：
'django.contrib.sessions.middleware.SessionMiddleware',
```

#### 使用session
- 启用会话后，每个HttpRequest对象将具有一个session属性，它是一个类字典对象
- get(key, default=None)：根据键获取会话的值
- clear()：清除所有会话
- flush()：删除当前的会话数据并删除会话的Cookie
- del request.session['member_id']：删除会话
- 一个典型的案例是用户登录的操作，登录完成之后在这个session上跳转回网站主页，用户登录操作效果图如下，具体代码见Ddjangostudying\test3\booktest\views.py
  ![image](https://note.youdao.com/yws/api/personal/file/F27355C6DB6F4BE3A3A75777F4638F92?method=download&shareKey=9bedd06f8f2d47ba88259032857ee4a2)

#### 会话过期时间
- set_expiry(value)：设置会话的超时时间
- 如果没有指定，则两个星期后过期
- 如果value是一个整数，会话将在values秒没有活动后过期
- 若果value是一个imedelta对象，会话将在当前时间加上这个指定的日期/时间过期
- 如果value为0，那么用户会话的Cookie将在用户的浏览器关闭时过期
- 如果value为None，那么会话永不过期
- 修改视图中login_handle函数，查看效果
- 登录过程总结图如下：
  ![image](https://note.youdao.com/yws/api/personal/file/231F981C6EDD4B02A7C9EF13489DAB23?method=download&shareKey=ee3b1f3383c8adff608e51a4192285df)

#### 存储session
- 使用存储会话的方式，可以使用settings.py的SESSION_ENGINE项指定
- 基于数据库的会话：这是django默认的会话存储方式，需要添加django.contrib.sessions到的INSTALLED_APPS设置中，运行manage.py migrate在数据库中安装会话表，可显示指定为
```
SESSION_ENGINE='django.contrib.sessions.backends.db'
```

- 基于缓存的会话：只存在本地内在中，如果丢失则不能找回，比数据库的方式读写更快
```
SESSION_ENGINE='django.contrib.sessions.backends.cache'
```

- 可以将缓存和数据库同时使用：优先从本地缓存中获取，如果没有则从数据库中获取
```
SESSION_ENGINE='django.contrib.sessions.backends.cached_db'
```


