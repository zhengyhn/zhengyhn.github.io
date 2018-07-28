---
date: 2013-03-10
title: tcpl读书笔记：输入和输出
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

错误处理——stderr and exit
=========================

-   为什么要有stderr呢？

当我们用到屏幕的时候，如果出错了，可以通过stdout把错误信息输出到屏幕，但是，
如果我们不是打印到屏幕，而是重定向到文件或者其它文件，就不能使用printf等函数
来输出错误信息了。这时候就该stderr上场了，因为它只会输出到屏幕。

``` {.c}
#include <stdio.h>

int main()
{
    fprintf(stderr, "slfdjdfj\n");

    return 0;
}
```

结果如下： \[monkey@itlodge test\]\$ ./t slfdjdfj \[monkey@itlodge
test\]\$ ./t &gt; t.txt slfdjdfj

-   对于exit(int status)函数，status为0表示成功，非0表示异常退出。
-   命令执行函数system(char
    \*s),传进去一个字符串，表示命令，会执行这个命令。

但是这个命令强烈依赖于系统，比如：

``` {.example}
system("date");
```

在Linux会打印出来日期信息，而在win中可能不行。
