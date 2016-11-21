Title: 使用Pelican+Github Pages搭建静态博客
Date: 2016-11-16
Modified: 2016-11-16
Tags: Pelican,Github Pages
Slug: Pelican_Github_Pages_Create_Blog
Authors: Cokernut
Summary: 使用Pelican+Github Pages搭建静态博客

## Pelican介绍

什么是Pelican

Perlican 是用Python实现的一个静态网站生成器，支持 reStructuredText 或 Markdown 。它支持以下功能：

+ 博客文章和静态网页
+ 支持评论。评论是通过第三方服务 Disqus 支持的。即评论数据保存在第三方服务器上
+ 主题支持
+ 把博客文章生成PDF格式文档
+ 多语言博客支持，如可以用英文和中文写同一篇博客。不同语言访问者访问相应语言的博文
+ 支持Atom/RSS订阅
+ 博文中代码高亮支持
+ 博客搬家支持(WordPress, Dotclear, 或RSS feeds)
+ 支持插件，如Twiter, Google Analytics等

## 为什么选择Pelican

首先排除掉WordPress之类的CMS系统。因为我不想要数据库，我只需要一个轻量级的静态网站生成器。我的博客使用Markdown编写，且保存在GitHub上。我想要的，只是用Markdown写完博客之后，git commit + git push即可直接发布到博客网站上。

选择Pelican是基于如下原因：
使用Python实现。由于最近在学习Python，我可以阅读源码并按照我的需求来改造Pelican使之完全符合我的需求。下次学习Ruby，用 jekyll 再折腾一遍。因为Jekyll是用Ruby实现的。且GitHub Pages的后台就是用Jekyll，到时可直接用GitHub Pages实现个人博客。
足够轻量级。总的代码量才1MB多。安装也方便。
有一堆现成的主题可以使用。这对我这种非专业前端的开发者来说，省了不少事。
文档齐全。
开发活动活跃。GitHub上代码提交活跃。上面文章里介绍的很多博客系统基本上都2+年前就停止更新了。
最后两点对使用任何开源工具来说都是很重要的，只有开发活跃，社区资源多，文档齐全，遇到问题的时候才能较快地得到解决。

## Pelican安装与配置

安装Pelican并创建项目

详细的信息可以参阅 Pelican官方文档 。假设电脑上已经安装Python和pip。首先，通过pip安装pelican和markdown：

pip install pelican markdown
然后创建你的博客项目：

mkdir ~/blogs
cd ~/blogs
pelican-quickstart
在运行pelican-quickstart时，系统会问一系列问题，比如你的博客网址啊，作者名字啊之类的，根据真实情况填写即可，这些问题只是用来生成配置文件的，我们后面都可以通过修改配置文件来手动修改这些设置。我填写的内容如下：
```
Cokernut@Cokernut-laptop:~/lab/blogs$ pelican-quickstart
Welcome to pelican-quickstart v3.4.0.

This script will help you create a new Pelican-based website.

Please answer the following questions so this script can generate the files
needed by Pelican.

> Where do you want to create your new web site? [.]
> What will be the title of this web site? Cokernut
> Who will be the author of this web site? Cokernut
> What will be the default language of this web site? [en]
> Do you want to specify a URL prefix? e.g., http://example.com   (Y/n)
> What is your URL prefix? (see above example; no trailing slash) http://cokernut.top
> Do you want to enable article pagination? (Y/n) Y
> How many articles per page do you want? [10]
> Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n) Y
> Do you want an auto-reload & simpleHTTP script to assist with theme and site development? (Y/n)
> Do you want to upload your website using FTP? (y/N) N
> Do you want to upload your website using SSH? (y/N) y
> What is the hostname of your SSH server? [localhost]/cokernut.top
> What is the port of your SSH server? [22]
> What is your username on that server? [root] file
> Where do you want to put your web site on that server? [/var/www] /home/file/blogs
> Do you want to upload your website using Dropbox? (y/N) N
> Do you want to upload your website using S3? (y/N) N
> Do you want to upload your website using Rackspace Cloud Files? (y/N) N
> Do you want to upload your website using GitHub Pages? (y/N) N
Done. Your new project is available at /home/Cokernut/blogs
```
其中第14行的 http://cokernut.top 以及第21行的cokernut.top 是我的域名，如果你只是在本机试验，可以填localhost。创建完项目后，目录下看起来象这样。
```
Cokernut@Cokernut-laptop:~/lab/blogs$ tree
.
├── content     # 这个就是放博客内容目录，这个目录及子目录下的所有md和rst文件将会被转成html文件
├── develop_server.sh   #这个是用来在本地运行一个服务器来实时查看生成的html文档的脚本
├── fabfile.py  # 这个是使用Python的fabric来实现文件上传的工具，即Deploy工具
├── Makefile    # 这个是使用是用来生成网站内容并上传的工具。后文详细解释
├── output      # 这个是从content目录生成的html目标文件的存放目录
├── pelicanconf.py      # 这个是本地开发时的配置文件
└── publishconf.py      # 这个是发布时的配置文件
```
2 directories, 5 files
配置pelicanconf.py和publishconf.py

Pelican的配置文件是直接用Python写的，我本地开发配置文件 pelicanconf.py 内容如下：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #
from __future__ import unicode_literals
AUTHOR = u'Cokernut'
SITENAME = u"cokernut.top"
SITEURL = 'http://localhost'
DISQUS_SITENAME = 'Cokernut'
PATH = 'content'
TIMEZONE = 'Asia/Shanghai'
DEFAULT_LANG = u'zh_CN'
DEFAULT_DATE_FORMAT = ('%Y-%m-%d(%A) %H:%M')
USE_FOLDER_AS_CATEGORY = True
DEFAULT_CATEGORY = 'hide'
# Feed generation is usually not desired when developing
FEED_ATOM = 'feeds/atom.xml'
FEED_RSS = 'feeds/rss.xml'
FEED_ALL_ATOM = None
FEED_ALL_RSS = None
CATEGORY_FEED_ATOM = None
TRANSLATION_FEED_ATOM = None
# menu items
MENUITEMS = [('Home', SITEURL),
      ('About', 'about.html'),]
DEFAULT_PAGINATION = 10
MD_EXTENSIONS = [
  "extra",
  "toc",
  "headerid",
  "meta",
  "sane_lists",
  "smarty",
  "wikilinks",
  "admonition",
  "codehilite(guess_lang=False,pygments_style=emacs,noclasses=True)"]
CNZZ_ANALYTICS = True
MONTH_ARCHIVE_SAVE_AS = 'posts/{date:%Y}/{date:%m}/index.html'
THEME = "themes/foundation-default-colours"
```
第6行的SITENAME是博客网站的名称，可以是任何字符；第7行是博客网站的网址，这个字段在本地开发和发布版本是不一样的，本地直接填localhost即可，发布版本里需要填博客网址。  
第8行：我使用了Disqus作为我的评论系统，Disqus也是 YC 毕业生。启用Disqus评论系统非常简单，在官网上注册一个Disqus帐户，然后把帐户名填在 DISQUS_SITENAME 值里即可启用。我的Disqus帐号刚好也是 Cokernut 。  
第33－42行：这里是配置Markdown扩展，用来支持代码高亮。并且使用Emacs风格的代码高亮。  
第44行：由于GFW的存在，我把Google Analize换成了国内的CNZZ统计。  
第46行：我的博客使用了 foundation-default-colours 这套主题。关于主题，后文详解。  
开发环境和发布环境的配置差不多，除SITEURL不一样外，还多了两个配置：  
```
SITEURL = 'http://cokernut.top'
# usful setting for publish
RELATIVE_URLS = False   # 禁用相对路径引用
DELETE_OUTPUT_DIRECTORY = True      # 编译之前删除output目录，这样保证output下生成的内容是干净的
其它的配置项，可以参阅 Pelican设置文档 。
```
### 配置Makefile

撰写完博客，并在本地预览后，需要发布到服务器上。我使用Makefile的形式来生成文档并发布。我的Makefile核心代码如下：
```
#!/makefile
SSH_HOST/cokernut.top
SSH_PORT=22
SSH_USER=ubuntu
SSH_TARGET_DIR=/home/ubuntu/blogs/
SSH_KEY=/home/Cokernut/work/aws/Cokernut-key-tokyo.pem

rsync_upload: publish
    rsync -e "ssh -p $(SSH_PORT) -i $(SSH_KEY)" -P -rvzc --delete $(OUTPUTDIR)/ $(SSH_USER)@$(SSH_HOST):$(SSH_TARGET_DIR) --cvs-exclude
```
第2－4行：指定了要上传内容的目的服务器的地址，端口以及用户名  
第5行：指定了远程服务器上保存博客内容的目录  
第6行：我添加的SSH Identity文件路径。这是因为Amazon EC2登录时我是用SSH Identity文件登录的，而不是使用用户名和密码  
第8-9行：我使用rsync来进行上传操作。rsync可以在本地和远程服务器之间同步文件。同步过程中只同步那些改变了的文件，且传输过程中会压缩数据，它比scp要所需要的带宽要小。这里要注意的是，我在默认生成的Makefile上增加了 -i $(SSH_KEY) ，这个就是指定SSH Identity文件登录远程SSH的方法。

### 配置主题

Pelican支持大量的开源主题，GitHub上的 pelican-themes 项目有几十套主题，大部分都带了效果预览图。可以从里面挑一个你喜欢的主题样式来使用。还有一个更方便的挑选主题的方式，直接打开 www.pelicanthemes.com 挑选吧。一个网页里就列出了几乎所有的主题。我的博客是使用 foundation-default-colours 主题，并在这套主题的基础上进行了一些定制。选定好喜欢的主题后，从GitHub上下载下来所有的主题：

mkdir ~/pelican
cd ~/pelican
git clone https://github.com/getpelican/pelican-themes.git
从里面拷贝一份你选中的主题到项目根目录的 themes 目录下，在本文的例子中是 ~/lab/blogs/themes 。然后在 pelicanconf.py 和 publishconf.py 里通过下面代码指定博客主题：

THEME = "themes/foundation-default-colours"
通常的做法是，选中一个自己喜欢的主题后，会进行一些定制。Pelican使用 Jinja2 来配置主题。一个主题的典型结构如下：
```
├── static
│   ├── css
│   └── images
└── templates
  ├── analytics_cnzz.html // 这个是我添加的使用cnzz的统计服务的代码
  ├── analytics.html    // 这是Google Analytics的代码
  ├── archives.html      // 这个是博客归档页面的模板
  ├── article.html        // 这个是博客正文的显示模板
  ├── base.html          // 这个是所有页面的父类模板，即所有页面都引用这个页面。比如网页导航栏啊之类的，都定义在这里
  ├── categories.html  // 所有博客文章的分类列表
  ├── category.html      // 某个博客分类的文章列表模板
  ├── index.html        // 主页
  ├── page.html          // 分页显示的模板
  ├── tag.html            // 某类标签下的文章列表
  └── tags.html          // 所有的标签列表页面模板
```
稍微有点Jinja的知识加上一些HTML和CSS的知识，就可以自己定义主题了。

为什么博客主页打开半天都不显示出来

因为GFW封锁了几乎所有和Google相关的网站，这些主题里又用了Google的字体，所以下载这些字体时会导致无法下载成功而半天不显示网页。解决方案很简单，直接修改css文件，不去下载Google字体即可。比如针对 foundation-default-colours 主题，打开主题根目录下的 static/css/foundation.css 和 static/css/foundation.min.css 文件，删除掉 @import url("//fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,700italic,400,300,700"); 内容即可。当然，如果你和你的读者都是翻墙高手，那就不会遇到这个问题了。

下载风格包pelican-themes与插件包pelican-plugins

> git clone git://github.com/getpelican/pelican-themes.git

> git clone git://github.com/getpelican/pelican-plugins.git

4. 配置pelicanconf.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #
from __future__ import unicode_literals

# Site settings
AUTHOR = u'Cokernut'
AUTHOR_EMAIL = u'Cokernut@foxmail.com'
SITENAME = u'Cokernut'
TAGLINE = 'Whatever is worth doing is worth doing well.'
SITEURL = 'http://Cokernut.github.io'
DEFAULT_DATE_FORMAT = ('%Y-%m-%d')

TIMEZONE = 'Asia/Shanghai'

DEFAULT_LANG = u'zh'
DEFAULT_METADATA = (
)

DELETE_OUTPUT_DIRECTORY = False

# Blogroll
LINKS = (
  ('HeyLinux', 'http://heylinux.com'),
)

# Social widget
SOCIAL = (
  ('Github', 'http://github.com/Cokernut/...'),
  ('Twitter', 'http://twitter.com/...'),
)

MENUITEMS = (
)

# Disqus
DISQUS_SITENAME = u"Cokernut"

# Content path
PATH = 'content'
PAGE_PATHS = ['pages']
ARTICLE_PATHS = ['articles']
STATIC_PATHS = ['images', 'files']
EXTRA_PATH_METADATA = {
  'files/robots.txt': {'path': 'robots.txt'},
  'images/favicon.ico': {'path': 'favicon.ico'},
}

# URL settings
PAGINATION_PATTERNS = (
  (1, '{base_name}/', '{base_name}/index.html'),
  (2, '{base_name}/page/', '{base_name}/page/{number}.html'),
)
ARTICLE_URL = ('articles/{slug}.html')
ARTICLE_SAVE_AS = ('articles/{slug}.html')
PAGE_LANG_SAVE_AS = False

# Feed
FEED_DOMAIN = SITEURL
FEED_ALL_ATOM = 'feeds/all.atom.xml'
CATEGORY_FEED_ATOM = 'feeds/%s.atom.xml'
TRANSLATION_FEED_ATOM = None

# Theme
THEME = 'pelican-themes/zurb-F5-basic'
COVER_BG_COLOR = '#375152'
DEFAULT_PAGINATION = 10

# Plugin
PLUGIN_PATHS = ['pelican-plugins']
PLUGINS = [ 'sitemap', 'gravatar' ]

# Sitemap
SITEMAP = {
  'format': 'xml',
  'priorities': {
    'articles': 1,
    'pages': 0.9,
    'indexes': 0.8,
  },
  'changefreqs': {
    'indexes': 'daily',
    'articles': 'daily',
    'pages': 'weekly'
  }
}

# Can be useful in development, but set to False when you're ready to publish
RELATIVE_URLS = False
```
5. 配置 publishconf.py
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- #
from __future__ import unicode_literals

# This file is only used if you use `make publish` or
# explicitly specify it as your config file.

import os
import sys
sys.path.append(os.curdir)
from pelicanconf import *

SITEURL = 'http://Cokernut.github.io'
RELATIVE_URLS = False

FEED_ALL_ATOM = 'feeds/all.atom.xml'
CATEGORY_FEED_ATOM = 'feeds/%s.atom.xml'

DELETE_OUTPUT_DIRECTORY = True

# Following items are often useful when publishing

DISQUS_SITENAME = u"Cokernut"
#GOOGLE_ANALYTICS = ""
```
6. 撰写第一篇文章

> cd content

> mkdir articles files images pages

> vim articles/hello.md
```markdown
Title: Hello
Date: 2014-09-24 02:06
Category: Uncategorized
Tags: Pelican, Markdown
Slug: hello
Author: Cokernut
Summary: Hello Pelican, Markdown and GitHub Pages.
Status: draft

## 说明
第5行：Slug是文档的唯一标识，生成html时，会直接使用这个值当html的文件名。
所以，不同博客文章这个值需要保证唯一性，否则生成html时会报错。
第8行：这个表示本文是草稿。比如我们一篇博客经常不是一次性写完的，
写了一半暂不想让读者看到，或者写完想让别人帮忙审查一下，就可以加这一行标识。
这样Pelican在处理时，这篇文章也会生成html，但不会放在博客的主页及分类索引里，
这样普通的读者一般看不到这个文章。有这个标识的文章生成时放在 output/drafts 目录下，
你就可以通过分享url的方式让你的co-worker帮你review你的文章。
我们可以在 content 目录下任意建子目录来组织管理博客文章。
由于我们在设置文件里指定这个值 USE_FOLDER_AS_CATEGORY = True ，这样这些目录名称就自动变成博文分类的目录了。

Hello Pelican, Markdown and GitHub Pages.
```
7. 生成robots.txt与favicon.ico vim files/robots.txt
```
User-agent: *
Sitemap: http://Cokernut.github.io/sitemap.xml
scp heylinux.com:/webserver/blog/rainbow/favicon.ico .
```

8. 配置Disqus

在 Disqus 上注册一个用户并生成一个站点Cokernut.disqus.com；

设置Cokernut.disqus.com站点使其允许域名Cokernut.github.io；

设置以上配置文件为DISQUS_SITENAME = u"Cokernut"，Cokernut 为站点ID

9. 创建GitHub Pages

直接创建一个新的repo，但是其名称必须与ID相同，并加上github.io或github.com后缀。

就我而言，就必须创建一个repo名为 Cokernut.github.io

10. 创建好GitHub Pages之后，生成Blog静态HTML文件
```
cd ~

cd pelican

pelican ./content -o ./output -s ./pelicanconf.py 或者 make html 

pelican /home/dong/pelican/content -o /home/dong/pelican/output -s /home/dong/pelican/pelicanconf.py
WARNING: AUTHOR_SAVE_AS is set to False
Done: Processed 1 article(s), 0 draft(s) and 0 page(s) in 0.23 seconds.
```
11. 进入output目录，将生成好的静态HTML文件上传到GitHub Pages站点Cokernut.github.io中
```
cd output

git init

git remote add origin https://github.com/Cokernut/Cokernut.github.io.git

git add -A

git commit -m "Update Blog"

git push -u origin master
```
12. 等待15分钟左右，访问 Cokernut.github.io 即可看到生成的网站效果

13. 注意

由于我设置了在重新生成HTML时默认不删除output目录，因此每次更新Blog时都需要手动执行'rm -rf output/*'。

这样做的目的，是为了避免删除output/.git目录，方面在生成之后立刻提交到GitHub Pages。

当然在实际操作当中我是编写了alias和scripts来完成这一系列动作的。

预览博客文章

撰写文章的过程中，可以随时在浏览器里预览博客文章。方法是先在博客项目的根目录下执行下面命令来启动预览服务器：

make devserver
这条命令会自动使用 pelicanconf.py 的配置文件来生成html网页，同时在本地的8000端口上启动一个http服务器，供你预览文章。这样，直接打开浏览器，输入 localhost:8000 即可打开本地服务器上的你的博客主页。比如，撰写本文时，我就直接在gedit里码字，然后在浏览器里输入 http://localhost:8000/drafts/build-blog-system-by-pelican.html 来实时预览效果。需要注意的是，上述命令会在后台持续监听 content 目录下的内容，如果这个目录下的内容发生变化，会自动重新生成html页面。所以，在gedit里写完一段内容，切换到浏览器，直接刷新一下就可以看到最新的结果了。

当文章写完后，需要在博客项目根目录上运行 make stopserver 来停止这个预览服务以及数据监控功能。

文章在主页上没看到？

撰写完文章，需要发布时，需要把 Status: draft 这行元数据去掉。否则文章不会出现在博客主页。只会在drafts下看得到。

发布博客