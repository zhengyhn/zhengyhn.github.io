---
date: 2013-08-13
title: elisp笔记：运算
---

声明：此文章为学习[xahlee](http://www.xahlee.org)
的[elisp教程](http://ergoemacs.org/emacs/elisp_basics.html) 而写的笔记。

数学计算基础
------------

### 运算符也是函数

和其它函数式语言一样，elisp中，加减乘除也是一个函数。我做了下面的练习：

``` {.commonlisp}
ELISP> (+ 1 1)
2
ELISP> (+ 1 2 3)
6
ELISP> (+ 1 2 3 4)
10
ELISP> (- 4 1)
3
ELISP> (- 4 2 1)
1
ELISP> (* 3 3)
9
ELISP> (/ 2 1)
2
ELISP> (/ 1 2)
0
ELISP> (/ 1.0 2)
0.5
ELISP> (% 5 3)
2
ELISP> (expt 2 10)
1024
```

### 判断对象的类型

规律为：类型+p，如：

``` {.commonlisp}
(integerp 1)
```

用于判断1是不是整数,其中p表示predicate（是）。我做了下面的练习：

``` {.commonlisp}
ELISP> (integerp 3.)
t
ELISP> (integerp 3)
t
ELISP> (floatp 3.1)
t
ELISP> (stringp "")
t
ELISP> (listp '(1 2))
t
```

### 数字与字符串互转

可以猜到函数，数字转字符串为number-to-string，字符串转数字为string-to-number：

``` {.commonlisp}
ELISP> (string-to-number "1")
1
ELISP> (number-to-string 1.0)
"1.0"
```

数字操作
--------

### 整数
