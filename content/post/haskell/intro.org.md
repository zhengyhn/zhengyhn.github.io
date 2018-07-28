---
date: 2013-02-16
title: introduction
---

what is haskell
===============

-   haskell是一种纯粹的函数式编程语言
-   没有副作用
-   一种懒惰的语言，只有当要显示结果的时候才执行函数
-   静态类型化，不用指定类型，编译器能自己决定
-   高雅，简洁，代码量少
-   由顶尖人才发明

how to program
==============

-   2个著名的haskell编译器是GHC(glasgow haskell compiler)和Hugs
-   安装完ghc后，在终端打ghci，可以进入互动形式的编程环境，就像python和ielm

``` {.example}
[monkey@itlodge note]$ ghci
GHCi, version 7.6.2: http://www.haskell.org/ghc/  :? for help
Loading package ghc-prim ... linking ... done.
Loading package integer-gmp ... linking ... done.
Loading package base ... linking ... done.
Prelude> 
```

这里的是Prelude&gt;，可以通过

``` {.example}
Prelude> :set prompt "ghci> "
ghci> 
```

来修改成ghci&gt;

play
====

-   来玩点数学运算：

``` {.haskell}
ghci> 1 + 2
3
ghci> 1991 * 1992
3966072
ghci> 1992 - 19
1973
ghci> 1992 / 19
104.84210526315789
ghci> 5 * (-1 - 1)
-10
ghci> 5 * -1

<interactive>:9:1:
    Precedence parsing error
        cannot mix `*' [infixl 7] and prefix `-' [infixl 6] in the same infix
 expression
ghci> 5 * (-1)
-5
ghci> 
```

-   再来玩一点布尔运算

``` {.haskell}
ghci> True && False
False
ghci> False || True
True
ghci> not True
False
ghci> not (False || True)
False
ghci> 1 == 1
True
ghci> 1 == 1.0
True
ghci> 1 /= 1.0
False
ghci> "abc" == "abc"
True
ghci> 
```

-   +和\*这种也是函数，它们是中缀函数，因为夹在两个操作数之间
-   前缀函数，格式如下：

``` {.example}
function-name space parameters
```

如haskell中最无聊的函数succ：

``` {.haskell}
ghci> succ 7
8
ghci> 
```

succ函数用于返回一个对象的successor 再看：

``` {.haskell}
ghci> min 1.2 2
1.2
ghci> max 1991 1991
1991
ghci> max 222 111
222
```

-   函数的优先级最高，看下例：

``` {.haskell}
ghci> min 2 3 + max 4 5 + 2
9
ghci> (min 2 3) + (max 4 5) + 2
9
```

上面两行是等价的。所以，当我们要求4和5 + 2的最大值时，要这样：

``` {.haskell}
ghci> max 4 (5 + 2)
7
```

-   如果一个前缀函数有2个参数，我们也可以把它改成中缀函数。比如div函数，看下例，

我们不知道是100除以10还是10除以100,所以改成中缀函数会清晰很多。方法是给div
加上backtick（即\`号，在我键盘上是和波浪号一个键的）

``` {.haskell}
ghci> div 100 10
10
ghci> 100 `div` 10
10
ghci> 100 / 10
10.0
```

baby's first function
=====================

-   新建一个文件test.hs，写一个函数：

``` {.haskell}
twice x = x + x
```

其中函数名叫twice，只有一个参数x 然后打开ghci编译：

``` {.haskell}
ghci> :l ~/test/haskell/test.hs 
[1 of 1] Compiling Main             ( /home/monkey/test/haskell/test.hs, interpreted )
Ok, modules loaded: Main.
ghci> twice 4
8
```

:l中l是load的意思，把这个script
load进来，然后它会编译，就可以使用这个函数了。

-   带2个参数的是类似这样的：

``` {.haskell}
twice x y = x * 2 + y * 2
```

-   haskell中函数没有调用的顺序之分，不用考虑哪个函数使用哪个函数而要先声明哪个函数。
-   if结构，else是必须要有的。

``` {.haskell}
twice x = if x > 10
             then x
             else x * 2
```

-   撇号的英文：apostrophe
-   撇号用于函数名的时候，可以表示这是个严格（不懒惰）的函数，或者是一个已经

修改过的函数。

-   函数名不能以大写开头
-   没有参数的函数称为定义definition（或名字name）

an intro to lists
=================

列表是齐次的数据结构，存储了相同类型的元素
------------------------------------------

有点像数组，可以使用let来定义一个名字：
---------------------------------------

``` {.haskell}
ghci> let num = [1, 2, 3, 4, 5]
ghci> num
[1,2,3,4,5]
```

可见，list通过方括号(square bracket)来定义，元素由逗号(comma)来分隔开。
但是这样却是不行的：

``` {.haskell}
ghci> let num = [1, 'a']

<interactive>:16:12:
    No instance for (Num Char) arising from the literal `1'
    Possible fix: add an instance declaration for (Num Char)
    In the expression: 1
    In the expression: [1, 'a']
    In an equation for `num': num = [1, 'a']
```

连接两个list，使用++
--------------------

``` {.haskell}
ghci> [1, 2, 3] ++ [4, 5, 6]
[1,2,3,4,5,6]
```

注意，不要用在很长的list里面上，因为haskell会遍历++左边的list。

使用:(cons operator)来把元素放在list的左边
------------------------------------------

``` {.haskell}
ghci> 0 : [1, 2, 3]
[0,1,2,3]
```

\[\]为空list
------------

可以这样连接：
--------------

``` {.haskell}
ghci> 1 : 2 : []
[1,2]
```

通过下标拿到一个元素(下标从0开始)：
-----------------------------------

``` {.haskell}
ghci> [1, 2, 3] !! 1
2
```

list的比较：
------------

``` {.haskell}
ghci> [3, 2, 1] > [3, 1, 2]
True
ghci> [1, 2, 3] /= [1, 2, 3]
False
```

一些基础的对list的操作函数
--------------------------

-   获取头部元素：

``` {.haskell}
ghci> head [1, 2, 3]
1
```

-   获取尾（即去掉第一个元素）：

``` {.haskell}
ghci> tail [1, 2, 3]
[2,3]
```

-   获取尾部元素：

``` {.haskell}
ghci> last [1, 2, 3]
3
```

-   获取头（即去掉最后一个元素）：

``` {.haskell}
ghci> init [1, 2, 3]
[1,2]
```

-   取得长度：

``` {.haskell}
ghci> length [1, 2, 3]
3
```

-   判断是否为空：

``` {.haskell}
ghci> null [1, 2, 3]
False
ghci> null []
True
```

-   翻转

``` {.haskell}
ghci> reverse [1, 2, 3]
[3,2,1]
```

-   从头截断并拿到list

``` {.haskell}
ghci> take 2 [1, 2, 3]
[1,2]
ghci> take 5 [1, 2, 3]
[1,2,3]
ghci> take 0 [1, 2, 3]
[]
```

-   从头截断并丢掉list

``` {.haskell}
ghci> drop 2 [1, 2, 3]
[3]
ghci> drop 5 [1, 2, 3]
[]
ghci> drop 0 [1, 2, 3]
[1,2,3]
```

-   求最大最小值

``` {.haskell}
ghci> maximum [1, 2, 3]
3
ghci> minimum [1, 2, 3]
1
```

-   求和

``` {.haskell}
ghci> sum [1, 2, 3]
6
```

-   求积

``` {.haskell}
ghci> product [2, 3, 4]
24
```

-   判断元素是否在list中

``` {.haskell}
ghci> 2 `elem` [1, 2, 3]
True
ghci> elem 2 [1, 2, 3]
True
ghci> elem 10 [1, 2, 3]
False
ghci> 10 `elem` [1, 2, 3]
False
```

texas ranges
============

-   要构造一个包含数字1-20的list，使用下面的方法：

``` {.haskell}
ghci> [1..20]
[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
```

另外一个例子：

``` {.haskell}
ghci> ['a'..'z']
"abcdefghijklmnopqrstuvwxyz"
```

还能按照规律枚举

``` {.haskell}
ghci> [1, 3..20]
[1,3,5,7,9,11,13,15,17,19]
```

只能指定一步，下面这样是不行的：

``` {.haskell}
ghci> [1, 2, 4, 8..100]

<interactive>:7:12: parse error on input `..'
```

如果要降序，这样是不行的：

``` {.haskell}
ghci> [20..1]
[]
```

可以这样：

``` {.haskell}
ghci> [20, 19..1]
[20,19,18,17,16,15,14,13,12,11,10,9,8,7,6,5,4,3,2,1]
```

最好不要对浮点数这样操作，看下例：

``` {.haskell}
ghci> [0.1, 0.3..1]
[0.1,0.3,0.5,0.7,0.8999999999999999,1.0999999999999999]
```

可以指定无穷list，看下例：

``` {.haskell}
ghci> take 24 [13, 26..]
[13,26,39,52,65,78,91,104,117,130,143,156,169,182,195,208,221,234,247,260,273
,286,299,312]
```

上面获得的是前24个元素。
我试了一下无穷地列举，果然它会一直列下去，不管CPU怎么样。
看看cycle函数，循环列举

``` {.haskell}
ghci> take 5 (cycle [1, 2, 3])
[1,2,3,1,2]
```

再看repeat函数，会重复一个元素

``` {.haskell}
ghci> take 5 (repeat 1)
[1,1,1,1,1]
```

replicate函数也有同样的功能：

``` {.haskell}
ghci> replicate 3 1
[1,1,1]
```

I'm a list comprehension
========================

-   这东西就像集合的表示一样，看下例：

``` {.haskell}
ghci> [x * 3 | x <- [1..10]]
[3,6,9,12,15,18,21,24,27,30]
```

更神奇的在这里：

``` {.haskell}
ghci> [x * 3 | x <- [1..10], x * 3 >= 10]
[12,15,18,21,24,27,30]
```

还可以用来排列组合：

``` {.haskell}
ghci> [x * y | x <- [1, 2, 3], y <- [4, 5]]
[4,5,8,10,12,15]
```

巧妙地用来计算字符串的长度：

``` {.haskell}
len s = sum [1 | _ <- s]
```

调用的时候：

``` {.haskell}
ghci> len "abc"
3
```

解释一下，~代表用不到的变量名~，这里，把每个变量用1来代替，然后求和，得到
的即为字符串的长度。

tuples
======

元组，可以包含不同类型
以括号括起来，逗号分隔开，比如一个包含我的名字，网站，年龄的元组为：

``` {.haskell}
ghci> ("yuanhang", "itfoot", 23)
("yuanhang","itfoot",23)
```

下面是一些基本的函数：

-   fst，对于一个二元元组，返回第一个(first)
-   snd，对于一个二元元组，返回第二个(second)

``` {.haskell}
ghci> fst (2, 1)
2
ghci> snd (2, 1)
1
```

-   zip，把2个list拉在一起，变成一个包含二元元组的list

``` {.haskell}
ghci> zip [1, 2, 3] [3, 2, 1]
[(1,3),(2,2),(3,1)]
```

如果两个list元素个数不同呢？长的list会只使用一部分元素

``` {.haskell}
ghci> zip [1..] ['a', 'b', 'c']
[(1,'a'),(2,'b'),(3,'c')]
```

幂运算居然直接使用^号^，来个计算三角形的：

``` {.haskell}
ghci> let triangle = [(a, b, c) | a <- [1..10], b <- [1..10], c <- [1..10], a ^ 2 + b ^ 2 == c ^ 2]
ghci> triangle
[(3,4,5),(4,3,5),(6,8,10),(8,6,10)]
```
