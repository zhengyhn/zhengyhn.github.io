---
date: 2022-08-17
title: types and typeclasses
tags: ['haskell']
categories: ['haskell']
---

概述
====

-   haskell有类型推理机制，如果我们写一个数字，我们不用告诉编译器它是一个数字，

编译器能够推理出它是一个数字。

-   使用:t命令可以查看一个表达式的类型。

``` {.haskell}
ghci> :t 'a'
'a' :: Char
ghci> :t False
False :: Bool
ghci> :t "aaa"
"aaa" :: [Char]
ghci> :t ('a', "aaa")
('a', "aaa") :: (Char, [Char])
```

::读作has type of 我们看到"aaa"有类型\[Char\]，表明是一个list
而元组('a', "aaa")有类型(char, \[char\]) 类型的首字母都是大写

-   函数也有类型，看下面这个函数

``` {.haskell}
remove_uppercase str = [ ch | ch <- str, ch `elem` ['a'..'z']]
```

查看它的类型：

``` {.haskell}
ghci> :t remove_uppercase
remove_uppercase :: [Char] -> [Char]
```

如果带多个参数会怎么样呢？

``` {.haskell}
my_sum a b c = a + b + c
```

这样，会得到下面的结果：

``` {.haskell}
ghci> :t my_sum
my_sum :: Num a => a -> a -> a -> a
```

这是什么乱七八糟的呢？先不管，我们给这个函数声明类型：

``` {.haskell}
my_sum :: Int -> Int -> Int -> Int
my_sum a b c = a + b + c
```

再来看一下：

``` {.haskell}
ghci> :t my_sum
my_sum :: Int -> Int -> Int -> Int
```

原来多个参数的类型是这样表示的。

一些基本的数据类型
==================

-   Int

用来表示整数，它是有界的，比如32位的环境下是-2 \^ 31 - 1至2 \^ 31

-   Integer

也是用来表示整数的，但是它是无界的，于是它可以用来表示很大的数字，
虽然很大，但是你懂的，不可能无限大。可以想到，有界的Int要高效一 些。

-   Float

表示单精度浮点数

-   Double

表示双精度浮点数

-   Bool

boolean类型，只有两个值True和False

-   Char

字符

类型变量(type variable)
=======================

操作list中有一个函数叫head，它的类型是：

``` {.haskell}
Prelude> :t head
head :: [a] -> a
```

由于a不是以大写字母开头的，所以它不是一个类型，而事实上它是一个类型变量（
type variable)，表明a可以是任意类型，这个有点像C++中的泛型。存在类型变量
的函数称为多态函数(polymorphic function)。 再来看看fst函数：

``` {.haskell}
Prelude> :t fst
fst :: (a, b) -> a
```

可以看出，类型变量使用小写字母a, b, c, d...来表示。

类型类(typeclass)
=================

一个类型类是一个定义了一些动作的接口，如果一个类型是一个类型类的一部分，则这
个类型实现了这个类型类定义的一些动作。

``` {.haskell}
Prelude> :t (==)
(==) :: Eq a => a -> a -> Bool
```

这里，=&gt;右边表明==这个函数接受2个任意类型的参数，返回Bool类型，左边表明参数
a必须是Eq类的成员。左边的称为类约束(class constraint)。 再看看elem：

``` {.haskell}
Prelude> :t elem
elem :: Eq a => a -> [a] -> Bool
```

下面是一些基本的typeclass：

-   Eq，用于比较相等，它里面的成员实现的函数有=`和/`
-   Ord，用于操作顺序，它的成员实现的函数有&gt;, &lt;, &gt;=,
    &lt;=和compare

``` {.haskell}
Prelude> 5 > 2
True
Prelude> compare 5 3
GT
```

compare接受2个相同类型的参数，返回类型Ordering.Ordering可以为GT，LT或者EQ。
要成为Ord的成员，必须要先是Eq的成员。

-   Show，用于显示字符串，它的成员实现的函数主要是show。

``` {.haskell}
Prelude> show 1.23
"1.23"
```

-   Read，用于读取一个字符串,它的成员实现的函数主要是read，读取一个字符串，返回

一个在Read成员中有的类型。

``` {.haskell}
Prelude> read "1.2" + 2.1
3.3
```

它是怎么知道是哪个类型的呢？看下面的例子：

``` {.haskell}
Prelude> read "2"

<interactive>:19:1:
    No instance for (Read a0) arising from a use of `read'
    The type variable `a0' is ambiguous
    Possible fix: add a type signature that fixes these type variable(s)
    Note: there are several potential instances:
      instance Read () -- Defined in `GHC.Read'
      instance (Read a, Read b) => Read (a, b) -- Defined in `GHC.Read'
      instance (Read a, Read b, Read c) => Read (a, b, c)
        -- Defined in `GHC.Read'
      ...plus 25 others
    In the expression: read "2"
    In an equation for `it': it = read "2"
```

它提示错误了，因为它不知道是哪个类型。看一下read函数的类型：

``` {.haskell}
Prelude> :t read
read :: Read a => String -> a
```

它返回了a，表明可以是任意变量，而上例中两个数相加，是因为编译器猜到了类型，
如果要用read函数，则最好是给它强制转换类型,通过加上::符号：

``` {.haskell}
Prelude> read "2" :: Int
2
```

-   Enum，它的成员是顺序的有序类型，有2个函数：succ和pred，用来求某个元素

的successor和predecesor。它里面的成员有：(), Bool, Char, Ordering, Int,
Integer, Float, Double。其中()是空元组类型，它们都是可以枚举的。

-   Bounded，它的成员都有一个上界和一个下界，有2个有趣的函数，minBound和

maxBound，看下例：

``` {.haskell}
Prelude> :t minBound 
minBound :: Bounded a => a
Prelude> minBound :: Int
-9223372036854775808
Prelude> :t maxBound
maxBound :: Bounded a => a
Prelude> maxBound :: Char
'\1114111'
```

所有的元组都是它的成员：

``` {.haskell}
Prelude> minBound :: (Bool, Char, Int)
(False,'\NUL',-9223372036854775808)
```

-   Num，顾名思义，它是一个数字的类型类。

一个数字就是一个多态常量：

``` {.haskell}
Prelude> :t 2
2 :: Num a => a
Prelude> 2 :: Int
2
Prelude> 2 :: Integer
2
Prelude> 2 :: Float
2.0
Prelude> 2 :: Double
2.0
```

要成为Num的成员，必须要先成为Show和Eq的成员。

-   Integral，Num包括所有的数字类型，而Integral只包含整数类型（Int和Integer）。

一个有用的函数是fromIntegral，它的类型为：

``` {.haskell}
Prelude> :t fromIntegral
fromIntegral :: (Integral a, Num b) => a -> b
```

注意到一个奇怪的地方，它的类约束有2个，用逗号隔开。

-   Floating，只包括浮点数类型(Float, Double)。

