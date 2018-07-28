---
date: 2013-03-06
title: 函数中的语法
---

模式匹配(pattern matching)
==========================

这里的模式匹配有点像switch...case...下面的函数判断传入的参数是不是7

``` {.haskell}
lucky :: (Integral a) => a -> String
lucky 7 = "yes"
lucky x = "no"
```

这个函数会从上而下判断，先看是不是7,如果是，则返回"yes"，再看下一条规则，
这里的x表示任何变量（可以用任何名字表示）。
可以想到，可以使用这个办法来实现递归：

``` {.haskell}
fact :: (Integral a) => a -> a
fact 0 = 1
fact n = n * (fact (n - 1))
```

注意：如果找不到匹配项，则程序会崩溃。
模式匹配也可以用于list中，我们知道在list中，:号可以用于在前面加入元素，如：

``` {.haskell}
1 : [2, 3]
```

结果为：

``` {.haskell}
[1, 2, 3]
```

模式x:xs用于表示把list的头绑定在x上，剩下的绑定在xs上，看下例：

``` {.haskell}
-- function head'
head' :: [a] -> a
head' [] = error "list empty!"
head' (x:_) = x
```

又学习到了注释的写法，单选注释是--，多行注释是{- -}，如：

``` {.haskell}
-- comment
{- ---------------------------
-- comment.hs 
-- Author: yuanhang zheng
--------------------------- -}
```

有一种给模式起别名的表示方法，使用@，下次使用的时候就不用重新写这个模式了,
这叫as pattern

``` {.haskell}
-- function first
first :: String -> String
first "" = "empty!"
first all@(x:rest) = "first of " ++ all ++ "is" ++ [x]
```

guard
=====

翻译成“哨兵”应该是不错的。
它就和if...else差不多，判断一个布尔表达式，使用管道符|,pipe，从上往下检查，
一般最后一个就是otherwise，看下例：

``` {.haskell}
-- function checkAge
checkAge :: (Integral a) => a -> String
checkAge age
  | age < 18             = "too young!"
  | age < 25             = "fine!"
  | age < 40             = "old."
  | otherwise            = "too old!"               
```

guard和pattern
matching有点类似，唯一不同的是guard检查的是一个布尔表达式， 而pattern
matching检查的是一个模式。

where语句
=========

我们来写一个函数，传入收入和成本，计算是否有利润，这个程序太简单了：

``` {.haskell}
-- function showProfit
showProfit :: (RealFloat a) => a -> a -> String
showProfit cost income
  | (income - cost) < 0  = "oh no!the profit is minus"
  | (income - cost) == 0 = "come on,zero profit"
  | otherwise            = "congratulation!Your profit is good"
```

但是，这里有个问题，(income -
cost)这个东西重复了很多次，程序员最忌的就是
重复了，在其它语言里面，可以用一个变量来代替，这里我们也可以用，于是出来了where
语句：

``` {.haskell}
-- function showProfit
showProfit :: (RealFloat a) => a -> a -> String
showProfit cost income
  | profit < 0  = "oh no!the profit is minus"
  | profit == 0 = "come on,zero profit"
  | otherwise   = "congratulation!Your profit is good"
  where profit = income - cost                           
```

有点类型于C\#中的where语句。 当然后面还可以加更多变量：

``` {.haskell}
where profit = income - const
      one = 1
      two = 2
```

这些变量只属于这个函数，不影响其它函数。所有的变量都要对齐到同一列。
where语句后面还能定义函数。

let语句
=======

它和where非常像，

``` {.haskell}
-- function cylinder_volume
cylinder_volume :: (RealFloat a) => a -> a -> a
cylinder_volume radius height =
    let area = pi * radius * radius
    in  area * height
```

主要形式为let ... in
...在let后面定义变量，在in中使用这些变量。where是放到
后面的，let是放到前面的。let绑定的是表达式，而where绑定的是语法结构。
if和let还有下面这种奇葩的用法：

``` {.haskell}
*Main> [if 1 > 0 then "abc" else "cba"]
["abc"]
*Main> (let a = 1 in a + a) * 2
4
```

let后面如果有多个变量，并且在同一行，则用分号分开，最后一个变量后面可以加分号也
可以不加。

``` {.haskell}
*Main> (let a = 1; b = 2; c = 3 in a * b * c) + 1
7
```

有时候，in可以忽略。

case语句
========

和C语言中的case很像：

``` {.haskell}
-- function start
start :: [a] -> a
start s = case s of [] -> error "empty"
                    (x:_) -> x
```

注意后面两个情况的代码一定要对齐。
