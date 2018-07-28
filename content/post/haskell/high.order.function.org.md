---
date: 2013-03-11
title: 高阶函数
---

curried function
================

我还是直接翻译成“用咖啡煮的函数吧”。 有一个有趣的现象：

``` {.haskell}
Prelude> max 1 2
2
Prelude> (max 1) 2
2
```

我刚开始觉得这太不可思议了，max
1居然可以执行，而且它的返回值还拿2来当参数！
原来这个函数可以写成下面这两种形式：

``` {.haskell}
Prelude> :t max
max :: Ord a => a -> a -> a
Prelude> :t max
max :: Ord a => a -> (a -> a)
```

第二种，如果给max传一个参数x，则它返回一个函数，这个函数会接受一个参数y返回和x
比较之后的最大值。
这意味着我们可以传部分参数给一个函数。是不是只有库函数才这样的呢？不是！

``` {.haskell}
-- addThree - adds three numbers, returns their sum
addThree :: (Num a) => a -> a -> a -> a
addThree x y z = x + y + z            
```

运行结果：

``` {.haskell}
*Main> addThree 1 2 3
6
*Main> ((addThree 1) 2) 3
6
```

这令我非常震惊！ 中缀函数也能有这种用法：

``` {.haskell}
{- isUpper - if a character is uppercase, return True,
             otherwise, return False.
-}
isUpper :: Char -> Bool
isUpper = (`elem` ['A'..'Z'])
```

虽然只有一个参数，但是也能看到基本的形式是加一个括号，称为section。

其它的高阶函数
==============

函数可以接受函数作为参数，也可以返回函数。
实现一个标准库函数zipWith，接受一个函数，两个list，根据这个函数的作用返回
2个list合并后的效果。

``` {.haskell}
zip_with :: (a -> b -> c) -> [a] -> [b] -> [c]
zip_with _ [] _ = []
zip_with _ _ [] = []
zip_with func (xFirst:xRest) (yFirst:yRest) =
    (func xFirst yFirst) : (zip_with func xRest yRest)
```

使用一下：

``` {.haskell}
*Main> zip_with (*) [1, 2] [3, 4]
[3,8]
*Main> zip_with max [1, 2] [3, 4]
[3,4]
```

这就是通用的函数，太厉害了！

一些库函数
==========

map
---

接受一个函数和一个list作为参数，把这个函数作用于list中的每个函数。

``` {.haskell}
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (first:rest) = f first : map f rest
```

关键是它的应用，这个函数用起来非常方便。

``` {.haskell}
*Main> map fst [(1, 2), (3, 4)]
[1,3]
*Main> map (* 2) [1, 2, 3]
[2,4,6]
```

filter
------

顾名思义，是用来过滤的，接受一个predicate，即判断函数，和一个list，返回一个
满足条件的list。

``` {.haskell}
filter :: (a -> Bool) -> [a] -> [a]
filter _ [] = []
filter predicate (first:rest)
    | predicate first = first : filter predicate rest
    | otherwise = filter predicate rest
```

应用：

``` {.haskell}
*Main> filter' (> 3) [1, 2, 3, 4]
[4]
```

上面这2个函数都可以通过list
comprehension来代替。但是使用这样的函数会更容易 理解，可读性更强。

lambdas
=======

其实就是匿名函数(anonymous function)。比如： map (+3) \[1, 2, 3\] 和 map
(\x -&gt; x + 3) \[1, 2, 3\]是等价的。
使用\来表示匿名函数，上例中，传入参数x，返回值为x + 3。
