---
date: 2013-03-10
title: tcpl读书笔记：结构体
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

基础
====

-   原来可以这样初始化：

``` {.c}
struct point {
    int x;
    int y;
};

struct maxpt = { 300, 200 };
```

这样就定义了一个坐标为(300, 200)的点。

-   在定义的时候初始化：

``` {.c}
struct key {
    char *word;
    int count;
} keytab[] = {
    "break", 0,
    "continue", 0
};
```

-   sizeof不能用在\#if里面，因为预处理器不能解释类型名
-   指针的加法是非法的，而指针的减法是合法的。

typedef
=======

-   看下面这个例子：

``` {.example}
typedef int (*PFI)(char *, char *);
```

居然可以这样用！上面的代码起了一个类型别名，用于表示一个指向一个函数的指针，
这个函数有2个char \*参数，返回类型为int。
看到这里，我终于知道typedef应该怎样理解了。比如：

``` {.example}
typedef struct point *Point;
```

这里，添加的类型别名为Point，当使用它的时候：

``` {.example}
Point pp;
```

当理解的时候，只需要把Point换成pp就行了：

``` {.example}
typedef struct point *pp;
```

这样，就可以解释上面那个函数指针了。

union
=====

-   联合体里面的存储空间等于占用空间最大的元素所占的空间

位域
====

-   看下面的例子：

``` {.c}
#include <stdio.h>

typedef unsigned int uint;

struct {
    uint is_keyword : 1;
    uint is_extern  : 1;
    uint is_static  : 1;
} flags;

int main()
{
    printf("%d\n", sizeof(flags));
    flags.is_keyword = 1;
    printf("%d\n", flags.is_keyword);

    return 0;
}
```

结果为:

  ---
  4
  1
  ---

占用4个字节，而不是12个字节，因为这里只用到3位，为了内存对齐，就是4个字节了。
