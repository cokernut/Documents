---
title: 使用Hexo+GitHub Pages搭建静态博客
date: 2016-11-29
tags: [GitHub]
categories: GitHub
description:
---
使用Hexo+GitHub Pages搭建静态博客
<!--more-->

## Hexo

一个基于Node.js的快速，简单和强大的博客框架。

## GitHub Pages  

GitHub Pages可以被认为是用户编写的、托管在github上的静态网页。

## 建立GitHub Pages项目  

在GitHub上建立一个名称为：username.github.io的项目，username一定要填写自己的GitHub的名称，之后可以把生成的静态网页提交到这个项目上，
然后可以通过 https://username.github.io 这个地址访问到我们生成的静态网页。

## Hexo安装  

+ Hexo是基于Node.js的，所以首先要安装[Node.js](https://nodejs.org/en/)
+ 使用命令安装hexo:  

  > npm install hexo-cli -g  

  注意：使用-g表示全局安装，没有加-g表示本地安装（当前目录安装）。

查看版本：

> hexo version  或者 hexo v  

```
hexo: 3.2.2
hexo-cli: 1.0.2
os: Windows_NT 10.0.14393 win32 x64
http_parser: 2.7.0
node: 6.9.1
v8: 5.1.281.84
uv: 1.9.1
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 57.1
modules: 48
openssl: 1.0.2j
```

查看帮助：

> hexo help  或者 hexo h  

```
Usage: hexo <command>

Commands:
  clean     Removed generated files and cache. #删除生成的文件和缓存。
  config    Get or set configurations.
  deploy    Deploy your website. #部署网站，就是提交到项目中
  generate  Generate static files. #生成静态文件
  help      Get help on a command.   #查看帮助信息
  init      Create a new Hexo folder. #创建一个hexo项目，不指定文件夹名，则在当前目录创建，否则在后面指定的文件夹创建
  list      List the information of the site
  migrate   Migrate your site from other system to Hexo.
  new       Create a new post. #新建文件，可以指定模板，否则使用默认模板
  publish   Moves a draft post from _drafts to _posts folder.
  render    Render files with renderer plugins.
  server    Start the server. #开启服务器
  version   Display version information. #查看版本信息

Global Options:
  --config  Specify config file instead of using _config.yml #指定配置文件，代替默认的_config.yml
  --cwd     Specify the CWD #自定义当前工作目录
  --debug   Display all verbose messages in the terminal #调试模式，输出所有日志信息
  --draft   Display draft posts #草稿模式
  --safe    Disable all plugins and scripts #安全模式，禁用所有的插件和脚本
  --silent  Hide output on console #无日志输出模式
```

## 新建项目

> hexo init [项目文件夹]  

注意：项目文件夹是可选的，不指定文件夹名，则在当前目录创建项目，否则在后面指定的文件夹创建项目  

新建项目之后切换到项目的根目录使用：

> npm install 

这条命令来为项目安装依赖包，具体安装内容可以在package.json文件里找到。  

### 项目结构

```
> scaffolds ：模板文件夹，新建文章时，Hexo 会根据 scaffold 来建立文件。Hexo 有三种默认布局： post 、 page 和 draft ，它们分别对应不同的路径。新建文件的默认布局是 post ，可以在配置文件中更改布局。用 draft 布局生成的文件会被保存到 source/_drafts 文件夹。
> source ：资源文件夹是存放用户资源的地方。
> > source/_post ：文件箱。（低版本的hexo还会存在一个 _draft ，这是草稿箱）除 _posts 文件夹之外，开头命名为 _ (下划线)的文件/ 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去
> themes ：主题 文件夹。Hexo 会根据主题来生成静态页面。
> > themes/landscape ：默认的皮肤文件夹
> > > themes/landscape/_config.yml :当前主题配置文件
> _config.yml ：全局的配置文件，每次更改要重启服务。
```
之后使用：  

> hexo server 或者 hexo s  

来开启本地服务器进行预览。
我们可以过浏览器打开地址， http://localhost:4000/ 或者 http://0.0.0.0:4000 来预览我们的项目。

## Hexo配置  

> _config.yml  

```xml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site 站点配置
title: Cokernut #网站标题
subtitle: #网站副标题
description: #网站描述
author: Cokernut #作者
language: zh-CN #网站使用的语言
timezone: Asia/Shanghai #网站时区

# URL 配置
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://cokernut.top #网址，搜索时会在搜索引擎中显示
root: / #网站根目录
permalink: :year/:month/:day/:title/ #永久链接格式
permalink_defaults: #永久链接中各部分的默认值

# Directory 目录配置
source_dir: source #资源文件夹，这个文件夹用来存放内容
public_dir: public #公共文件夹，这个文件夹用于存放生成的站点文件
tag_dir: tags #标签文件夹
archive_dir: archives #归档文件夹
category_dir: categories #分类文件夹
code_dir: downloads/code #Include code 文件夹
i18n_dir: :lang #国际化文件夹
skip_render: #跳过指定文件的渲染，您可使用 glob 来配置路径

# Writing 写作配置
new_post_name: :title.md # 新文章的文件名称
default_layout: post #默认布局
titlecase: false #把标题变成titlecase
external_link: true #在新标签页打开外部链接
filename_case: 0 #把文件名称转换为 (1) 小写或 (2) 大写
render_drafts: false #显示草稿
post_asset_folder: false #是否启动资源文件夹
relative_link: false #把链接改为与根目录的相对位址
future: true
highlight: #代码块的设置
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag 分类 & 标签
default_category: uncategorized #默认分类
category_map: #分类别名
tag_map: #标签别名

# Date / Time format 日期/时间
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination 分页
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions 扩展
## Plugins: https://hexo.io/plugins/ #插件
## Themes: https://hexo.io/themes/ #主题
theme: landscape-plus #当前主题名称

# Deployment #部署
## Docs: https://hexo.io/docs/deployment.html
deploy: #github配置
  type: git
  repo: https://github.com/cokernut/cokernut.github.io.git
  branch: master
```

主题相关配置可以在主题下面的_config.yml里进行配置，不同的主题有些配置不同。

## 新建文件

> hexo new [模板] <标题> 或者 hexo n [模板] <标题>   

注意：其中[模板]是可选参数，默认值为 post，如果没有设置[模板]的话，默认使用 _config.yml 
中的 default_layout 参数代替。如果标题包含空格的话，需用引号括起来。  

Hexo提供的模板在 scaffolds 目录下，也可以在此目录下自建模板文件或者修改模板文件。新建的文件会在
source/_post 目录下。

发表的文章会全部显示，如果文章很长，就只要显示文章的摘要就行了。写法如下：
```
---
title:
date:
tag： # tags 和 categories 有多个，则用数组形式。[Java, Android]
categories：
...
---

摘要

<!--more-->

全文

```

这时主页就能够看到只显示标题和摘要了，同时会有Read More的链接按钮，
这个链接按钮的文字可以更改，在主题的配置文件(themes/主题文件夹/_config.yml)中，
找到 Content:
```
# Content
excerpt_link: Read More #可以更改成想要显示的文字
fancybox: true
```

## 页面的生成

在项目部署之前要通过命令把所有的文章都做静态化处理，生成对应的html, javascript, css，
使得所有的文章都是由静态文件组成的。

> hexo generate 或者 hexo g  

运行这个命令之后在本地目录下，会生成一个public的目录，里面包括了所有静态化的文件。 

## 项目部署

生成静态文件之后，如果要发布到github，还需要配置 deploy 指令。
在全局的配置文件(_config.yml)中找到 deploy ：
```
# Deployment #部署
## Docs: https://hexo.io/docs/deployment.html
deploy: #github配置
  type: git #类型GitHub的类型属于git
  repo: https://github.com/cokernut/cokernut.github.io.git #GitHub Pages的git地址
  branch: master #提交到的分支
```

之后安装hexo-deployer-git插件：

> npm install hexo-deployer-git -s  

然后使用命令：

> hexo deploy 或者 hexo d

把项目部署到GitHub Pages上，提交的文件是在username.github.io项目上。  
之后你可以访问http://username.github.io 查看效果。

## 绑定自己的域名（可选）

1. 购买一个自己的域名
2. 在username.github.io项目的根目录建立一个CNAME文件，在文件中填写上自己购买的域名，比如：username.net，不需要添加http或者是www等前缀。
3. 使用ping命令得到username.github.io的IP地址并记录。
4. 在域名的DNS配置上添加记录，两种方式选一种：  

添加A记录：  

记录类型  |主机记录  |记录值
----     |----     |--------
A        |@        |IP地址
A        |www      |IP地址

或者添加CNAME记录：  

记录类型  |主机记录  |记录值  
----     |----     |------  
CNAME    |@        |username.github.io  
CNAME    |www      |username.github.io  
    
两种记录类型相同主机记录只能添加一种不然会提示冲突，并且不能添加。