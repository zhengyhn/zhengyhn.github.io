---
date: 2013-03-10
title: 递归
---

一些基本的递归函数
==================

在haskell中，没有循环，有的只是函数，要完成一些事情，只能使用递归来做，而且
完成得比较好。看下面的求最大值函数：

``` {.haskell}
-- function maxi, to get the maximum value of a list
maxi :: (Ord a) => [a] -> a
maxi [] = error "empty"
maxi [only] = only
maxi (first:rest)
     | first > maxRest = first
     | otherwise = maxRest
     where maxRest = maxi rest
```

使用max函数，还可以简化这个函数：

``` {.haskell}
-- function maxi, to get the maximum value of a list
maxi :: (Ord a) => [a] -> a
maxi [] = error "empty"
maxi [only] = only
maxi (first:rest) = max first (maxi rest)
```

再来看一个很牛逼的反转list的函数：

``` {.haskell}
-- function rever, to reverse a list
rever :: [a] -> [a]
rever [] = []
rever (first:rest) = (rever rest) ++ [first]
```

快排
====

以前听人家说函数式编程语言，排序只要一行代码，我当时就笑话他，它肯定是用
sort了，我用C++也是直接sort，也是一行代码。但是现在看到用haskell实现的
快排，把算法都实现了，效率完全不亚于其他语言。

``` {.haskell}
-- function qsort, sort a list using the quick sort algorithm
-- in ascending order
qsort :: (Ord a) => [a] -> [a]
qsort [] = []
qsort (first:rest) =
      let smallerPart = qsort [elem | elem <- rest, elem < first]
          largerPart = qsort [elem | elem <- rest, elem >= first]
      in smallerPart ++ [first] ++ largerPart
```
