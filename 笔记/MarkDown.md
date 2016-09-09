Title: MarkDown语法学习
Date: 2016-09-09
Tags: MarkDown
Slug: Markdown-grammar
Authors: Cokernut
Summary: MarkDown基础语法学习与练习

# MarkDown

## 代码

如果你只想高亮语句中的某个函数名或关键字，可以使用 `function_name()` 实现

通常编辑器根据代码片段适配合适的高亮方法，但你也可以用 ``` 包裹一段代码，并指定一种语言

```
    ```javascript
    $(document).ready(function () {
        alert('hello world');
    }); 
    ```
```
支持的语言：actionscript, apache, bash, clojure, cmake, coffeescript, cpp, cs, css, d, delphi, django, erlang, go, haskell, html, http, ini, java, javascript, json, lisp, lua, markdown, matlab, nginx, objectivec, perl, php, python, r, ruby, scala, smalltalk, sql, tex, vbscript, xml

也可以使用 4 空格缩进，再贴上代码，实现相同的的效果
```
    def g(x):
        yield from range(x, 0, -1)
    yield from range(x)
```

## 标题

文章内容较多时，可以用标题分段：
```
标题1
======

标题2
-----

## 大标题 ##
### 小标题 ###
```
## 粗斜体
```
*斜体文本*    _斜体文本_
**粗体文本**    __粗体文本__
***粗斜体文本***    ___粗斜体文本___
```

## 链接

常用链接方法
```
文字链接 [链接名称](http://链接网址)
网址链接 <http://链接网址>
```
高级链接技巧
```
这个链接用 1 作为网址变量 [Google][1].
这个链接用 yahoo 作为网址变量 [Yahoo!][yahoo].
然后在文档的结尾为变量赋值（网址）

  [1]: http://www.google.com/
  [yahoo]: http://www.yahoo.com/
```

## 列表

普通无序列表
```
- 列表文本前使用 [减号+空格]
+ 列表文本前使用 [加号+空格]
* 列表文本前使用 [星号+空格]
```
普通有序列表
```
1. 列表前使用 [数字+空格]
2. 我们会自动帮你添加数字
7. 不用担心数字不对，显示的时候我们会自动把这行的 7 纠正为 3
```
列表嵌套
```
1. 列出所有元素：
    - 无序列表元素 A
        1. 元素 A 的有序子列表
    - 前面加四个空格
2. 列表里的多段换行：
    前面必须加四个空格，
    这样换行，整体的格式不会乱
3. 列表里引用：

    > 前面空一行
    > 仍然需要在 >  前面加四个空格

4. 列表里代码段：

    ```
    前面四个空格，之后按代码语法 ``` 书写
    ```

        或者直接空八个，引入代码块
```

## 分割线
使用 --- 或者 *** 或者 * * * 表示水平分割线。

## 次常用标记
使用 标签: 或者 Tags: 表示标签标记。
### 删除线
使用 ~~ 表示删除线。
```
~~这是一条删除线~~
```
1.注意 ~~ 和 要添加删除线的文字之间不能有空格。
2.我常使用在显示的告诉自己这行文字是要删除的。
### 注脚
使用 [^footer] 表示注脚。
```
这是一个注脚测试[^footer1]。
[^footer1]: 这是一个测试，用来阐释注脚。
```

## 不常用标记
### 实现页内跳转
使用html代码实现页内跳转。在要跳转到的位置定义个锚 <span id = "jump">hehe</span> ，然后使用 [你好](#jump) 将 你好 设置为一单击即跳转到 hehe 所在位置的效果。
```
[你好](#jump)
<span id = "jump">hehe</span>
```

## 表格

具体使用方式请看示例。
```
------: 为右对齐。
:------ 为左对齐。
:------: 为居中对齐。
------- 为使用默认居中对齐。
```
示例
|         序号    |    交易名    |    交易说明    |    备注    |
|    ------: |    :-------:    |    :---------   |    ------    |
|    1    |    prfcfg    |    菜单配置    |    可以通过此交易查询到所有交易码和菜单的对应关系    |
|    2    |    gentmo    |    编译所有交易    |    |
|    100000    |    sysdba    |    数据库表模型汇总    |    |
序号	交易名	交易说明	备注
1	prfcfg	菜单配置	可以通过此交易查询到所有交易码和菜单的对应关系
2	gentmo	编译所有交易	
100000	sysdba	数据库表模型汇总	
### 注意
每个Markdown解析器都不一样，可能左右居中对齐方式的表示方式不一样。

## 引用

普通引用
```
> 引用文本前使用 [大于号+空格]
> 折行可以不加，新起一行都要加上哦
```
引用里嵌套引用
```
> 最外层引用
> > 多一个 > 嵌套一层引用
> > > 可以嵌套很多层
```
引用里嵌套列表
```
> - 这是引用里嵌套的一个列表
> - 还可以有子列表
>     * 子列表需要从 - 之后延后四个空格开始
```
引用里嵌套代码块
```
>     同样的，在前面加四个空格形成代码块
>  
> ```
> 或者使用 ``` 形成代码块
> ```
```
## 图片

跟链接的方法区别在于前面加了个感叹号 !，这样是不是觉得好记多了呢？
```
![图片名称](http://图片网址)
```
当然，你也可以像网址那样对图片网址使用变量
```
这个链接用 1 作为网址变量 [Google][1].
然后在文档的结尾位变量赋值（网址）

 [1]: http://www.google.com/logo.png
```
也可以使用 HTML 的图片语法来自定义图片的宽高大小
```
<img src="htt://example.com/sample.png" alt="alt" Title="title" width="400" height="100">
```
## 换行

如果另起一行，只需在当前行结尾加 2 个空格
```
在当前行的结尾加 2 个空格  
这行就会新起一行
```
如果是要起一个新段落，只需要空出一行即可。

## 分隔符

如果你有写分割线的习惯，可以新起一行输入三个减号-。当前后都有段落时，请空出一行：
```
前面的段落

---

后面的段落
```

## 高级技巧
行内 HTML 元素

目前只支持部分段内 HTML 元素效果，包括 ```<kdb> <b> <i> <em> <sup> <sub> <br>```

如 : 键位显示
```
使用 <kbd>Ctrl<kbd>+<kbd>Alt<kbd>+<kbd>Del<kbd> 重启电脑
```
代码块
```
使用 <pre></pre> 元素同样可以形成代码块
```
粗斜体
```
<b> Markdown 在此处同样适用，如 *加粗* </b>
```

## 符号转义

如果你的描述中需要用到 markdown 的符号，比如 _ # * 等，但又不想它被转义，这时候可以在这些符号前加反斜杠，如 \_ \# \* 进行避免。
```
\_不想这里的文本变斜体\_
\*\*不想这里的文本被加粗\*\*
```

## 扩展

支持 jsfiddle、gist、runjs、优酷视频，直接填写 url，在其之后会自动添加预览点击会展开相关内容。
```
http://{url_of_the_fiddle}/embedded/[{tabs}/[{style}]]/
https://gist.github.com/{gist_id}
http://runjs.cn/detail/{id}
http://v.youku.com/v_show/id_{video_id}.html
```

## 公式

当你需要在编辑器中插入数学公式时，可以使用两个美元符 $$ 包裹 TeX 或 LaTeX 格式的数学公式来实现。提交后，问答和文章页会根据需要加载 Mathjax 对数学公式进行渲染。如：
```
$$ x = {-b \pm \sqrt{b^2-4ac} \over 2a}. $$

$$
x \href{why-equal.html}{=} y^2 + 1
$$
```
同时也支持 HTML 属性，如：
```
$$ (x+1)^2 = \class{hidden}{(x+1)(x+1)} $$

$$
(x+1)^2 = \cssId{step1}{\style{visibility:hidden}{(x+1)(x+1)}}
$$
```

# MarkDown练习

头部
====

头部
----

## 列表
* Candy.
* Gum.
* Booze.

+ Candy.
+ Gum.
+ Booze

- Candy.
- Gum.
- Booze.

1. Red
2. Green
3. Blue


3. Bird
1. McHale
8. Parish

## 代码片段
### 不指定语言
`java code`
### 指定语言
```javascript
$(document).ready(function () {
    alert('hello world');
});
```

## 链接
[baidu]("www.baidu.com")

<http://www.baidu.com/>
```
常用链接方法

文字链接 [链接名称](http://链接网址)
网址链接 <http://链接网址>
高级链接技巧

这个链接用 1 作为网址变量 [Google][1].
这个链接用 yahoo 作为网址变量 [Yahoo!][yahoo].
然后在文档的结尾为变量赋值（网址）

[1]: http://www.google.com/
[yahoo]: http://www.yahoo.com/
```

## 图片
![alt text](http://www.86ps.com/uploadfiles/jpg/2011-11/2011111414003220188.jpg "Title")
<img src="http://img2.3lian.com/2014/c7/12/d/77.jpg" alt="图片名称" Title="风景" width=256 height=256 />

> 引用

## 字体
*斜体文本*    _斜体文本_

**粗体文本**    __粗体文本__

***粗斜体文本***    ___粗斜体文本___

## 表格
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

dog | bird | cat
----|------|----
foo | foo  | foo
bar | bar  | bar
baz | baz  | baz

## 反斜杠转义
\*literal asterisks\*
```
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井字号
+   加号
-   减号
.   英文句点
!   惊叹号
```
