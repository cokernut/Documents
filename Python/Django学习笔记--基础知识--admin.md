---
title: Django学习笔记--基础知识--admin
date: 2017-01-12 10:03:44
tags: [Python, Django]
categories: Python
description: Django学习笔记--基础知识--admin
---
Django学习笔记--基础知识--admin--Python3.5.x、Django1.10.x
<!--more-->

# 后台

### 新建一个 名称为 django_admin 的项目

> django-admin startproject django_admin

### 新建一个 叫做 blog 的app

> cd django_admin
> python manage.py startapp blog

注意：不同版本的 Django 创建 project 和 app 出来的文件会有一些不同

### 修改 blog 文件夹中的 models.py

```python
# coding:utf-8
from django.db import models

class Article(models.Model):
    title = models.CharField(u'标题', max_length=256)
    content = models.TextField(u'内容')
    pub_date = models.DateTimeField(u'发表时间', auto_now_add=True, editable = True)
    update_time = models.DateTimeField(u'更新时间',auto_now=True, null=True)
```

### 把 blog 加入到settings.py中的INSTALLED_APPS中

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
]
```

### 同步所有的数据表

进入包含有 manage.py 的文件夹执行：

> python manage.py makemigrations
> python manage.py migrate

如果 Django 不主动提示创建管理员,可以用下面的命令创建一个管理员帐号：

> python manage.py createsuperuser

### 修改 admin.py

进入 blog 文件夹，修改 admin.py 文件（如果没有新建一个），内容如下:

```python
from django.contrib import admin
from .models import Article

admin.site.register(Article)
```
只需要这三行代码，我们就可以拥有一个强大的后台！
提示：urls.py中关于 admin的已经默认开启。

### 打开开发服务器

> python manage.py runserver
> 如果提示 8000 端口已经被占用，可以用 python manage.py runserver 8001 以此类推

访问 http://localhost:8000/admin/ 输入设定的帐号和密码, 就可以看到管理员界面。

点击 Articles， 添加几篇文章，就可以看到新添加的文章列表。
这时我们会发现所有的文章都是叫 Article object，这样肯定不好，我们修改一下 blog 中的models.py：

```python
# coding:utf-8
from django.db import models

class Article(models.Model):
    title = models.CharField(u'标题', max_length=256)
    content = models.TextField(u'内容')
 
    pub_date = models.DateTimeField(u'发表时间', auto_now_add=True, editable = True)
    update_time = models.DateTimeField(u'更新时间',auto_now=True, null=True)
 
    def  __str__ (self):
        return self.title
```

加了一个 __str__  函数，刷新后台网页，会看到文章列表显示的已经是文章的标题了。
所以建议定义 Model 的时候 写一个 __str__函数)。

兼容python2.x和python3.x：
models.py：

```python
# coding:utf-8
from __future__ import unicode_literals
from django.db import models
from django.utils.encoding import python_2_unicode_compatible

@python_2_unicode_compatible
class Article(models.Model):
    title = models.CharField('标题', max_length=256)
    content = models.TextField('内容')
    pub_date = models.DateTimeField('发表时间', auto_now_add=True, editable = True)
    update_time = models.DateTimeField('更新时间',auto_now=True, null=True)

    def __str__(self):
        return self.title
```

python_2_unicode_compatible 会自动做一些处理去适应python不同的版本，这里的 unicode_literals 可以让python2.x 也像 python3 那个处理 unicode 字符，以便有更好地兼容性。

### 在列表显示与字段相关的其它内容

如果我们还需要显示一些其它的fields，如何做呢？

> blog/admin.py

```python
from django.contrib import admin
from .models import Article

class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title','pub_date','update_time',)

admin.site.register(Article,ArticleAdmin)
```

list_display 就是来配置要显示的字段的，当然也可以显示非字段内容，或者字段相关的内容，比如：

> blog/models.py

```python
class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def my_property(self):
        return self.first_name + ' ' + self.last_name
    my_property.short_description = "Full name of the person"

    full_name = property(my_property)
```

> blog/admin.py

```python
from django.contrib import admin
from .models import Article, Person

class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'pub_date', 'update_time',)

class PersonAdmin(admin.ModelAdmin):
    list_display = ('full_name',)

admin.site.register(Article, ArticleAdmin)
admin.site.register(Person, PersonAdmin)
```

到这里我们发现我们又有新的需求，比如要改 models.py 中的字段，添加一个文章的状态（草稿，正式发布），这时候我们就需要更改表，django 1.7以前的都不会自动更改表，我们需要用第三方插件 South，Django 1.7 及以上用以下命令来同步数据库表的更改：

> python manage.py makemigrations
> python manage.py migrate

#### 其它一些常用的功能：

搜索功能：search_fields = ('title', 'content',) 这样就可以按照 标题或内容搜索了
https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields

筛选功能：list_filter = ('status',) 这样就可以根据文章的状态去筛选，比如找出是草稿的文章
https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_filter

新增或修改时的布局顺序：https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.fieldsets

有时候我们需要对django admin site进行修改以满足自己的需求：
举例说明：
1.定制加载的列表, 根据不同的人显示不同的内容列表，比如输入员只能看见自己输入的，审核员能看到所有的草稿，这时候就需要重写get_queryset方法

```python
class MyModelAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super(MyModelAdmin, self).get_queryset(request)
        if request.user.is_superuser:
            return qs
        else:
            return qs.filter(author=request.user)
```

该类实现的功能是如果是超级管理员就列出所有的，如果不是，就仅列出访问者自己相关的。

2.定制搜索功能（django 1.6及以上才有)

```python
class PersonAdmin(admin.ModelAdmin):
    list_display = ('name', 'age')
    search_fields = ('name',)

    def get_search_results(self, request, queryset, search_term):
        queryset, use_distinct = super(PersonAdmin, self).get_search_results(request, queryset, search_term)
        try:
            search_term_as_int = int(search_term)
            queryset |= self.model.objects.filter(age=search_term_as_int)
        except:
            pass
        return queryset, use_distinct
```

queryset 是默认的结果，search_term 是在后台搜索的关键词

3.修改保存时的一些操作，可以检查用户，保存的内容等，比如保存时加上添加人

```python
from django.contrib import admin

class ArticleAdmin(admin.ModelAdmin):
    def save_model(self, request, obj, form, change):
        obj.user = request.user
        obj.save()
```

其中obj是修改后的对象，form是返回的表单（修改后的），当新建一个对象时 change = False, 当修改一个对象时 change = True
如果需要获取修改前的对象的内容可以用

```python
from django.contrib import admin

class ArticleAdmin(admin.ModelAdmin):
    def save_model(self, request, obj, form, change):
        obj_original = self.model.objects.get(pk=obj.pk)
        obj.user = request.user
        obj.save()
```

那么又有问题了，这里如果原来的obj不存在，也就是如果我们是新建的一个怎么办呢，这时候可以用try,except的方法尝试获取,当然更好的方法是判断一下这个对象是新建还是修改，是新建就没有 obj_original，是修改就有

```python
from django.contrib import admin

class ArticleAdmin(admin.ModelAdmin):
    def save_model(self, request, obj, form, change):
        if change:# 更改的时候
            obj_original = self.model.objects.get(pk=obj.pk)
        else:# 新增的时候
            obj_original = None

        obj.user = request.user
        obj.save()
```

4, 删除时做一些处理,

```python
from django.contrib import admin

class ArticleAdmin(admin.ModelAdmin):
    def delete_model(self, request, obj):
        """
        Given a model instance delete it from the database.
        """
        # handle something here
        obj.delete()
```