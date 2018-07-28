---
date: 2013-03-01
title: c陷井和缺陷
---

@&lt;font color="red"&gt; 本文为"c traps and pitfalls"一书的笔记
@&lt;/font&gt;

词法陷井
========

-   看下面的代码：

``` {.example}
y = x/*p;  /* p points to the divisor */
```

本意是x除以p指向的变量，而事实上这样写/\*会认为是注释的开头。
所以，双目运算符两边加空格是非常好的。

语法陷井
========

-   !=的优先级要比&之类的逻辑运算符高，所以不要有下面的写法

``` {.example}
if (flags & FLAG != 0)
```

-   +号等运算符优先级要比[]{#和}的高，所以不要有下面的写法

``` {.example}
r = h << 4 + 1
```

-   运算符优先级的记忆方法

1.  优先级最高的是那些不是“真的”运算符：函数调用()，数组下标\[\]，

结构体成员-&gt;和.

1.  然后是一元运算符，! \~ ~~+ -- 正号~~ 负号- 指针\* 类型转换(type)
    sizeof，

所以使用函数指针的时候，一定要这样：(\*p)()。记住，所有的一元运算符都是右结合的。
所以\*p++会被认为是\*(p++)

1.  接着是二元运算符，它们有下面的规律：

代数运算符 &gt; 移位运算符 &gt; 关系运算符 &gt; 逻辑运算符 &gt;
赋值运算符 &gt; 条件运算符

链接陷井
========

-   假设一个文件里面有下面的定义：

``` {.example}
char filename[] = "/etc/passwd";
```

另外一个文件里面有下面的定义：

``` {.example}
char *filename;
```

这2个变量的类型一样吗？虽然指针和数组很相似，但是它们不是同一样东西。
第一个filename，它是一个字符串数组的名字，使用这个名字会生成一个指向这个
字符串第一个字符的指针，仅在需要的时候生成，不会一直存在。
第二个filename，它是一个指针，定义时就给它分配了空间，程序员想要让它指向
哪里它就会指向哪里，如果不给它赋值，它会指向NULL

语义陷井
========

-   表达式的顺序陷井。看下面的例子：

``` {.c}
#include <stdio.h>

int main()
{
    int a[] = { 1, 2, 3 };
    int b[] = { 10, 20, 30};
    int i = 0;

    while (i < 3) {
        a[i++] = b[i];
    }
    for (i = 0; i < 3; i++) {
        printf("%d\n", a[i]);
    }

    return 0;
}
```

会输出什么呢？我刚开始是这样想的，先计算右边，再赋值给左边，所以会输出
10 20 30 作者再给了一个例子：

``` {.c}
#include <stdio.h>

int main()
{
    int a[] = { 1, 2, 3 };
    int b[] = { 10, 20, 30};
    int i = 0;

    while (i < 3) {
        a[i] = b[i++];
    }
    for (i = 0; i < 3; i++) {
        printf("%d\n", a[i]);
    }

    return 0;
}
```

会输出什么呢？根据我刚才的推理，理应数组越界的，因为赋值给最后一个元素的时候，
i++，i已经超过了数组a的范围了。而事实上，结果还是 10 20 30
作者的解释是，编译器无法保证获取数组元素的操作先于i自增的操作，这是不确定的。
所以在写程序的时候，最好分开写。

库函数陷井
==========

-   getchar()函数居然返回int!下面是从&lt;stdio.h&gt;复制过来的代码：

``` {.c}
/* Read a character from stdin.  */
__STDIO_INLINE int
getchar (void)
{
  return _IO_getc (stdin);
}
```

而且putchar居然也是传进int参数，返回int参数

``` {.c}
/* Write a character to stdout.  */
__STDIO_INLINE int
putchar (int __c)
{
  return _IO_putc (__c, stdout);
}
```

预处理陷井
==========

-   宏不是函数，看下面的例子：

``` {.c}
#define max(a,b) ((a) > (b) ? (a) : (b))

largest = max(largest,x[i++]);
```

虽然，在定义宏的时候很小心，加了很多括号，不至于有表达式优先级的问题，但是
这样调用还是背离了本义，因为x\[i++\]，会使i自加多次，因为宏只是替换，而不是
拷贝参数。

-   宏不是类型定义，看下面的例子：

``` {.c}
#define STU struct student *
typedef struct student *STUDENT;

STU a, b;
STUDENT c, d;
```

后面的两条定义等价于：

``` {.c}
struct student *a, b;
struct student *c, *d;
```

显然，第一条背离了本意，b定义成了结构体类型，而不是指向它的指针类型。虽然
typedef的定义看起来有点别扭，但是在实际的代码中还是要使用typedef，不要
使用宏。
