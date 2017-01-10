---
title: Python3学习笔记--基础知识
date: 2016-10-04
tags: [Python]
categories: Python
description:
---
Python3学习笔记--基础知识
<!--more-->

```python
''' 
Python3 基本数据类型 
Numbers（数字） 
String（字符串） 
List（列表） 
Tuple（元组） 
Sets（集合） 
Dictionaries（字典） 
空值是Python里一个特殊的值，用None表示。None不能理解为0，因为0是有意义的，而None是一个特殊的空值。 
布尔值和布尔代数的表示完全一致，一个布尔值只有True、False两种值，要么是True，要么是False，在Python中，可以直接用True、False表示布尔值（请注意大小写） 
'''  
  
""" 
Numbers（数字） 
Python 3支持int、float、bool、complex（复数）。 
数值类型的赋值和计算都是很直观的，就像大多数语言一样。内置的type()函数可以用来查询变量所指的对象类型。 
>>> a, b, c, d = 20, 5.5, True, 4+3j 
>>> print(type(a), type(b), type(c), type(d)) 
<class 'int'> <class 'float'> <class 'bool'> <class 'complex'> 
数值运算： 
>>> 5 + 4  # 加法 
9 
>>> 4.3 - 2 # 减法 
2.3 
>>> 3 * 7  # 乘法 
21 
>>> 2 / 4  # 除法，得到一个浮点数 
0.5 
>>> 2 // 4 # 除法，得到一个整数 
0 
>>> 17 % 3 # 取余 
2 
>>> 2 ** 5 # 乘方 
32 
注意： 
1、Python可以同时为多个变量赋值，如a, b = 1, 2。 
2、一个变量可以通过赋值指向不同类型的对象。 
3、数值的除法（/）总是返回一个浮点数，要获取整数使用//操作符。 
4、在混合计算时，Python会把整型转换成为浮点数。 
 
String（字符串） 
Python中的字符串str用单引号(' ')或双引号(" ")括起来，同时使用反斜杠(\)转义特殊字符。 
>>> s = 'Yes,he doesn\'t' 
>>> print(s, type(s), len(s)) 
Yes,he doesn't <class 'str'> 14 
如果你不想让反斜杠发生转义，可以在字符串前面添加一个r，表示原始字符串： 
>>> print('C:\some\name') 
C:\some 
ame 
>>> print(r'C:\some\name') 
C:\some\name 
另外，反斜杠可以作为续行符，表示下一行是上一行的延续。还可以使用"""#"""或者'''...'''跨越多行。  
"""字符串可以使用 + 运算符串连接在一起，或者用 * 运算符重复：  
>>> print('str'+'ing', 'my'*3)  
string mymymy  
Python中的字符串有两种索引方式，第一种是从左往右，从0开始依次增加；第二种是从右往左，从-1开始依次减少。  
注意，没有单独的字符类型，一个字符就是长度为1的字符串。  
>>> word = 'Python'  
>>> print(word[0], word[5])  
P n  
>>> print(word[-1], word[-6])  
n P  
还可以对字符串进行切片，获取一段子串。用冒号分隔两个索引，形式为变量[头下标:尾下标]。  
截取的范围是前闭后开的，并且两个索引都可以省略：  
>>> word = 'ilovepython'  
>>> word[1:5]  
'love'  
>>> word[:]  
'ilovepython'  
>>> word[5:]  
'python'  
>>> word[-10:-6]  
'love'  
与C字符串不同的是，Python字符串不能被改变。向一个索引位置赋值，比如word[0] = 'm'会导致错误。  
注意：  
1、反斜杠可以用来转义，使用r可以让反斜杠不发生转义。  
2、字符串可以用+运算符连接在一起，用*运算符重复。  
3、Python中的字符串有两种索引方式，从左往右以0开始，从右往左以-1开始。  
4、Python中的字符串不能改变。  
  
List（列表）  
List（列表） 是 Python 中使用最频繁的数据类型。  
列表是写在方括号之间、用逗号分隔开的元素列表。列表中元素的类型可以不相同：  
>>> a = ['him', 25, 100, 'her']  
>>> print(a)  
['him', 25, 100, 'her']  
和字符串一样，列表同样可以被索引和切片，列表被切片后返回一个包含所需元素的新列表。详细的在这里就不赘述了。  
列表还支持串联操作，使用+操作符：  
>>> a = [1, 2, 3, 4, 5]  
>>> a + [6, 7, 8]  
[1, 2, 3, 4, 5, 6, 7, 8]  
与Python字符串不一样的是，列表中的元素是可以改变的：  
>>> a = [1, 2, 3, 4, 5, 6]  
>>> a[0] = 9  
>>> a[2:5] = [13, 14, 15]  
>>> a  
[9, 2, 13, 14, 15, 6]  
>>> a[2:5] = []   # 删除  
>>> a  
[9, 2, 6]  
List内置了有很多方法，例如append()、pop()等等。  
list.append(x)  把一个元素添加到列表的结尾，相当于 a[len(a):] = [x]。  
list.extend(L)  通过添加指定列表的所有元素来扩充列表，相当于 a[len(a):] = L。  
list.insert(i, x)   在指定位置插入一个元素。第一个参数是准备插入到其前面的那个元素的索引，例如 a.insert(0, x) 会插入到整个列表之前，而 a.insert(len(a), x) 相当于 a.append(x) 。  
list.remove(x)  删除列表中值为 x 的第一个元素。如果没有这样的元素，就会返回一个错误。  
list.pop([i])   从列表的指定位置删除元素，并将其返回。如果没有指定索引，a.pop()返回最后一个元素。元素随即从列表中被删除。（方法中 i 两边的方括号表示这个参数是可选的，而不是要求你输入一对方括号，你会经常在 Python 库参考手册中遇到这样的标记。）  
list.clear()    移除列表中的所有项，等于del a[:]。  
list.index(x)   返回列表中第一个值为 x 的元素的索引。如果没有匹配的元素就会返回一个错误。  
list.count(x)   返回 x 在列表中出现的次数。  
list.sort() 对列表中的元素进行排序。  
list.reverse()  倒排列表中的元素。  
list.copy() 返回列表的浅复制，等于a[:]。  
注意：  
1、List写在方括号之间，元素用逗号隔开。  
2、和字符串一样，list可以被索引和切片。  
3、List可以使用+操作符进行拼接。  
4、List中的元素是可以改变的。  
  
Tuple（元组）  
元组（tuple）与列表类似，不同之处在于元组的元素不能修改。元组写在小括号里，元素之间用逗号隔开。  
元组中的元素类型也可以不相同：  
>>> a = (1991, 2014, 'physics', 'math')  
>>> print(a, type(a), len(a))  
(1991, 2014, 'physics', 'math') <class 'tuple'> 4  
元组与字符串类似，可以被索引且下标索引从0开始，也可以进行截取/切片（看上面，这里不再赘述）。  
其实，可以把字符串看作一种特殊的元组。  
>>> tup = (1, 2, 3, 4, 5, 6)  
>>> print(tup[0], tup[1:5])  
1 (2, 3, 4, 5)  
>>> tup[0] = 11  # 修改元组元素的操作是非法的  
虽然tuple的元素不可改变，但它可以包含可变的对象，比如list列表。  
构造包含0个或1个元素的tuple是个特殊的问题，所以有一些额外的语法规则：  
tup1 = () # 空元组  
tup2 = (20,) # 一个元素，需要在元素后添加逗号  
另外，元组也支持用+操作符：  
>>> tup1, tup2 = (1, 2, 3), (4, 5, 6)  
>>> print(tup1+tup2)  
(1, 2, 3, 4, 5, 6)  
string、list和tuple都属于sequence（序列）。  
注意：  
1、与字符串一样，元组的元素不能修改。  
2、元组也可以被索引和切片，方法一样。  
3、注意构造包含0或1个元素的元组的特殊语法规则。  
4、元组也可以使用+操作符进行拼接。  
  
Sets（集合）  
集合（set）是一个无序不重复元素的集。  
基本功能是进行成员关系测试和消除重复元素。  
可以使用大括号 或者 set()函数创建set集合，注意：创建一个空集合必须用 set() 而不是 { }，因为{ }是用来创建一个空字典。  
>>> student = {'Tom', 'Jim', 'Mary', 'Tom', 'Jack', 'Rose'}  
>>> print(student)   # 重复的元素被自动去掉  
{'Jim', 'Jack', 'Mary', 'Tom', 'Rose'}  
>>> 'Rose' in student  # membership testing（成员测试）  
True  
>>> # set可以进行集合运算  
...  
>>> a = set('abracadabra')  
>>> b = set('alacazam')  
>>> a  
{'a', 'b', 'c', 'd', 'r'}  
>>> a - b     # a和b的差集  
{'b', 'd', 'r'}  
>>> a | b     # a和b的并集  
{'l', 'm', 'a', 'b', 'c', 'd', 'z', 'r'}  
>>> a & b     # a和b的交集  
{'a', 'c'}  
>>> a ^ b     # a和b中不同时存在的元素  
{'l', 'm', 'b', 'd', 'z', 'r'}  
  
Dictionaries（字典）  
字典（dictionary）是Python中另一个非常有用的内置数据类型。  
字典是一种映射类型（mapping type），它是一个无序的键 : 值对集合。  
关键字必须使用不可变类型，也就是说list和包含可变类型的tuple不能做关键字。  
在同一个字典中，关键字还必须互不相同。  
>>> dic = {}  # 创建空字典  
>>> tel = {'Jack':1557, 'Tom':1320, 'Rose':1886}  
>>> tel  
{'Tom': 1320, 'Jack': 1557, 'Rose': 1886}  
>>> tel['Jack']   # 主要的操作：通过key查询  
1557  
>>> del tel['Rose']  # 删除一个键值对  
>>> tel['Mary'] = 4127  # 添加一个键值对  
>>> tel  
{'Tom': 1320, 'Jack': 1557, 'Mary': 4127}  
>>> list(tel.keys())  # 返回所有key组成的list  
['Tom', 'Jack', 'Mary']  
>>> sorted(tel.keys()) # 按key排序  
['Jack', 'Mary', 'Tom']  
>>> 'Tom' in tel       # 成员测试  
True  
>>> 'Mary' not in tel  # 成员测试  
False  
构造函数 dict() 直接从键值对sequence中构建字典，当然也可以进行推导，如下：  
>>> dict([('sape', 4139), ('guido', 4127), ('jack', 4098)])  
{'jack': 4098, 'sape': 4139, 'guido': 4127}  
  
>>> {x: x**2 for x in (2, 4, 6)}  
{2: 4, 4: 16, 6: 36}  
  
>>> dict(sape=4139, guido=4127, jack=4098)  
{'jack': 4098, 'sape': 4139, 'guido': 4127}  
另外，字典类型也有一些内置的函数，例如clear()、keys()、values()等。  
注意：  
1、字典是一种映射类型，它的元素是键值对。  
2、字典的关键字必须为不可变类型，且不能重复。  
3、创建空字典使用{ }。  
"""  
  
#输出  
print('hello, world')  
  
#输入  
name = input()  
  
# 获取用户输入的数字  
num = int(input("请输入一个数字: "))  
a = float(input('请输入一个数字:'))  
  
#条件判断if..else  
a = 1  
if a >= 0:  
    if a == 0:  
        print('a=0')  
    else:  
        print('a是正数')  
else:  
    print('a是负数')  
#if..elif..else  
if a == 0:  
    print('a=0')  
elif a > 0:  
    print('a是正数')  
else:  
    print('a是负数')  
  
#循环  
#for...in循环  
nums = [1, 2, 3]  
for num in nums:  
    print(num)  
#while循环  
i = 0  
while i < len(nums):  
    print(nums[i])  
    i += 1  
  
#pass语句什么都不做。它只在语法上需要一条语句但程序不需要任何操作时使用  
#break语句可以跳出for和while的循环体。如果你从for或while循环中终止，任何对应的循环else块将不执行。  
#continue语句被用来告诉Python跳过当前循环块中的剩余语句，然后继续进行下一轮循环。  
  
#交换变量  
x = input('输入 x 值: ')  
y = input('输入 y 值: ')  
# 不使用临时变量  
x,y = y,x  
  
#文件的读写  
#由于文件读写时都有可能产生IOError，一旦出错，后面的f.close()就不会调用。所以，为了保证无论是否出错都能正确地关闭文件，我们可以使用try ... finally来实现：  
'''''r、w、a为打开文件的基本模式，对应着只读、只写、追加模式； 
b、t、+、U这四个字符，与以上的文件打开模式组合使用，二进制模式，文本模式，读写模式、通用换行符，根据实际情况组合使用'''  
try:  
    f = open('test.txt', 'r')  
    print(f.read())  
finally:  
    if f:  
        f.close()  
#Python引入了with语句来自动帮我们调用close()方法  
# 写文件  
with open("test.txt", "wt") as out_file:  
    out_file.write("写入到文件中\n换行了")  
  
# 读文件  
with open("test.txt", "rt") as in_file:  
    text = in_file.read()  
```