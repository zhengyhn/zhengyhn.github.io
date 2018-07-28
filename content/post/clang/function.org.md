---
date: 2013-03-10
title: tcpl读书笔记：函数与程序结构化
---

@&lt;font color="red"&gt; 本文为the c programming language一书的笔记
@&lt;/font&gt;

exercise 4-2
============

-   扩展atof函数，支持科学计数法。下面是我写的代码，未完全测试正确。

``` {.c}
#include <stdio.h>
#include <string.h>
#include <math.h>

int isBlank(char c)
{
    return c == ' ' ||
        c == '\n' ||
        c == '\t';
}

int isDigit(char c)
{
    return c >= '0' && c <= '9';
}

double atof(char *str)
{
    double intPart,fraction,power;
    int i,len,exp,sign,exp_sign;

    len = strlen(str);
    i = 0;
    if(str[i] == '-'){
        sign = -1;
        i++;
    }else{
        sign = 1;
    }
    while(isBlank(str[i])){
        i++;
    }
    for(intPart = 0.0; isDigit(str[i]); i++){
        intPart = intPart * 10 + str[i] - '0';
    }
    if(str[i] == '.'){
        i++;
    }else if(str[i] == '\0'){
        return sign * intPart;
    }else{
        printf("digit form incorrect!\n");
        return 0.0;
    }
    for(fraction = 0.0,power = 10.0; isDigit(str[i]); i++){
        fraction += (str[i] - '0') / power;
        power *= 10;
    }
    if(str[i] == 'e' || str[i] == 'E'){
        i++;
    }else if(str[i] == '\0'){
        return sign * (intPart + fraction);
    }else{
        printf("digit form incorrect!\n");
        return 0.0;
    }
    if(str[i] == '-'){
        exp_sign = -1;
        i++;
    }else{
        exp_sign = 1;
    }
    for(exp = 0; isDigit(str[i]); i++){
        exp = exp * 10 + str[i] - '0';
    }
    if(str[i] == '\0'){
        return sign * (intPart + fraction) * pow(10,exp_sign * exp);
    }else{
        printf("digit form incorrect!\n");
        return 0.0;
    }
}

int main()
{
    char *str[5] = {"123","-123.4","123.45e4","-123.123e-2","0.0"};
    int i;

    for(i = 0; i < 5; i++){
        printf("%lg\n",atof(str[i]));
    }

    return 0;
}
```

递归的quick sort例子。
======================

-   书上的算法，刚开始我有点看不懂，不太像我以前使用的qsort，倒更像选择排序。
    但是后来研究了一番之后，发现是我错了，人家写得真的很巧妙。下面的是书上的代码：

``` {.c}
void swap(int *arr, int i, int j)
{
    int temp;

    temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

void qsort(int *arr, int left, int right)
{
    int i, last;

    if (left >= right){
        return;
    }
    swap(arr, left, (left + right) / 2);
    last = left;
    for (i = left + 1; i <= right; i++) {
        if (arr[i] < arr[left]) {
            swap(arr, ++last, i);
        }
    }
    swap(arr, left, last);
    qsort(arr, left, last - 1);
    qsort(arr, last + 1, right);
}
```

刚开始，找一个元素作为参照元素，作者选中间的那个，并把它与第一个元素交换。
接着，遍历后面的每个元素，与参照元素进行比较，如果发现有元素比参照元素小，则
放到参照元素的后面，并把一个变量last来记录最后一个比参照元素小的元素的下标。
遍历一次后，再把last记录的那个元素与第一个元素（参照元素）交换。这样，在参照
元素左边的元素都比它小，在它右边的元素都比它大。二分两边再递归qsort，即实现了
整个算法。

-   因为听说面试要现场写快排，现在不赶紧练一下，到时候会OVER的，于是参照上面的

方法写了一个快排，尽量写得漂亮并且readable。

``` {.c}
#include <stdio.h>

void swap(int *arr, int i, int j)
{
    int temp;

    temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}

void qsort(int *arr, int first, int last)
{
    int i, last_small;

    if (first >= last){
        return;
    }
    last_small = first;
    for (i = first + 1; i <= last; i++) {
        if (arr[i] < arr[first]) {
            last_small++;
            swap(arr, last_small, i);
        }
    }
    swap(arr, first, last_small);
    qsort(arr, first, last_small - 1);
    qsort(arr, last_small + 1, last);
}

int main()
{
    int arr[7] = { 5, 2, 7, 1, 4, 3, 6 };
    int i;

    qsort(arr, 0, 6);
    for (i = 0; i < 7; i++) {
        printf("%d\n", arr[i]);
    }

    return 0;
}
```

-   上面的代码其实和K&R的差不多，只不过我使用的是第一个元素作为参照元素，这样就

减少了一次交换（其实选择哪个元素貌似和概率有关），我其实只是把变量改了一下，使
程序读起来更容易，第一次看last的时候，都不知道它想表达什么意思，而使用last~small~
就很容易看懂这个变量的作用。

预处理
======

-   在带参数的宏定义中，通过\#符号可引用该参数，并转化为字符串，看下面的例子：

``` {.c}
#include <stdio.h>

#define printd(digit) printf(#digit"=%d\n",digit)

int main()
{
    printd(10);

    return 0;
}
```

结果如下：

``` {.example}
10=10
```

这里，\#digit将会引用参数10,并替换成"10"，在C语言中，两个相连的字符串是可以连接的，
则最后会变成printf("10=%d\n",digit)

-   \#\#这个运算符用于连接宏参数，看下面的例子：

``` {.c}
#include <stdio.h>

#define concatenate(a,b) a##b

int main()
{
    int concatenate(hello,world) = 1;
    printf("%d\n", concatenate(hello,world));

    return 0;
}
```

结果如下：

``` {.example}
1
```

这东西比较神奇。

-   练习4-14,写一个宏swap(t, x, y)，交换2个t类型的元素。下面是我的代码：

``` {.c}
#include <stdio.h>

#define swap(t, x, y) \
    t temp = x;       \
    x = y;            \
    y = temp;         \

int main()
{
    double a = 1, b = 2;

    swap(double, a, b);

    printf("a = %lf\n"
           "b = %lf\n", a, b);

    return 0;
}
```

结果如下：

  --- --- -----
  a   =   2.0
  b   =   1.0
  --- --- -----


