---
title: Django学习笔记--基础知识--sql
date: 2017-01-10 16:05:37
tags: [Python, Django]
categories: Python
description: Django学习笔记--基础知识--sql
---
Django学习笔记--基础知识--sql--Python3.5.x、Django1.10.x
<!--more-->

# 模型（数据库）

Django 模型是与数据库相关的，与数据库相关的代码一般写在 models.py 中，Django 支持 sqlite3, MySQL, PostgreSQL等数据库，只需要在settings.py中配置即可，不用更改models.py中的代码，丰富的API极大的方便了使用。

新建demo_sql项目和新建应用people：

```
django-admin startproject demo_sql # 新建一个项目
cd demo_sql # 进入到该项目的文件夹
django-admin startapp people # 新建一个 people 应用（app)
```

注意：新建app也可以用 python manage.py startapp people, 需要指出的是，django-admin 是安装Django后多出的一个命令，并不是指一个 django-admin.py 脚本在当前目录下。

一个项目一般包含多个应用，一个应用也可以用在多个项目中。

将我们新建的应用（people）添加到 settings.py 中的 INSTALLED_APPS中，也就是告诉Django有这么一个应用。

```

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'people',
]
```

我们打开 people/models.py 文件，修改其中的代码如下：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()
```

新建了一个Person类，继承自models.Model, 一个人有姓名和年龄。

我们来同步一下数据库（我们使用默认的数据库 SQLite3，无需配置）:

```
进入 manage.py 所在的那个文件夹下执行这些命令：
python manage.py makemigrations
python manage.py migrate
```

我们会看到，Django生成了一系列的表，也生成了我们新建的people_person这个表。

Django提供了丰富的API, 下面演示如何使用它。

> python manage.py shell

```
>>> from people.models import Person
>>> Person.objects.create(name="Tom", age=24)
<Person: Person object>
>>>
我们新建了一个用户Tom 那么如何从数据库是查询到它呢？

>>> Person.objects.get(name="Tom")
<Person: Person object>
>>>
```
我们用了一个 .objects.get() 方法查询出来符合条件的对象，但是大家注意到了没有，查询结果中显示<Person: Person object>，这里并没有显示出与Tom的相关信息，如果用户多了就无法知道查询出来的到底是谁，查询结果是否正确，我们重新修改一下 people/models.py

> name 和 age 等字段中不能有 __（双下划线，因为在Django QuerySet API中有特殊含义（用于关系，包含，不区分大小写，以什么开头或结尾，日期的大于小于，正则等）

> 也不能有Python中的关键字，name 是合法的，student_name 也合法，但是student__name不合法，try, class, continue 也不合法，因为它是Python的关键字( import keyword; print(keyword.kwlist) 可以打出所有的关键字)

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()

    def __str__(self)：
        return self.name
```

新建一个对象的方法有以下几种：

```
Person.objects.create(name=name,age=age)
p = Person(name="Tom", age=23)
p.save()
p = Person(name="Joy")
p.age = 23
p.save()
Person.objects.get_or_create(name="Joy", age=23)
这种方法是防止重复很好的方法，但是速度要相对慢些，返回一个元组，第一个为Person对象，第二个为True或False, 新建时返回的是True, 已经存在时返回False.
```

获取对象有以下方法：

```
Person.objects.all()
Person.objects.all()[:10] 切片操作，获取10个人，不支持负索引，切片可以节约内存
Person.objects.get(name=name)

get是用来获取一个对象的，如果需要获取满足条件的一些人，就要用到filter

Person.objects.filter(name="abc") # 等于Person.objects.filter(name__exact="abc") 名称严格等于 "abc" 的人
Person.objects.filter(name__iexact="abc") # 名称为 abc 但是不区分大小写，可以找到 ABC, Abc, aBC，这些都符合条件
Person.objects.filter(name__contains="abc") # 名称中包含 "abc"的人
Person.objects.filter(name__icontains="abc") #名称中包含 "abc"，且abc不区分大小写
Person.objects.filter(name__regex="^abc") # 正则表达式查询
Person.objects.filter(name__iregex="^abc")# 正则表达式不区分大小写

filter是找出满足条件的，当然也有排除符合某条件的

Person.objects.exclude(name__contains="WZ") # 排除包含 WZ 的Person对象
Person.objects.filter(name__contains="abc").exclude(age=23) # 找出名称含有abc, 但是排除年龄是23岁的
```

# 自定义Field(字段)

Django 的官方提供了很多的 Field，但是有时候还是不能满足我们的需求，不过Django提供了自定义 Field 的方法。

1. 减少文本的长度，保存数据的时候压缩，读取的时候解压缩，如果发现压缩后更长，就用原文本直接存储：
Django 1.8 以上版本，可以用：

```python
#coding:utf-8
from django.db import models

class CompressedTextField(models.TextField):
    """
    model Fields for storing text in a compressed format (bz2 by default)
    """

    def from_db_value(self, value, expression, connection, context):
        if not value:
            return value
        try:
            return value.decode('base64').decode('bz2').decode('utf-8')
        except Exception:
            return value

    def to_python(self, value):
        if not value:
            return value
        try:
            return value.decode('base64').decode('bz2').decode('utf-8')
        except Exception:
            return value

    def get_prep_value(self, value):
        if not value:
            return value
        try:
            value.decode('base64')
            return value
        except Exception:
            try:
                return value.encode('utf-8').encode('bz2').encode('base64')
            except Exception:
                return value
```
Django 1.8及以上版本中，from_db_value 函数用于转化数据库中的字符到 Python的变量，get_prep_value 用于将Python变量处理后(此处为压缩）保存到数据库，使用和Django自带的 Field 一样。

# 数据表更改

#### 初始化、更新数据库

Django 1.7.x 及以后的版本集成了 South 的功能，在修改models.py了后运行：

> python manage.py makemigrations
> python manage.py migrate

这两行命令就会对我们的models.py 进行检测，自动发现需要更改的，应用到数据库中去。
你会发现应用people的migrations 目录，里面生成了一个 0001_initial.py 文件。

修改models.py后再运行：

> python manage.py makemigrations
> python manage.py migrate

你会发现应用people的migrations 目录，里面生成了一个 0002_auto_20170111_1446.py 文件。
这里的0001_initial.py和0002_auto_20170111_1446.py是数据库的版本变化记录文件，可以用来恢复到指定版本（0001、0002）。

#### 恢复到以前

South好处就是可以随时恢复到之前的一个版本，比如我们想要回到最开始的那个版本：

> python manage.py migrate people 0001
> “0001”指向的是更新数据库是生成的版本文件号（0001_initial.py）。
> 比如要恢复到0002版本可以执行python manage.py migrate people 0002命令这样就用根据0002_auto_20170111_1446.py的内容来恢复数据库

```
> python manage.py migrate people 0001
Operations to perform:
  Target specific migration: 0001_initial, from people
Running migrations:
  Rendering model states... DONE
  Unapplying people.0002_auto_20170111_1446... OK
```

这样就搞定了，数据库就恢复到以前了，比你手动更改要方便太多了。

# QuerySet API

从数据库中查询出来的结果一般是一个集合，这个集合叫做 QuerySet。

> blog/models.py

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()
 
    def __unicode__(self):  # __str__ on Python 3
        return self.name
 
class Author(models.Model):
    name = models.CharField(max_length=50)
    email = models.EmailField()
 
    def __unicode__(self):  # __str__ on Python 3
        return self.name
 
class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()
 
    def __unicode__(self):  # __str__ on Python 3
        return self.headline
```

#### QuerySet 创建对象的方法

```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()

总之，一共有四种方法
# 方法 1
Author.objects.create(name="Tom", email="Tom@gmail.com")

# 方法 2
tom = Author(name="Tom", email="Tom@gmail.com")
tom.save()

# 方法 3
tom = Author()
tom.name="Tom"
tom.email="Tom@gmail.com"
tom.save()

# 方法 4，首先尝试获取，不存在就创建，可以防止重复
Author.objects.get_or_create(name="Tom", email="Tom@gmail.com")
# 返回值(object, True/False)
```
备注：前三种方法返回的都是对应的 object，最后一种方法返回的是一个元组，(object, True/False)，创建时返回 True, 已经存在时返回 False

当有一对多，多对一，或者多对多的关系的时候，先把相关的对象查询出来

```python
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

#### 获取对象的方法

```python
Person.objects.all() # 查询所有
Person.objects.all()[:10] 切片操作，获取10个人，不支持负索引，切片可以节约内存，不支持负索引，后面有相应解决办法，第7条
Person.objects.get(name="Tom") # 名称为 Tom 的一条，多条会报错
 
get是用来获取一个对象的，如果需要获取满足条件的一些人，就要用到filter
Person.objects.filter(name="abc") # 等于Person.objects.filter(name__exact="abc") 名称严格等于 "abc" 的人
Person.objects.filter(name__iexact="abc") # 名称为 abc 但是不区分大小写，可以找到 ABC, Abc, aBC，这些都符合条件
 
Person.objects.filter(name__contains="abc") # 名称中包含 "abc"的人
Person.objects.filter(name__icontains="abc") #名称中包含 "abc"，且abc不区分大小写
 
Person.objects.filter(name__regex="^abc") # 正则表达式查询
Person.objects.filter(name__iregex="^abc")# 正则表达式不区分大小写
 
# filter是找出满足条件的，当然也有排除符合某条件的
Person.objects.exclude(name__contains="Tom") # 排除包含 Tom 的Person对象
Person.objects.filter(name__contains="abc").exclude(age=23) # 找出名称含有abc, 但是排除年龄是23岁的
```

#### QuerySet 是可迭代的，比如：

```python
es = Entry.objects.all()
for e in es:
    print(e.headline)
Entry.objects.all() 或者 es 就是 QuerySet 是查询所有的 Entry 条目。
```

注意事项：

> (1). 如果只是检查 Entry 中是否有对象，应该用 Entry.objects.all().exists()
> (2). QuerySet 支持切片 Entry.objects.all()[:10] 取出10条，可以节省内存
> (3). 用 len(es) 可以得到Entry的数量，但是推荐用 Entry.objects.count()来查询数量，后者用的是SQL：SELECT COUNT(*)
> (4). list(es) 可以强行将 QuerySet 变成 列表

#### QuerySet 是可以用pickle序列化到硬盘再读取出来的

```python
>>> import pickle
>>> query = pickle.loads(s)     # Assuming 's' is the pickled string.
>>> qs = MyModel.objects.all()
>>> qs.query = query            # Restore the original 'query'.
```

#### QuerySet 查询结果排序

作者按照名称排序

```python
Author.objects.all().order_by('name')
Author.objects.all().order_by('-name') # 在 column name 前加一个负号，可以实现倒序
```

#### QuerySet 支持链式查询

```python
Author.objects.filter(name__contains="Tom").filter(email="tuweizhong@163.com")
Author.objects.filter(name__contains="Wei").exclude(email="tuweizhong@163.com")
# 找出名称含有abc, 但是排除年龄是23岁的
Person.objects.filter(name__contains="abc").exclude(age=23)
```

#### QuerySet 不支持负索引

```python
Person.objects.all()[:10] 切片操作，前10条
Person.objects.all()[-10:] 会报错！！！
# 1. 使用 reverse() 解决
Person.objects.all().reverse()[:2] # 最后两条
Person.objects.all().reverse()[0] # 最后一条
# 2. 使用 order_by，在栏目名（column name）前加一个负号
Author.objects.order_by('-id')[:20] # id最大的20条
```

#### QuerySet 重复的问题，使用 .distinct() 去重

一般的情况下，QuerySet 中不会出来重复的，重复是很罕见的，但是当跨越多张表进行检索后，结果并到一起，可以会出来重复的值

```
qs1 = Pathway.objects.filter(label__name='x')
qs2 = Pathway.objects.filter(reaction__name='A + B >> C')
qs3 = Pathway.objects.filter(inputer__name='Tom')
# 合并到一起
qs = qs1 | qs2 | qs3
这个时候就有可能出现重复的
# 去重方法
qs = qs.distinct()
```

# QuerySet 进阶

准备阶段，可以直接在下面下载代码开始练习，建议看一下 models.py 的代码。

新建一个项目 sql，建一个 app 名称是 blog

```
django-admin startproject sql
cd dql
python manage.py startapp blog
```

把 blog 加入到 settings.py 中的 INSTALL_APPS 中

> blog/models.py

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.db import models
from django.utils.encoding import python_2_unicode_compatible
 
@python_2_unicode_compatible
class Author(models.Model):
    name = models.CharField(max_length=50)
    qq = models.CharField(max_length=10)
    addr = models.TextField()
    email = models.EmailField()
 
    def __str__(self):
        return self.name
 
@python_2_unicode_compatible
class Article(models.Model):
    title = models.CharField(max_length=50)
    author = models.ForeignKey(Author)
    content = models.TextField()
    score = models.IntegerField()  # 文章的打分
    tags = models.ManyToManyField('Tag')
 
    def __str__(self):
        return self.title
 
@python_2_unicode_compatible
class Tag(models.Model):
    name = models.CharField(max_length=50)
 
    def __str__(self):
        return self.name
```

假设一篇文章只有一个作者(Author)，一个作者可以有多篇文章(Article)，一篇文章可以有多个标签（Tag)。

创建 migrations 然后 migrate 在数据库中生成相应的表

```
python manage.py makemigrations
python manage.py migrate
```

生成示例数据，运行 initdb.py:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from __future__ import unicode_literals
import random
from sql.wsgi import *
from blog.models import Author, Article, Tag
 
author_name_list = ['Tom', 'AA', 'BB', 'CC', 'DD']
article_title_list = ['Django 教程', 'Python 教程', 'HTML 教程']
 
def create_authors():
    for author_name in author_name_list:
        author, created = Author.objects.get_or_create(name=author_name)
        # 随机生成9位数的QQ，
        author.qq = ''.join(
            str(random.choice(range(10))) for _ in range(9)
        )
        author.addr = 'addr_%s' % (random.randrange(1, 3))
        author.email = '%s@ziqiangxuetang.com' % (author.addr)
        author.save()
 
def create_articles_and_tags():
    # 随机生成文章
    for article_title in article_title_list:
        # 从文章标题中得到 tag
        tag_name = article_title.split(' ', 1)[0]
        tag, created = Tag.objects.get_or_create(name=tag_name)
 
        random_author = random.choice(Author.objects.all())
 
        for i in range(1, 21):
            title = '%s_%s' % (article_title, i)
            article, created = Article.objects.get_or_create(
                title=title, defaults={
                    'author': random_author,  # 随机分配作者
                    'content': '%s 正文' % title,
                    'score': random.randrange(70, 101),  # 随机给文章一个打分
                }
            )
            article.tags.add(tag)

def main():
    create_authors()
    create_articles_and_tags()

if __name__ == '__main__':
    main()
    print("Done!")
```

执行
> python initdb.py

导入数据后，我们确认一下数据是不是已经导入。

```
>python manage.py shell
In [1]: from blog.models import Article, Author, Tag

In [2]: Article.objects.all()
Out[2]: <QuerySet [<Article: Django 教程_1>, <Article: Django 教程_2>, <Article: Django 教程_3>, <Article: Django 教程_4>, <Article: Django 教程_5>, <Article: Django 教程_6>, <Article: Django 教程_7>, <Article: Django 教程_8>, <Article: Django 教程_9>, <Article: Django 教程_10>, <Article: Django 教程_11>, <Article: Django 教程_12>, <Article: Django 教程_13>, <Article: Django 教程_14>, <Article: Django 教程_15>, <Article: Django 教程_16>, <Article: Django 教程_17>, <Article: Django 教程_18>, <Article: Django 教程_19>, <Article: Django 教程_20>, '...(remaining elements truncated)...']>

In [3]: Author.objects.all()
Out[3]: <QuerySet [<Author: Tom>, <Author: twz915>, <Author: wangdachui>, <Author: xiaoming>]>

In [4]: Tag.objects.all()
Out[4]: <QuerySet [<Tag: Django>, <Tag: Python>, <Tag: HTML>]>

```
准备阶段结束。

### 查看 Django queryset 执行的 SQL

> print str(Author.objects.all().query)
简化一下，就是：SELECT id, name, qq, addr, email FROM blog_author;

> print str(Author.objects.filter(name="Tom").query)
简化一下，就是：SELECT id, name, qq, addr, email FROM blog_author WHERE name=Tom;

所以，当不知道Django做了什么时，你可以把执行的 SQL 打出来看看，也可以借助 django-debug-toolbar 等工具在页面上看到访问当前页面执行了哪些SQL，耗时等。
还有一种办法就是修改一下 log 的设置。

### values_list 获取元组形式结果

#### 比如我们要获取作者的 name 和 qq

```
> authors = Author.objects.values_list('name', 'qq')
> authors
> list(authors)
如果只需要 1 个字段，可以指定 flat=True
> Author.objects.values_list('name', flat=True)
> list(Author.objects.values_list('name', flat=True))

```

#### 查询 AA 这个人的文章标题

`> Article.objects.filter(author__name='AA').values_list('title', flat=True)`

### values 获取字典形式的结果

#### 比如我们要获取作者的 name 和 qq

```
> Author.objects.values('name', 'qq')
> list(Author.objects.values('name', 'qq'))
```

####  查询 twz915 这个人的文章标题

```
> Article.objects.filter(author__name='twz915').values('title')
```

> 注意：
1. values_list 和 values 返回的并不是真正的 列表 或 字典，也是 queryset，他们也是 lazy evaluation 的（惰性评估，通俗地说，就是用的时候才真正的去数据库查）
2. 如果查询后没有使用，在数据库更新后再使用，你发现得到在是新内容！！！如果想要旧内容保持着，数据库更新后不要变，可以 list 一下
3. 如果只是遍历这些结果，没有必要 list 它们转成列表（浪费内存，数据量大的时候要更谨慎！！！）

### extra 实现 别名，条件，排序等

extra 中可实现别名，条件，排序等，后面两个用 filter, exclude 一般都能实现，排序用 order_by 也能实现。我们主要看一下别名这个

比如 Author 中有 name， Tag 中有 name 我们想执行:  
SELECT name AS tag_name FROM blog_tag;  
这样的语句，就可以用 select 来实现，如下：

```
> tags = Tag.objects.all().extra(select={'tag_name': 'name'})
> tags[0].name
> tags[0].tag_name
```

我们发现 name 和 tag_name 都可以使用，确认一下执行的 SQL

```
> Tag.objects.all().extra(select={'tag_name': 'name'}).query.__str__()
输出：u'SELECT (name) AS "tag_name", "blog_tag"."id", "blog_tag"."name" FROM "blog_tag"'
```

我们发现查询的时候弄了两次 (name) AS "tag_name" 和 "blog_tag"."name"  
如果我们只想其中一个能用，可以用 defer 排除掉原来的 name

```
> Tag.objects.all().extra(select={'tag_name': 'name'}).defer('name').query.__str__()
输出： u'SELECT (name) AS "tag_name", "blog_tag"."id" FROM "blog_tag"'
```

也许你会说为什么要改个名称，最常见的需求就是数据转变成 list，然后可视化等。

### annotate 聚合 计数，求和，平均数等

#### 计数

计算每个作者的文章数

```
> Article.objects.all().values('author_id').annotate(count=Count('author')).values('author_id', 'count')
>Article.objects.all().values('author_id').annotate(count=Count('author')).values('author_id','count').query.__str__()
输出：u'SELECT "blog_article"."author_id", COUNT("blog_article"."author_id") AS "count" FROM "blog_article" GROUP BY "blog_article"."author_id"'
简化一下SQL: SELECT author_id, COUNT(author_id) AS count FROM blog_article GROUP BY author_id
```
获取作者的名称 及 作者的文章数

> Article.objects.all().values('author__name').annotate(count=Count('author')).values('author__name', 'count')

#### 求和 与 平均值

求一个作者的所有文章的得分(score)平均值

```
> from django.db.models import Avg
> Article.objects.values('author_id').annotate(avg_score=Avg('score')).values('author_id', 'avg_score')
执行的SQL
> Article.objects.values('author_id').annotate(avg_score=Avg('score')).values('author_id', 'avg_score').query.__str__()
> 输出：u'SELECT "blog_article"."author_id", AVG("blog_article"."score") AS "avg_score" FROM "blog_article" GROUP BY "blog_article"."author_id"'
```

求一个作者所有文章的总分

```
> from django.db.models import Sum
> Article.objects.values('author__name').annotate(sum_score=Sum('score')).values('author__name', 'sum_score')
执行的SQL：
> Article.objects.values('author__name').annotate(sum_score=Sum('score')).values('author__name', 'sum_score').query.__str__()
输出：u'SELECT "blog_author"."name", SUM("blog_article"."score") AS "sum_score" FROM "blog_article" INNER JOIN "blog_author" ON ("blog_article"."author_id" = "blog_author"."id") GROUP BY "blog_author"."name"'
```

### select_related 做化一对一，多对一查询

开始之前我们修改一个 settings.py 让Django打印出在数据库中执行的语句

settings.py 尾部加上
```

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG' if DEBUG else 'INFO',
        },
    },
}
```

这样当 DEBUG 为 True 的时候，我们可以看出 django 执行了什么 SQL 语句

```
> python manage.py shell
> from blog.models import *
> Author.objects.all()
```

假如，我们取出10篇Django相关的文章，并需要用到作者的姓名:

```
> articles = Article.objects.all()[:10]
> a1 = articles[0]  # 取第一篇
> a1.title
> a1.author_id
> a1.author.name   # 再次查询了数据库，注意！！！
```

这样的话我们遍历查询结果的时候就会查询很多次数据库，能不能只查询一次，把作者的信息也查出来呢？
当然可以，这时就用到 select_related，我们的数据库设计的是一篇文章只能有一个作者，一个作者可以有多篇文章。
现在要查询文章的时候连同作者一起查询出来，“文章”和“作者”的关系就是多对一，换句说说，就是一篇文章只可能有一个作者。

```
> articles = Article.objects.all().select_related('author')[:10]
> a1 = articles[0]  # 取第一篇
> a1.title
> a1.author.name   # 没有再次查询数据库！！
```

### prefetch_related 做化一对多，多对多查询

和 select_related 功能类似，但是实现不同。  
select_related 是使用 SQL JOIN 一次性取出相关的内容。  
prefetch_related 用于 一对多，多对多 的情况，这时 select_related 用不了，因为当前一条有好几条与之相关的内容。
prefetch_related是通过再执行一条额外的SQL语句，然后用 Python 把两次SQL查询的内容关联（joining)到一起

看个例子，查询文章的同时，查询文章对应的标签。“文章”与“标签”是多对多的关系。

```
> articles = Article.objects.all().prefetch_related('tags')[:10]
> articles
```

遍历查询的结果：

不用 prefetch_related 时

> articles = Article.objects.all()[:3]
> for a in articles:
        print a.title, a.tags.all()

用 prefetch_related 我们看一下是什么样子

```
> articles = Article.objects.all().prefetch_related('tags')[:3]
> for a in articles:
       print a.title, a.tags.all()

```

第二条 SQL 语句，一次性查出了所有相关的内容。


### defer 排除不需要的字段

在复杂的情况下，表中可能有些字段内容非常多，取出来转化成 Python 对象会占用大量的资源。

这时候可以用 defer 来排除这些字段，比如我们在文章列表页，只需要文章的标题和作者，没有必要把文章的内容也获取出来（因为会转换成python对象，浪费内存）

```
> Article.objects.all()
> Article.objects.all().defer('content')
```

### only 仅选择需要的字段

和 defer 相反，only 用于取出需要的字段，假如我们只需要查出作者的名称

> Author.objects.all().only('name')

可以发现，我们让查 name ， id 也查了，这个 id 是 主键，能不能没有这个 id 呢？

试一下原生的 SQL 查询

```
> authors =  Author.objects.raw('select name from blog_author limit 1')
> author = authors[0]
InvalidQuery: Raw query must include the primary key
报错信息说 非法查询，原生SQL查询必须包含 主键！
```

再试试直接执行 SQL

```
> python manage.py dbshell
sqlite> select name from blog_author limit 1;
Tom       <---  成功！！！
```

虽然直接执行SQL语句可以这样，但是 django queryset 不允许这样做，一般也不需要关心，反正 only 一定会取出你指定了的字段。

### 自定义聚合功能

我们前面看到了 django.db.models 中有 Count, Avg, Sum 等，但是有一些没有的，比如 GROUP_CONCAT，它用来聚合时将符合某分组条件(group by)的不同的值，连到一起，作为整体返回。

实现 GROUP_CONCAT 功能:

新建一个文件my_aggregate.py:

```python
from django.db.models import Aggregate, CharField
 
class GroupConcat(Aggregate):
    function = 'GROUP_CONCAT'
    template = '%(function)s(%(distinct)s%(expressions)s%(ordering)s%(separator)s)'
 
    def __init__(self, expression, distinct=False, ordering=None, separator=',', **extra):
        super(GroupConcat, self).__init__(
            expression,
            distinct='DISTINCT ' if distinct else '',
            ordering=' ORDER BY %s' % ordering if ordering is not None else '',
            separator=' SEPARATOR "%s"' % separator,
            output_field=CharField(),
            **extra        )
```

使用时先引入 GroupConcat 这个类，比如聚合后的错误日志记录有这些字段 time, level, info

我们想把 level, info 一样的 聚到到一起，按时间和发生次数倒序排列，并含有每次日志发生的时间。

```
ErrorLogModel.objects.values('level', 'info').annotate(count=Count(1), time=GroupConcat('time', ordering='time DESC', separator=' | ')).order_by('-time', '-count')
```
