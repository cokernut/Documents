Title: 基础模板
Date: 2016-09-09
Modified: 2016-09-09
Tags: base-template
Slug: base-template
Authors: Cokernut
Summary: 基础模板摘要
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