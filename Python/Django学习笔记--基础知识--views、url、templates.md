---
title: Django学习笔记--基础知识--views、url、templates
date: 2017-01-10 10:40:43
tags: [Python, Django]
categories: Python
description: Django学习笔记--基础知识--views、url、templates
---
Django学习笔记--基础知识--views、url、templates--Python3.5.x、Django1.10.x
<!--more-->

# 基本命令

### 新建一个 django项目

> django-admin.py startproject project-name

project-name 项目名称，要符合Python 的变量命名规则（以下划线或字母开头）

### 新建 app

> python manage.py startapp app-name
或 django-admin.py startapp app-name

一般一个项目有多个app, 当然通用的app也可以在多个项目中使用。

### 同步数据库

> python manage.py syncdb
注意：Django 1.7.1及以上的版本需要用以下命令
python manage.py makemigrations
python manage.py migrate

这种方法可以创建表，当你在models.py中新增了类时，运行它就可以自动在数据库中创建表了，不用手动创建。

备注：对已有的 models 进行修改，Django 1.7之前的版本的Django都是无法自动更改表结构的，不过有第三方工具 south,详见 Django 数据库迁移 一节。

### 使用开发服务器

开发服务器，即开发时使用，一般修改代码后会自动重启，方便调试和开发，但是由于性能问题，建议只用来测试，不要用在生产环境。

> python manage.py runserver

```
# 当提示端口被占用的时候，可以用其它端口：
python manage.py runserver 8001
python manage.py runserver 9999
（当然也可以kill掉占用端口的进程）

# 监听所有可用 ip （电脑可能有一个或多个内网ip，一个或多个外网ip，即有多个ip地址）
python manage.py runserver 0.0.0.0:8000
# 如果是外网或者局域网电脑上可以用其它电脑查看开发服务器
# 访问对应的 ip加端口，比如 http://172.16.20.2:8000
```

### 清空数据库

> python manage.py flush

此命令会询问是 yes 还是 no, 选择 yes 会把数据全部清空掉，只留下空表。

### 创建超级管理员

```
python manage.py createsuperuser
# 按照提示输入用户名和对应的密码就好了邮箱可以留空，用户名和密码必填
# 修改 用户密码可以用：
python manage.py changepassword username
```

### 导出数据、导入数据

> python manage.py dumpdata appname > appname.json
> python manage.py loaddata appname.json

关于数据操作 详见：数据导入数据迁移，现在了解有这个用法就可以了。

### Django 项目环境终端

> python manage.py shell

如果你安装了 bpython 或 ipython 会自动用它们的界面，推荐安装 bpython。

这个命令和 直接运行 python 或 bpython 进入 shell 的区别是：你可以在这个 shell 里面调用当前项目的 models.py 中的 API，对于操作数据，还有一些小测试非常方便。

### 数据库命令行

> python manage.py dbshell

Django 会自动进入在settings.py中设置的数据库(默认是SQLite)，如果是 MySQL 或 postgreSQL,会要求输入数据库用户密码。
在这个终端可以执行数据库的SQL语句。如果您对SQL比较熟悉，可能喜欢这种方式。

### 更多命令

终端上输入 python manage.py 可以看到详细的列表，在忘记子名称的时候特别有用。

# 视图与网址

### 新建项目，新建app，创建数据库表

> django-admin startproject demo
> python manage.py startapp app
> python manage.py migrate

目录结构为（Django1.10.x）：
```
demo
├── app
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── templates
│   │   └── home.html
│   ├── admin.py
│   ├── models.py
│   ├── apps.py
│   ├── tests.py
│   └── views.py
├── manage.py
├── db.sqlite3
└── demo
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
注：Django 1.8.x 以上的，才有 migrations 文件夹。Django 1.9.x 会在 Django 1.8 的基础上多出一个 apps.py 文件。

### 把我们新定义的app加到settings.py中的INSTALL_APPS中：

```python
# 新建的 app 如果不加到 INSTALL_APPS 中的话, django 就不能自动找到app中的模板文件(app-name/templates/下的文件)和静态文件(app-name/static/中的文件)
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app',
]
```

### 定义视图函数（访问页面时的内容）

我们在app这个目录中,把views.py打开,修改其中的源代码,改成下面的

```python
#coding:utf-8
from django.http import HttpResponse

def index(request):
    return HttpResponse(u"欢迎光临!")
```

第一行是声明编码为utf-8, 因为我们在代码中用到了中文,如果不声明就报错.  
第二行引入HttpResponse，它是用来向网页返回内容的，就像Python中的 print 一样，只不过 HttpResponse 是把内容显示到网页上。  
我们定义了一个index()函数，第一个参数必须是 request，与网页发来的请求有关，request 变量里面包含get或post的内容，用户浏览器，系统等信息在里面。  
函数返回了一个 HttpResponse 对象，可以经过一些处理，最终显示几个字到网页上。  

### 定义视图函数相关的URL(网址)  （即规定 访问什么网址对应什么内容）

我们打开 demo/demo/urls.py 这个文件, 修改其中的代码:
Django 1.8.x及以上，Django 官方鼓励（或说要求）先引入，再使用：

```python
from django.conf.urls import url
from django.contrib import admin
from app import views as app_views

urlpatterns = [
    url(r'^$', app_views.index, name='index'),
    url(r'^admin/', admin.site.urls),
]
```

### 在终端上运行 python manage.py runserver 开启服务器

我们打开浏览器,访问 http://127.0.0.1:8000/ 即可看到网页中显示的“欢迎光临!”。

注意：如果是在另一台电脑上访问要用 python manage.py ip:port 的形式，比如监听所有ip:

```language
python manage.py runserver 0.0.0.0:8000
监听机器上所有ip 8000端口，访问时用电脑的ip代替 127.0.0.1
```

### 采用 /add/?a=4&b=5 这样GET方法进行加法运算

修改 app/views.py文件，添加add方法：

```python
def add(request):
    a = request.GET['a']
    b = request.GET['b']
    c = int(a)+int(b)
    return HttpResponse(str(c))
```

打开 demo/demo/urls.py 这个文件, 修改其中的代码:

```python
from django.conf.urls import url
from django.contrib import admin
from app import views as app_views

urlpatterns = [
    url(r'^add/$', app_views.add, name='add'),
  	url(r'^$', app_views.index, name='index'),
    url(r'^admin/', admin.site.urls),
]
```

然后我们访问 http://127.0.0.1:8000/add/?a=4&b=5
就可以看到网页上显示运算结果 9。

### 采用 /add/3/4/ 这样的网址的方式进行加法运算

修改 app/views.py文件，添加add2方法：

```python
def add2(request, a, b):
    c = int(a) + int(b)
    return HttpResponse(str(c))
```

打开 demo/demo/urls.py 这个文件, 添加一个url映射:

```python
# (\d+), 正则表达式中 \d 代表一个数字，+ 代表一个或多个前面的字符，写在一起 \d+ 就是一个或多个数字，用括号括起来的意思是保存为一个子组，每一个子组将作为一个参数，被 views.py 中的对应视图函数接收。
    url(r'^add/(\d+)/(\d+)/$', app_views.add2, name='add2'),
```

然后我们访问 http://127.0.0.1:8000/add/4/5/
就可以看到网页上显示运算结果 9。

### reverse

> reverse('add2', args=(4,5))
> 得到 u'/add/4/5/'

reverse 接收 url 中的 name 作为第一个参数，我们在代码中就可以通过 reverse() 来获取对应的网址（这个网址可以用来跳转，也可以用来计算相关页面的地址，可以用在 views.py，models.py等各种需要转换得到网址的地方），只要对应的 url 的name不改，就不用改代码中的网址。

### 网页模板中也是一样，可以很方便的使用

```html
不带参数的：
{% url 'name' %}
带参数的：参数可以是变量名
{% url 'name' 参数 %}

例如：
<a href="{% url 'add2' 4 5 %}">link</a>
上面的代码渲染成最终的页面是
<a href="/add/4/5/">link</a>
这样就可以通过 {% url 'add2' 4 5 %} 获取到对应的网址 /add/4/5/
```

当 urls.py 进行更改，前提是不改 name（这个参数设定好后不要轻易改），获取的网址也会动态地跟着变，比如改成：

```
url(r'^new_add/(\d+)/(\d+)/$', app_views.add2, name='add2'),
注意add 变成了 new_add，但是后面的 name='add2' 没改，这时 {% url 'add2' 4 5 %}就会渲染对应的网址成 /new_add/4/5/
```
reverse 函数也是一样，获取的时候会跟着变成新的网址，这样，在想改网址时只需要改 urls.py 中的正则表达式（url 参数第一部分），其它地方都“自动”跟着变了。

另外，比如用户收藏夹中收藏的URL是旧的，如何让以前的 /old_add/3/4/自动跳转到现在新的网址呢？
要知道Django不会帮你做这个，这个需要自己来写一个跳转方法：
修改 app/views.py文件，添加add3方法：

```python
# 有 /old_add/4/5/ ，访问时就会自动跳转到新的 /add/4/5/
def add3(request, a, b):
    return HttpResponseRedirect(reverse('add2', args=(a, b)))
```

打开 demo/demo/urls.py 这个文件, 添加一个url映射:

```python
 # 有 /old_add/4/5/ ，访问时就会自动跳转到新的 /add/4/5/
    url(r'^old_add/(\d+)/(\d+)/$', app_views.add3),
```

然后我们访问 http://127.0.0.1:8000/old_add/4/5/ 时会自动跳转到 http://127.0.0.1:8000/add/4/5/
就可以看到网页上显示运算结果 9。

# 模板

### 显示一个基本的字符串在网页上、基本的 for 循环 和 List内容的显示、显示字典中内容、条件判断和 for 循环的详细操作

> views.py

```python
#coding:utf-8
from django.shortcuts import render

def home(request):
    # 一般的变量之类的用 {{ }}（变量），功能类的，比如循环，条件判断是用 {%  %}（标签）
    # {{string}}
    string = u"学习Django，用它来建网站"
    # {% for i in TutorialList %} {{ i }} {% endfor %}
    Tlist = ["HTML", "CSS", "jQuery", "Python", "Django"]
    # 站点：{{ info_dict.site }} 内容：{{ info_dict.content }}
    # {% for key, value in info_dict.items %} {{ key }}: {{ value }} {% endfor %
    dict = {'title': u'标题', 'content': u'内容'}
    # {% for item in List %} {{ item}}{% if not forloop.last %},{% endif %} {% endfor %}
    # 变量 forloop.last 这个变量，如果是最后一项其为真，否则为假
    List = map(str, range(10))# 一个长度为100的 List
    return render(request, 'home.html', {'info_dict': dict, 'string': string, 'TutorialList': Tlist, 'List': List})
```

> home.html

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Django</title>
    </head>
    <body>
        {{string}}<br> 
        教程列表： {% for i in TutorialList %} {{ i }} {% endfor %}<br> 
        标题：{{ info_dict.title }} 内容：{{ info_dict.content}}<br> 
        {% for key, value in info_dict.items %} {{ key }}: {{ value }} {% endfor %} <br> 
        {% for item in List %} {{ item}}{% if not forloop.last %},{% endif %}  {% endfor %} <br>
    </body>
</html>
```

### 在for循环中还有很多有用的方法，如下：

| 	变量 	| 	描述	 |
|--------|--------|
| forloop.counter |	索引从 1 开始算 |
| forloop.counter0 |	索引从 0 开始算 |
| forloop.revcounter |	索引从最大长度到 1|
| forloop.revcounter0	| 索引从最大长度到 0 |
| forloop.first | 当遍历的元素为第一项时为真 |
| forloop.last | 当遍历的元素为最后一项时为真 |
| forloop.parentloop | 用在嵌套的 for 循环中，获取上一层 for 循环的 forloop |

当列表中可能为空值时用 for  empty

```html
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% empty %}
    <li>抱歉，列表为空</li>
{% endfor %}
</ul>
```

### 模板上得到视图对应的网址

修改 app/views.py文件，添加add方法：

```python
def add(request, a, b):
    c = int(a) + int(b)
    return HttpResponse(str(c))
```

打开 demo/demo/urls.py 这个文件, 添加一个url映射:

```python
url(r'^add/(\d+)/(\d+)/$', app_views.add, name='add'),
```

在html中使用：
可以使用 as 语句将内容取别名（相当于定义一个变量），多次使用（但视图名称到网址转换只进行了一次）.

```html
{% url 'add' 4 5 as the_add %}
<a href="{{ the_add }}">计算4+5</a>
```

### 模板中的逻辑操作

#### ==, !=, >=, <=, >, < 这些比较都可以在模板中使用，比如：

```html
{% if var >= 90 %}
成绩优秀
{% elif var >= 80 %}
成绩良好
{% elif var >= 70 %}
成绩一般
{% elif var >= 60 %}
需要努力
{% else %}
不及格
{% endif %}
```

#### and, or, not, in, not in 也可以在模板中使用

假如我们判断 num 是不是在 0 到 100 之间：

```html
{% if num <= 100 and num >= 0 %}
num在0到100之间
{% else %}
数值不在范围之内！
{% endif %}
```

假如我们判断 'string' 在不在一个列表变量 List 中：

```html
{% if 'string' in List %}
存在
{% endif %}
```

### 模板中 获取当前网址，当前用户等

如果不是在 views.py 中用的 render 函数，是 render_to_response 的话，需要将 request 加入到 上下文渲染器。
Django 1.8 及以后 修改 settings.py 

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                ...
                'django.template.context_processors.request',
                ...
            ],
        },
    },
]
```

然后在 模板中我们就可以用 request 了。一般情况下，推荐用 render 而不是用 render_to_response。

#### 获取当前用户

```
{{ request.user }}
```

如果登陆就显示内容，不登陆就不显示内容：

```
{% if request.user.is_authenticated %}
    {{ request.user.username }}，您好！
{% else %}
    请登陆，这里放登陆链接
{% endif %}
```

#### 获取当前网址

```
{{ request.path }}
```

#### 获取当前 GET 参数

```
{{ request.GET.urlencode }}
```

#### 合并到一起用的一个例子

```
<a href="{{ request.path }}?{{ request.GET.urlencode }}&delete=1">当前网址加参数 delete</a>
```

比如我们可以判断 delete 参数是不是 1 来删除当前的页面内容。
