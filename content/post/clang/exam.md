---
date: 2013-04-20
title: 复习C/C++
---

@&lt;font color="red"&gt; 本文为《C笔试面试宝典》一书的笔记
@&lt;/font&gt;

new, delete, malloc, free的关系
===============================

new和delete是C++的运算符，new调用构造函数，delete调用析构函数。

delete和delete\[\]的区别
========================

delete只会调用一次析构函数，delete\[\]则会调用每个成员的析构函数。

写程序一测：

    #include <iostream>
    #include <string>

    class Computer {
     private:
      std::string name;
     public:
      Computer();
      ~Computer();
    };

    Computer::Computer()
    {

    }

    Computer::~Computer()
    {

    }

    int main(int argc, char *argv[])
    {
      Computer *cs = new Computer[5];
      Computer *c = new Computer;
      int *integers = new int[5];
      int *i = new int;

      delete[] i;
      delete[] integers;
      delete[] cs;
      delete[] c;

      return 0;
    }

发现编译可以通过，但是运行会Segment Fault。gdb一下，发现在delete\[\]
c时出错了。

原因是，对于基本数据类型，delete和delete\[\]的功能是相同的，对于自定义的类型，则
要严格区分，new\[\]完之后用delete\[\]释放，new完之后用delete释放。

继承和组合的优缺点
==================

引用自[这里](http://www.cnblogs.com/nuaalfm/archive/2010/04/23/1718453.html)
。

  组合 关 系                                                         继 承 关 系
  ------------------------------------------------------------------ ------------------------------------------------------------------------------
  优点：不破坏封装，整体类与局部类之间松耦合，彼此相对独立           缺点：破坏封装，子类与父类之间紧密耦合，子类依赖于父类的实现，子类缺乏独立性
  优点：具有较好的可扩展性                                           缺点：支持扩展，但是往往以增加系统结构的复杂度为代价
  优点：支持动态组合。在运行时，整体对象可以选择不同类型的局部对象   缺点：不支持动态继承。在运行时，子类无法选择不同的父类
  优点：整体类可以对局部类进行包装，封装局部类的接口，提供新的接口   缺点：子类不能改变父类的接口
  缺点：整体类不能自动获得和局部类同样的接口                         优点：子类能自动继承父类的接口
  缺点：创建整体类的对象时，需要创建所有局部类的对象                 优点：创建子类的对象时，无须创建父类的对象

引用
====

不能建立数组的引用。引用没有定义新的变量，不占用内存空间。

关联，聚合和组合
================

-   关联是两个类的一般性关联，如老师和学生。
-   聚合是has-a关系，聚合类不需要对被聚合类负责，用空的菱形表示，实现如下：

<!-- -->

    class A {

    };
    class B {
        A *a;
    };

-   组合是

初始化列表
==========

当类中含有const和引用成员变量时，基类构造函数只能使用初始化列表来初始化，
但是const int& a;这种就可以用赋值的方法。

类型安全
========

c++不是类型安全的，因为不同类型的指针之间可以强制转换。

全局和static变量是在编译阶段分配内存的
======================================

空类
====

当一个类没有任何成员时，大小是1byte，这个字节是用来区分这个类的不同对象的。

逻辑地址 to 物理地址
====================

给出的逻辑地址格式是这样的,
段地址:段内偏移地址，那么真实的地址（物理地址） 是：段地址 \* 10H +
段内偏移地址。当然，这只适合于Intel 8086。

4种类型转换
===========

1.  const\_cast，把const的变量变成非const的，用法：

``` {.example}
新变量  = const_cast<类型>(变量);
```

1.  static\_cast,
    用于基本类型的转换，不能用于无关类型（不是基类与子类）之间的

指针的转换

1.  dynamic\_cast,
    运行时会有安全检查，用于基类与子类之间的转换，常用于多态
2.  reinterpret\_cast，重新解释类型，没有转换，常用于函数指针的转换。

数组作参数
==========

当数组作为参数传递时，它会退化成同类型的指针。

override和隐藏
==============

-   override，基类中必须要有virtual。
-   如果基类函数名没有virtual，子类函数与父类函数签名一样，则称隐藏。
-   不管基类函数名有没有virtual，子类函数名一样，签名不一样，则也称隐藏。

求两个数中的最大的那个数
========================

不能用判断(if, :?, switch)。答案是用abs函数：

``` {.example}
((a + b) + abs(a - b)) / 2
```

而我觉得这种方法不好，因为用到了库函数，库函数里面还可能也要判断，其实
是换汤不换药。

我问了同学，他想到了下面的方法，我觉得很好：

``` {.c}
#include <stdio.h>

int max(int a, int b);

int main(int argc, char *argv[])
{
     int a = 9999, b = 23;

     printf("%d\n", max(a, b));

     return 0;
}

int max(int a, int b)
{
     int c = a - b;
     int flag = (unsigned)c >> (sizeof(int) * 8 - 1);

     return (1 - flag) * a + flag * b;
}
```

根据负数与正数的符号位的不一样，而得出那个数。

打印源文件的文件名和当前行号
============================

在C/C++中，可以用\_\_FILE\_\_和\_\_LINE\_\_，由编译器来识别。

main函数执行完之后还能执行代码？
================================

居然是可以的！&lt;stdlib.h&gt;中有一个奇葩的库函数叫on\_exit，在linux下的man
page中， 定义如下：

``` {.example}
int on_exit(void (*function)(int , void *), void *arg);
```

传进去一个函数指针，和一个参数，这个函数必须是2个参数，分别是int和void\*类型的，
可以调用多个，以LIFO形式执行，
on\_exit在任何地方调用都只会在main函数结束之后 才会执行。

``` {.c}
#include <stdio.h>
#include <stdlib.h>

void one(int status, void *arg);
void two(int status, void *arg);

int main(int argc, char *argv[])
{
     printf("top\n");
     on_exit(two, NULL);
     on_exit(one, NULL);

     printf("It may be the last one.\n");

     return 0;
}

void one(int status, void *arg)
{
     printf("one\n");
}

void two(int status, void *arg)
{
     printf("two\n");
}
```

书上说的是\_onexit，根据我找的资料，这函数应该只有在windows的VC中才有。

判断是由C编译器编译还是由C++编译器编译
======================================

使用一个宏\_\_cplusplus,

``` {.example}
#ifdef __cplusplus
...
#else
...
```

求n个数中第k大的数
==================

我智商低，只能想到普通的办法，就是选择排序的外面循环K次。不过不能因为这样
而找不到工作啊，学习了大牛的算法：

``` {.example}
吸取快排中的思想，随机取一个数，把比它小的数放到左边，比它大的数放到右边，
如果运气非常好，它的下标i刚好是n - k - 1，则它就是第k大的数，如果i小于
n - k - 1，则第k大的数在左边，否则在右边，再分为子问题进行求解。
```

写了很久终于写出来了，要是在面试的时候，估计写不出来。

``` {.c}
#include <stdio.h>
#include <stdlib.h>
#include <stdlib.h>

int get_maxk(int *arr, int n, int k);
void swap(int *arr, int i, int j);

int main(int argc, char *argv[])
{
     int a[10];
     int i;

     srand(time(NULL));
     for (i = 0; i < 10; i++) {
      a[i] = rand();
     }
     for (i = 1; i <= 10; i++) {
      printf("%d\n", get_maxk(a, 10, i));
     }

     return 0;
}

int get_maxk(int *arr, int n, int k)
{
     int pivot, last_left, i;

     if (k > n) {
      fprintf(stderr, "k can't be larger than n\n");
      exit(1);
     } else if (k == n) {
      return arr[0];
     }
     pivot = 0;
     last_left = pivot;
     for (i = 1; i < n; i++) {
      if (arr[i] < arr[pivot]) {
           last_left++;
           swap(arr, last_left, i);
      }
     }
     swap(arr, pivot, last_left);
     if ((n - last_left) == k) {
      return arr[k];
     } else if ((n - last_left) > k) {
      return get_maxk(arr + last_left + 1, n - last_left - 1, k);
     } else {
      return get_maxk(arr, last_left, k - (n - last_left));
     }
}

void swap(int *arr, int i, int j)
{
     int temp;

     if (i == j) {
      return;
     }
     temp = arr[i];
     arr[i] = arr[j];
     arr[j] = temp;
}
```

判断单链接有环
==============

想到这个方法的人就是神！

``` {.example}
用两个指针，一个每次走一步，另外一个每次走两步，如果有环必定重合，
否则走两步的那个指针将到达终点。
```

传值还是传地址
==============

看下面的代码：

``` {.c}
void func(char *a)
{
    a = (char *)malloc(10);
}

int main(void)
{
    char *a = NULL;
    func(a);
}
```

执行完主函数后，a还是NULL！a不是传进去申请了空间吗？我之前也是这样认为的，
后来发现，其实func函数的那个参数是以传值的方式传进去的，而不是传地址！
如果要传地址的话，应该是char \*\*a，func(&a)这样才行！

extern "C"
==========

这个是用于C/C++混合编程的，当引用C语言代码时在函数前面加上。

内联函数
========

编译器在编译内联函数时会对参数类型进行检查。

堆栈溢出的原因
==============

1.  分配了内存没有释放
2.  递归层次太深

唯一不能声明为虚函数的函数
==========================

构造函数！PS：析构函数可以声明为虚函数。

IP地址的两部分
==============

网络号：主机号

\#error
=======

当预处理执行到\#error时，会停止编译，并给出自定义的错误信息

指针&数组
=========

-   指向数组的指针：

``` {.example}
int (*a)[10];
```

-   指向函数（返回值为int,1个int参数）的指针：

``` {.example}
int (*a)(int);
```

volatile
========

修饰的变量，表明可能会被意想不到地改变，因此编译器不会从寄存器的备份中
读取（因为内存中的值可能已经被改变了），而要每次小心地从内存中读取。

事务处理的ACID
==============

-   actomic: 原子性，不能再细分
-   consistent: 事务处理前后，数据保持一致
-   isolated: 一个事务处理对另一个没有影响
-   durable: 永久保存

