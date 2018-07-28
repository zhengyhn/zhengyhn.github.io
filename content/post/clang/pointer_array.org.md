---
date: 2013-03-10
title: tcpl读书笔记：指针和数组
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

多维数组
========

-   对于一个二维数组，如果要作为函数参数来传递，必须要包含列数，行数可加可不加。

因为，传递过去的是指向行的指针。比如,可以这样：

``` {.example}
fun(int arr[2][13])
{
    ...
}
```

也可以这样：

``` {.example}
fun(int arr[][13])
{
    ...
}
```

还可以这样：

``` {.example}
fun(int (*arr)[13])
{
    ...
}
```

注意，\*arr一定要加括号，因为\[\]的优先级比\*要高。

命令行参数
==========

-   看下面的代码：

``` {.c}
#include <stdio.h>

int main(int argc, char *argv[])
{
    ...

    return 0;
}
```

其中argc代表argument count，argv代表argument vector。
argv\[0\]是程序的名字,因此argc至少为1。

复杂的表示
==========

``` {.example}
- int (*p)[13]，指向int类型13个元素的数组的指针
- int *p[13]，一个包含13个int类型的指针的数组
```
