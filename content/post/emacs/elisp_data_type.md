---
date: 2022-08-17
title: elisp笔记：introduction
tags: ['emacs', 'elisp']
categories: ['emacs']
---

lisp历史
========

-   原来lisp是list processing
    language,1950s后期推出，主要为了研究人工智能。

数据类型
========

概述
----

-   每个对象至少属于一种类型
-   对于打印出来的对象，一般都可读，但是有一些是不可读的.这些对象以hash表示法表示。

即以\#&lt;开头，以&gt;结束。比如：

``` {.example}
(current-buffer)
#<buffer *scratch*>
```

-   在elisp中有两大类类型
    1.  用于lisp编程的类型
    2.  用于emacs编辑的类型

整型
----

-   在32位机器上，整型是30位，即-2 \^ 29到2 \^ 29 -
    1，64位的会提供更多，但是官方

文档没有说明。

-   elisp中不会检测溢出。
-   整数的表示法一般为：

(sign)digit(period) 即，1可以表示成+1,1,1.,+1.

-   如果一个数字太小或太大，不在整数范围内，则会当成浮点数来存储。

浮点型
------

-   使用C语言中的double来存储，实际上对于十进制浮点数内部是2的幂而不是10的幂。

字符类型
--------

-   字符其实就是一个整数
-   在字符串和缓冲区中，字符的范围是0-4194303(22位整数),其中0-127个是ASCII码，而

剩下的是非ASCII码。

-   在C语言中，字符以'a'这种方式表示，而在elisp中，以?a这种方式表示

``` {.example}
ELISP> ?a
97
```

这种方式也可以用来表示标点符号(punctuation)字符，但是最好加一个\，因为有的字符
需要转义。

``` {.example}
ELISP> ?\\
92
ELISP> ?\(
40
```

-   部分字符的表示方法

  character presentation   corresponding integer   character
  ------------------------ ----------------------- ----------------- ---
  ?\a                      7                       C-g
  ?\b                      8                       backspace
  ?\t                      9                       tab
  ?\n                      10                      newline
  ?| 11                    vertical tab
  ?\f                      12                      formfeed
  ?\r                      13                      carriage return
  ?\e                      27                      ESC
  ?\s                      32                      space
  ?\                       92                      backslash
  ?\d                      127                     DEL


