---
date: 2013-03-25
title: learn python the hard way笔记
---

@&lt;font color="red"&gt; 本文为"learn python the hard way"一书的笔记
@&lt;/font&gt;

方法
====

在python交互窗口中，打help()，会进入帮助模式，这里可以查看所有的文档，
比如打modules会列出所有的模块，打函数会列出函数名，打关键字会列出关键字
的说明。按q退出。

不进入help模式，直接help(object)也可以查看相关的帮助。

在终端，通过pydoc工具可以查看文档。

要多读代码，试图理解别人的代码，不懂就去查。

输出
====

通过下面的练习：

``` {.python}
print "Hello World!"
print 'Yay! Printing.'
print "I'd much rather you 'not'"
print 'I "said" do not touch this'
```

可以看出，可以用双引号和单引号表示字符串，如果字符串里面有单引号，则用双引号，
如果有双引号，则用单引号，这一点和PHP很像。

\^*英*文叫caret

\#英文叫octothorpe(井号), 也叫pound, hash, mesh.
它就是python中的注释符号。

不禁想起多行注释，搜索了一下，找到[这篇文章](http://hi.baidu.com/5unge/item/8d650d2733c1ca3694f62bf2)
是这样的：

``` {.python}
#print "Hello World!"
print 'Yay! Printing.'
"""
print "I'd much rather you 'not'"
print 'I "said" do not touch this'
"""
print 'Yay! Printing.'
'''
print "I'd much rather you 'not'"
print 'I "said" do not touch this'
'''
```

前后使用3个号或者单引号，就可以多行注释了。但是估计旧版本的不行。

数字
====

+, -, \*, /, %, &lt;, &gt;, &lt;=,
&gt;=这种和其它语言是一样的，优先级也一样。

这里有一个地方，字符串用逗号(,)连接，会用空格替代。

``` {.python}
>>> print "aaa", 25 + 10 / 2
aaa 30
>>> print "aaa", "bbb"
aaa bbb
```

另外，布尔表达式返回的是True和False.

``` {.python}
>>> print 3 < 2
False
>>> print 2 < 3
True
```

用python来当计算机是非常好的，书上只给出了一些简单的运算，我去搜索了一下，
找到了[这篇文章](http://blog.csdn.net/wklken/article/details/6312876)
,学习了一些基础的东西。

用e表示的是10的多少次幂，而且结果是浮点数，如：

``` {.python}
>>> 2e10
20000000000.0
```

还能表示复数！虚部用j或J表示，如：

``` {.python}
>>> (2 + 3j) + (1 - 2J)
(3+1j)
```

幂的运算符为\*\*，浮点数的除法符号/是默认精确计算的，而//运算符则会取整：

``` {.python}
>>> 3.2 / 2
1.6
>>> 3.2 // 2
1.0
>>> 2 ** 10
1024
```

其它的符号和c语言的都差不多。

使用数学函数需要import
math模块，使用dir(math)可列出函数，help(math)可查看具体的
定义和函数原型：

``` {.python}
>>> dir(math)
['__doc__', '__file__', '__name__', '__package__', 'acos', 'acosh', 
'asin', 'asinh', 'atan', 'atan2', 'atanh', 'ceil', 'copysign', 'cos',
'cosh', 'degrees', 'e', 'erf', 'erfc', 'exp', 'expm1', 'fabs', 'factorial',
 'floor', 'fmod', 'frexp', 'fsum', 'gamma', 'hypot', 'isinf', 'isnan', 
'ldexp', 'lgamma', 'log', 'log10', 'log1p', 'modf', 'pi', 'pow', 
'radians', 'sin', 'sinh', 'sqrt', 'tan', 'tanh', 'trunc']
```

我猜其它模块也是这样用的。

调用的时候，都是math.XX这种形式，比如：

``` {.python}
>>> math.e
2.718281828459045
>>> math.pi
3.141592653589793
>>> math.factorial(10)
3628800
>>> math.ceil(3.3)
4.0
>>> math.floor(3.3)
3.0
>>> math.fabs(-3)
3.0
>>> math.hypot(3, 4)
5.0
>>> math.pow(2, 10)
1024.0
>>> math.sqrt(256)
16.0
>>> math.log(math.e)
1.0
>>> math.log10(10e2)
3.0
>>> math.trunc(3.3)
3
>>> math.isnan(3)
False
>>> math.radians(180)
3.141592653589793
>>> math.degrees(math.pi)
180.0
```

其中hypot其实是直角三角形的斜边，isnan是判断不是数字的，如果不是数字，则返回True。

格式化输出
==========

和C有一点不同，后面引用变量的时候，要加%号。

``` {.python}
>>> print "%d" %2
2
>>> print "%d" % 2
2
>>> print "%s %d" %('aa', 2)
aa 2
>>> print '%s %d' %('aa', 2)
aa 2
```

%后面与变量之间可以加也可以不加空格，如果是多个变量则用括号括起来，
中间用逗号分开。格式化字符串也可以用单引号。

书上的内容只是一个指引，更多的需要我自学，找到了[这篇文章](http://www.pythonclub.org/python-basic/print)
,发现了：

len库函数可以求字符串的长度：

``` {.python}
>>> len("abc")
3
```

对字符串进行有精度的输出：

``` {.python}
>>> print "%.3s" %('abcde')
abc
>>> print "%.*s" %(3, 'abcde')
abc
```

print默认自动换行，给它加上逗号就不会换行了。或者使用sys.stdout.write

``` {.python}
import sys

print "abc",
sys.stdout.write('aaa')
```

%r是万能的，它能输出任何格式。

``` {.python}
>>> print "%r" %2
2
>>> print "%r" %'a'
'a'
>>> print "%r" %True
True
```

但是智能意味着效率低，因为他还要有个判断过程。

输出多行，使用3个双引号或单引号：

``` {.python}
print """
sldfj
sdlfj
sldfjlfjldsfj
sldfjlsdfj
"""
print '''
sldfjjsf
sdlfj
'''
```

注意它和多行注释的区别。

字符串
======

看下面的例子：

``` {.python}
str1 = 'abcbc'
str2 = str1
print str2
print str1 + str2
pos = str1.index('b')
print pos
print str1.find('c')
print cmp(str1, str2)
print cmp(str1, str1 + str2)
print cmp(str1 + str2, str1)
str3 = str1.upper()
print str3
print str3.lower()
print str3[1:2]
print str3[::-1]
```

结果为:

``` {.example}
abcbc
abcbcabcbc
1
2
0
-1
1
ABCBC
abcbc
B
CBCBA
```

我们来分析：

-   字符串使用加号+进行连接
-   复制字符串，直接用=号来赋值。
-   查找字符串，可以用index，也可以用find，返回的都是下标
-   比较字符串，使用cmp相等返回0,左边小于右边返回-1,大于返回1
-   变成大写使用upper，变成小写用lower
-   截取字符串，通过\[begin:end\]，不包含end(这是很多语言的约定)
-   翻转字符串，通过\[::-1\]

另外，字符串还有乘法，如：

``` {.python}
>>> print 'abc' * 10
abcabcabcabcabcabcabcabcabcabc
```

输入
====

最简单的是raw\_input()

``` {.python}
>>> a = raw_input()
1
>>> print a
1
>>> a = raw_input()
abc
>>> print a
abc
```

加点提示：

``` {.python}
>>> a = raw_input("how old are you?")
how old are you?23
>>> print a
23
```

读入参数：

``` {.python}
from sys import argv

script, first, second = argv

print script
print first
print second
```

在编译的时候要这样：

``` {.example}
python test.py fst snd
```

这的文件名是test.py,传了后面2个参数，结果为：

``` {.example}
test.py
fst
snd
```

这东西其实就是C语言的main函数的参数那东西。
sys称为module或者library，这里是从sys里面导入了argv这个东西。

需要更多的学习，找到了[这篇文章](http://www.ibm.com/developerworks/cn/opensource/os-python8/)
,看一下从文件中输入输出。

``` {.python}
>>> myfile = file('test.txt', 'w')
>>> print >> myfile, 'abc'
>>> print >> myfile, 'de'
>>> myfile.close()
```

使用&gt;&gt;符号写到文件，这个和shell的重定向很像，后面接文件对象，逗号，字符串。
多次使用，默认是追加。

有一个地方比较奇特，这里的的文件读写都是在缓冲区完成的，只有当你调用了close()
之后才会写到文件中，刚好适合于我这种经常打开文件不关闭的人\^\_\^

再看从文件中读数据：

``` {.python}
>>> myfile = open('test.txt')
>>> myfile.read()
'abc\nde\n'
>>> str = myfile.read()
>>> print str

>>> myfile.seek(0)
>>> str = myfile.read()
>>> print str
abc
de

>>> myfile.close()
```

读文件的时候，可以不指定任何r, w, a, r+, r+a, b,读取的时候，文件指针
会移动，使用seek来移动文件指针，这跟C语言就是一样的。

但是这个时候就一定要记得关闭文件了。

用readline()读取一和地，用tell获得文件指针当前位置。用write写入文件。
用truncate来清空文件。

函数
====

下面一个例子：

``` {.python}
def print_none():
    print 'nothing'

def print_one(str):
    print str

def print_two(x, y):
    print "x=%d, y=%d" %(x, y)

def print_two_again(*args):
    x, y = args
    print "x=%d, y=%d" %(x, y)

print_none()
print_one('abc')
print_two(1, 2)
print_two_again(1, 2)
```

涉及到函数的定义，首先是def关键字，然后是函数名，然后是括号，括号里面是参数，
后面一个冒号，换行，然后缩进4个空格（必须是4个！），一行代码，接着的每行代码
都要缩进。函数是以没有缩进的一行作为结束的，这叫dedenting。如果没有这一行，
则表示还没结束。

\*args这种星号参数，把所有的参数打包成一个参数，在代码里面解压成多个参数。

如果是带返回值的函数，则是这样：

``` {.python}
def add(a, b):
    """ Add two variables """    
    return a + b

print add(1, 2)
```

其实和C语言是一样的。

要定义自己的模块，则编译完之后，import 去掉.py的文件名，就可以通过
module.function这种形式调用函数了。

第一行中的3个引号注释，则是函数文档，这在help(module.function)时会显示：

``` {.example}
>>> help(test.add)
Help on function add in module test:

add(a, b)
    Add two variables
```

当然是在import之后，也可以这样：

``` {.example}
>>> help(test)
Help on module test:

NAME
    test

FILE
    /home/monkey/note/lpthw/test.py

FUNCTIONS
    add(a, b)
        Add two variables

    print_none()

    print_one(str)

    print_two(x, y)

    print_two_again(*args)
        print two variables again

```

如果模块里面的代码修改了，要重新import，则使用reload，否则还是原来那个模块。

如果不想每次都打模块名，则可以这样：

``` {.example}
from module import *
```

表示从这个模块中导入所有的东西，这样就可以直接用函数名了，但是为了兼容性，还是
用module.function比较好，因为有可能重名。

逻辑
====

在python中，有and, or, not这种关键字式的，也有符号式的（和C一样）。
这里的if，没有括号，最右边是一个冒号，然后是换行，下面的代码都要缩进4个空格。
C语言中的else if在这里变成了elif，这个就和shell一样，elif和else后面都要
加冒号，加了冒号表示下面的代码都要缩进。最好在最后加上一个没有缩进的空行作为
判断的结束。

list和循环
==========

其实就是haskell的那种list, 空list为\[\]，有元素的，元素用逗号分开，而
for循环则为for element in list:这种格式。

``` {.python}
str = ['abc', 'de', 'fgh']

for s in str :
    print s

num = []

for i in range(0, 5) :
    num.append(i)

for i in num :
    print i

```

这里的range会返回一个从start开始到stop结束的list

``` {.python}
>>> range(0, 5)
[0, 1, 2, 3, 4]
```

而这里用到了append，就来看看list里面还有什么操作。找到了[这篇文章](http://www.pythonclub.org/python-basic/list)
,

list中的元素类型还可以不一样，访问的时候就像数组一样，如果下标是-1则会是
最后一个元素，如果是其它负数则依此类推，很明显不会越界。使用del来删除列表
元素。插入则像字符串一样（其实字符串是list），使用冒号。用in来判断某元素
是否在list中：

``` {.python}
>>> arr = ['a', 'b', 1, 2]
>>> print arr
['a', 'b', 1, 2]
>>> print arr[0]
a
>>> print arr[-1]
2
>>> print arr[4]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> print arr[-5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> del arr[0]
>>> print arr
['b', 1, 2]
>>> arr[2:2] = ['a']
>>> print arr
['b', 1, 'a', 2]
>>> arr[3:4] = ['c']
>>> print arr
['b', 1, 'a', 'c']
>>> del arr[3]
>>> del arr[2]
>>> print arr
['b', 1]
>>> arr[3:3] = ['c']
>>> print arr
['b', 1, 'c']
>>> 'b' in arr
True
```

得到长度，当然是len了，遍历就用for。

现在来看看while，其实也是和C差不多的

``` {.python}
i = 0;

while i < 5 :
    print i
    i += 1

```

令我困惑的是居然没有i++这东西的。

对于循环有一个好的规范，只有当死循环时才用while，其它时候用for.

字典
====

看起来是一个新的东西，事实上是所谓的哈希表。事实上是下标可以是任何东西的
数组，其实就是php的数组。list是用方括号括起来的，而字典则是用花括号。

``` {.python}
>>> stuff = { 'name' : 'abc', 'age' : 22 }
>>> print stuff
{'age': 22, 'name': 'abc'}
>>> print stuff['age']
22
>>> del stuff['name']
>>> print stuff
{'age': 22}
```

同样，使用del来删除。

字典支持嵌套。 字典的创建，原先只有用{key : value, key :
value}这种形式，后来又出现了 下面两种：

``` {.python}
>>> stuff = dict((['name', 'abc'], ['port', 80]))
>>> dict
<type 'dict'>
>>> stuff
{'name': 'abc', 'port': 80}
>>> stuff = {}.fromkeys(('x', 'y'), 0)
>>> stuff
{'y': 0, 'x': 0}
```

引用自[这里](http://skyfen.iteye.com/blog/567571) ,常用的操作方法有：

-   has~key~，判断是否有键存在
-   keys,返回所有键组成的list
-   values,返回所有值组成的list
-   items,返回所有条目组成的list
-   update, 合并字典
-   clear, 清除字典所有条目

``` {.python}
>>> stuff = { 'host' : 'localhost', 'port' : 3306 }
>>> stuff
{'host': 'localhost', 'port': 3306}
>>> stuff.has_key('host')
True
>>> stuff.has_key('abc')
False
>>> stuff.keys()
['host', 'port']
>>> stuff.values()
['localhost', 3306]
>>> stuff.items()
[('host', 'localhost'), ('port', 3306)]
>>> another = stuff
>>> another
{'host': 'localhost', 'port': 3306}
>>> stuff.update(another)
>>> stuff
{'host': 'localhost', 'port': 3306}
>>> another
{'host': 'localhost', 'port': 3306}
>>> another = { 'user' : 'monkey' }
>>> stuff.update(another)
>>> stuff
{'host': 'localhost', 'port': 3306, 'user': 'monkey'}
>>> another.clear()
>>> another
{}
```

类
==

看下面的代码：

``` {.python}
class User(object) :
    name = ''
    age = 18

    def __init__(self) :
        name = "monkey"
        age = 22

    def __del__(self) :
        pass

    def show(self) :
        print "name:%s, age:%d" %(self.name, self.age)

user = User()
user.show()

```

class 类名(父类名):

如果父类不是自己定义的，则以object作为父类，或者不用写，直接为：

class 类名 :

但是好的规范是把object作为父类。

成员变量的定义，全部要初始化（在python里面变量都要初始化，这是比较好的），
默认的变量全部为公有变量，如果要定义私有变量和函数，则只需要在变量名前面
加上2个下划线\_\_

\_\_init\_\_为构造函数，\_\_del\_\_为析构函数，所有的函数都要传至少一个
参数，第一个参数的名字可以自定义，但是最好定义为self，因为历史原因，如果你
不这样定义，人家会骂死你，这个self就是c++里面的this，在内部引用变量和函数
都要用self.variable或者self.method。

创建对象，直接用

对象名 = 类名()

这种形式。

pass为空操作。

项目框架
========

项目目录下要有4个目录：

``` {.example}
bin NAME tests docs 
```

NAME为主模块的名字，在NAME和tests目录下都要创建一个\_\_init\_\_.py文件。

在项目根目录下创建setup.py文件，里面的内容类似于：

``` {.python}
try:
    from setuptools import setup
except ImportError:
    from distutils.core import setup

config = {
    'version' : '1.01',
    'name' : 'Project name',
    'description' : 'Some discriptions',
    'author' : 'My name',
    'author_email' : 'My email',
    'url' : 'URL to visit it',
    'download_url' : 'URL to download it',
    'install_requires' : ['nose'],
    'packages' : ['NAME'],
    'scripts' : []
}    

setup(**config)
```

然后在tests目录下创建一个NAME~tests~.py文件，里面的内容类似于：

``` {.python}
from nose.tools import *
import NAME

def setup() :
    print "test setup"

def test_basic() :
    print "..."

```

python有专门的包管理工具pip，nose是用于测试的，可以用pip来安装。在根目录下
使用nosetests命令来运行测试。
