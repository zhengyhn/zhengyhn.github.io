---
date: 2013-02-16
title: Linux c编程一站式学习笔记：C语言本质
---

@&lt;font color="red"&gt; 本文为"linux c编程一站式学习"一书的笔记
@&lt;/font&gt;

计算机中数的表示
================

二进制数表示法
--------------

-   LSB称为第0位而不是第1位，所以如果一个数是32位，则MSB称为第31位。
-   sign and magnitude表示法：

第一位为符号位,在做加法运算时，需要这样：

1.  如果符号位相同，则符号位不变，其他位相加，在不溢出的情况下得到结果。
2.  如果符号位不同，则先比较非符号位谁大，然后用大数减小数，最后符号位

和大数的相同，这样就保证了结果的正确性。
!!这种方法，效率低，而且0的表示不唯一，既可以表示为1000 0000也可以表示
为0000 0000

-   1's complement表示法

一句话概括：负数用1的补码(1's
complement，也叫反码)表示，减法转换成加法，计算
结果的最高位如果有进位则要加回到最低位上去。

0000 1000 - 0000 0100 = 0000 1000 + (-0000 0100) =

0000 1000 + 1111 1011 = 0000 0100

!!这种方法，效率高了，正数和负数一样算，但是还是有2个0

-   2's complement表示法

为什么叫2的补码呢？因为对一个只有一位的二进制数b取补码就是1 - b + 1 = 1
+ 1 - b = 10 - b，即2减去b。 如果要计算-1的补码，则为

1111 1111 - 1000 0001 + 0000 0001 = 0111 1110 + 0000 0001 = 0111 1111

!!这种方法，减法转换成加法，计算结果最高位的进位直接忽略，不需要加到低位

-   溢出标志：相加过程中最高位产生的进位和次高位产生的进位如果相同则没有溢出，

不同则溢出。

浮点数
------

-   科学计数法：scientific notation
-   尾数：mantissa或significand
-   科学计数法基数(radix)是10,而浮点数的是2
-   大致的模型如下：

  -------- ---------- ---------- --
  符号位   指数部分   尾数部分   
  -------- ---------- ---------- --

-   但是不能表示负数，于是有人想出来了一种偏移指数(biased
    exponent)的方法，

规定一个偏移值，比如127,实际的指数要加上这个偏移值再填到指数部分，这样比
127大的就表示正指数，比127小的就表示负指数。

-   还有一个问题，浮点数的表示不唯一，9 = (0.1001) \* 2 \^
    4，还可以表示为：

9 = (0.01001) \* 2 \^
5，为了解决这个问题，我们需要制定一个标准，规定最高位
必须为1,也就是尾数必须是以0.1开头，这叫正规化(normalize)。由于尾数部分第一
位必须是1,所以这个1就不用保存了，可以节省一位来提高精度。

数据类型详解
============

-   ILP32和LP64规范

  type        ILP32   LP64
  ----------- ------- ------
  char        8       8
  short       16      16
  int         32      32
  long        32      64
  long long   64      64
  pointer     32      64

其中ILP32表示int,long,pointer是32位的。

-   有的处理器有浮点运算单元（FPU, floating point
    unit），称为硬浮点(hard-float)，

有的没有，用整数来模拟浮点运算，称为软浮点(soft-float)。

-   float一般是32位，double一般是64位。
-   而long
    double,在x86上，大部分编译器实现的为80位，因为x86的FPU是80位精度的。而

gcc实现的是96位，还有的编译器实现的和double的一样，也有更高精度的，比如IBM的powerpc
上的为128位。

-   如果一个函数的形参类型未知，或者使用了不定参数...，那么调用的时候要对实参做integer

promotion,另外，float要提升为double,这叫default argument
promotion，例如：

``` {.c}
char c = 'a';
printf("%c", c);
```

c要被提升为int型。

-   signed或unsigned的char,
    short和bit-field在做算术运算之前要先提升为int，因为

怕溢出。

-   usual arithmetic conversion：

1.  if 有long double，则要转化成long double
2.  else if 有double，则要转化成double
3.  else if 有float，则要转化成float
4.  else if 有int,则要转化成int
5.  对于有符号数和无符号数，则要转化成存储位数大的那个操作数的类型。

运算符详解
==========

-   当操作数是有符号负数时，执行右移运算，高位补1还是0是不确定的，这是

implementation-defined的，不同编译器有不同的实现。对于x86的gcc， 补1。

-   最好只对无符号数做移位运算，防止出错。
-   序列点(sequence point)的概念。

C标准中规定某些点是序列点，当执行到这个点时，在此之前的副作用必须全部
作用完毕，在此之后的副作用必须都没发生。

1.  调用一个函数时，在所有准备工作做完之后，函数调用开始之前是序列点。

比如调用一个函数

``` {.c}
sort(arr, get_len(arr), is_sorted(arr));
```

get\_len和is\_sorted哪个先执行是不确定的，但是必须它们都完成之后才调用sort函数。

1.  ? :和,和&&和||的第一个操作数求值之后是序列点
2.  一个完整的定义末尾是序列点。如：

``` {.example}
int name,age;
```

name后面是序列点，age后面也是。

计算机体系结构基础
==================

内存
----

-   一个内存地址对应的内存单元只能存一个字节。
-   如果一个整数占用了4个字节，则这个数据是占用了连续的多个地址，这种情况下数据

的地址为它所占内存单元的起始地址。

-   指令译码器(instruction
    decoder)的作用在于解释CPU获取的指令的含义，执行

相应操作。因为CPU获取到的指令由很多个字节组成，有的位表示内存地址，有的位表示
寄存器编号，有的表示做什么操作。

设备
----

-   有的设备像内存一样接到CPU的地址总线和数据总线上。设备中可供读写访问的单元通常

称为设备寄存器，操作设备的过程就是读写这些设备寄存器的过程，比如向串口发送寄存器
里写数据，串口设备就会把数据发送出去，读串口接收寄存器的值，就可以读取串口设备
接收到的数据。

-   有的设备集成在CPU里面，一些体系结构（如ARM），访问设备像访问内存一样，这种称为

内存映射I/O。但是x86比较特殊，它对于设备有独立的端口地址空间，需要引出额外的地址
线来连接CPU里面的设备，访问设备寄存器的时候使用特殊的in/out指令，这种称为端口I/O

-   在x86上，硬盘是挂载在IDE，SATA或SCSI总线上的设备。
-   每个设备都有一条中断线，通过中断控制器连接到CPU。
-   事实上linux内核的绝大部分源码是设备驱动程序。设备驱动程序通常是内核里的一组

函数，通过读写设备寄存器实现对设备的初始化，读和写等操作，有的设备还要提供一个
中断处理函数。

MMU
---

-   如果CPU里面没有MMU，则不能使用虚拟内存机制
-   异常的处理过程和中断类似，但是，中断是由外部设备产生的，而异常是由CPU内部

产生的，中断产生的原因和CPU当前执行的指令无关，而异常的产生就是因为CPU当前
执行的指令出了问题，例如除0操作。

存储设备
--------

-   cache是位于CPU里面的，cache一般使用的是SRAM，而内存一般使用的是DRAM，

可以推测，DRAM电器比SRAM简单，容量大，但是速度慢。

-   一级cache是用虚拟内存地址来寻址的，而二级cache则是用物理内存来寻址的。
-   硬盘位于设备总线上，并不直接和CPU相连，CPU通过设备总线的控制器访问硬盘。

汇编基础
========

最简单的汇编程序
----------------

-   代码simple.s

``` {.example}
.section .data
.section .text
.globl _start
_start:
        movl $1, %eax
        movl $4, %ebx

        int $0x80
```

-   先要使用下面的命令用汇编器as把汇编指令翻译成机器指令，生成simple.o

``` {.bash}
as simple.s -o simple.o
```

再用链接器ld把simple.o链接成可执行文件simple

``` {.bash}
ld simple.o -o simple
```

-   执行这个程序，它的作用就是退出，退出状态是4,相当于C语言中的return 4;

可以看到执行结果：

``` {.example}
[monkey@itlodge asm]$ ./simple 
[monkey@itlodge asm]$ echo $?
4
```

-   .section用于分段（数据段，代码段什么的），这里没有用到全局变量，所以数据段

是空的。.text是代码段，后面的代码都是这个段里面的。

-   .globl告诉汇编器，\_start这个符号要被链接器用到，所以要标记它是一个全局符号，

就相当于main函数一样，是一个程序的入口，

-   movl是一条数据传送指令，mov的后缀l表示long，说明是32位的传送指令。
-   \$1代表立即数，立即数前面要加\$，寄存器名前面要加%
-   为什么要赋一个1而不是其它立即数呢，因为\_exit()系统调用的系统调用号为1
-   mov指令第一个操作数是源操作数，第二个是目标操作数，和windows上的汇编不一样。
-   eax寄存器保存的是系统调用号，ebx保存的是返回状态
-   int指令是软中断指令，用来故意产生一个异常，立即数0x80是一个参数，在linux中，

这个参数表示系统调用。

-   x86汇编有两种语法:intel语法和AT&T语法，windows使用intel语法，而类Unix

系统都使用AT&T语法。

寄存器
------

-   x86的通用寄存器有eax, ebx, ecx, edx, edi,
    esi，大部分是可以任意选用的，

但是有的指令规定某个寄存器只能用于某种用途，比如除法指令idivl要求被除数必须在
eax中，edx必须为0,而除法可以任意选用，商必须保存在eax中，余数保存在edx中。

-   x86的特殊寄存器有ebp, esp, eip, eflags.eip是程序计数器，eflags保存着

计算过程中产生的标志位，ebp和esp用于维护函数调用的栈帧。

-   求最大值的汇编程序

代码biggest.s

``` {.example}
# find the maximum number

.section .data
data_items:
        .long 2, 7, 1, -5, 9, 0

.section .text
.globl _start
_start:
        movl $0, %edi
        movl data_items(, %edi, 4), %eax
        movl %eax, %ebx

loop:
        cmpl $0, %eax
        je exit
        incl %edi
        movl data_items(, %edi, 4), %eax
        cmpl %ebx, %eax
        jle loop

        movl %eax, %ebx
        jmp loop

exit:
        movl $1, %eax
        int $0x80
```

编译运行结果：

``` {.bash}
[monkey@itlodge asm]$ as biggest.s -o biggest.o
[monkey@itlodge asm]$ ld biggest.o -o biggest
[monkey@itlodge asm]$ ./biggest 
[monkey@itlodge asm]$ echo $?
9
```

-   data\_items相当于数组
-   .long表示定义的数占32位，没有用.globl，说明这个数组只在本文件内使用。
-   .byte占8位，.ascii取值为相应字符的ASCII码
-   edi保存的是数组当前的下标，eax是当前的值，ebx是最大值，所以最后返回

状态的是最大值。

-   data\_items(, %edi, 4)是一种地址表示形式。

寻址方式
--------

-   通用格式为：

``` {.example}
address_or_offset(%base_or_offset, %index, multiplier)
```

这种格式表示的地址是这样计算的：

``` {.example}
address = address_or_offset + base_or_offset + multiplier * index
```

其中address\_or\_offset,
multiplier必须是常数，base\_or\_offset,index必须是寄存器。
省略这些项，表示它们是0

-   直接寻址(direct addressing mode),只使用address\_or\_offset
-   变址寻址(indexed addressing mode)，前面的data\_items就是这种
-   间接寻址(indirect addressing mode)，只使用base\_or\_offset，如movl
    (%ebx), %eax
-   基址寻址(base pointer addressing
    mode)，使用address\_or\_offset和base\_or\_offset

寻址，如movl 4(%eax), %ebx

-   立即数寻址(immediate addressing mode)
-   寄存器寻址(register addressing mode)

ELF文件
-------

-   ELF格式是很多类UNIX系统都采用的文件格式，它有三种类型
    -   可重定位的目标文件(relocatable or object file)
    -   可执行文件(executable file)
    -   共享库(shared object or shared library)，libc.so之类的
-   readelf工具可以查看.o文件和可执行文件
-   hexdump工具可以打印出来ELF文件的十六进制表示和ASCII码表示
-   objdump工具可以把机器指令反汇编

汇编与C的关系
=============

常识
----

-   要查看编译后的汇编代码，给gcc加上-S参数，这样只生成汇编文件而不产生可执行文件

比如下面的一个C程序：

``` {.c}
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("yuanhang\n");

    return 0;
}
```

通过编译之后，产生的汇编文件如下：

``` {.example}
    .file   "t.c"
    .section    .rodata
.LC0:
    .string "yuanhang"
    .text
    .globl  main
    .type   main, @function
main:
.LFB0:
    .cfi_startproc
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    subq    $16, %rsp
    movl    %edi, -4(%rbp)
    movq    %rsi, -16(%rbp)
    movl    $.LC0, %edi
    call    puts
    movl    $0, %eax
    leave
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc
.LFE0:
    .size   main, .-main
    .ident  "GCC: (GNU) 4.7.2"
    .section    .note.GNU-stack,"",@progbits
```

-   如果在编译的时候加上-g参数，则在使用objdump工具反汇编时，可以看到C代码，还是

刚刚那个程序：

``` {.example}
[monkey@itlodge test]$ gcc -g t.c
[monkey@itlodge test]$ objdump -dS a.out > t.txt
...
00000000004004fc <main>:
#include <stdio.h>

int main(int argc, char *argv[])
{
  4004fc:   55                      push   %rbp
  4004fd:   48 89 e5                mov    %rsp,%rbp
  400500:   48 83 ec 10             sub    $0x10,%rsp
  400504:   89 7d fc                mov    %edi,-0x4(%rbp)
  400507:   48 89 75 f0             mov    %rsi,-0x10(%rbp)
    printf("yuanhang\n");
  40050b:   bf c4 05 40 00          mov    $0x4005c4,%edi
  400510:   e8 cb fe ff ff          callq  4003e0 <puts@plt>

    return 0;
  400515:   b8 00 00 00 00          mov    $0x0,%eax
}
  40051a:   c9                      leaveq 
  40051b:   c3                      retq   
  40051c:   0f 1f 40 00             nopl   0x0(%rax)
...
```

只复制了部分结果

-   gdb命令diassemble可以反汇编当前函数或指定函数
-   gdb中表示寄存器要在前面加\$
-   gcc的一条命令：

``` {.bash}
gcc t.c -o t
```

可以分解为下面3条：

``` {.bash}
gcc -S t.c
gcc -c t.s
gcc t.o -o t
```

-   可以用nm工具查看ELF文件的符号表
-   返回状态被shell解释为无符号数，如果return -1，则会变成255
-   看一个有关位域的例子：

``` {.c}
#include <stdio.h>

typedef unsigned int uint;

union demo {
    struct {
        uint one:1;
        uint two:3;
        uint three:10;
        uint four:5;
        uint :2;
        uint five:8;
        uint six:8;
    } bitfield;
    char byte[8];
};

int main(int argc, char *argv[])
{
    union demo d = { { 1, 5, 513, 17, 129, 0x18 } };
    int i;

    printf("size of demo:%u\n", sizeof(union demo));

    printf("values:%u %u %u %u %u %u\n",
           d.bitfield.one, d.bitfield.two, d.bitfield.three,
           d.bitfield.four, d.bitfield.five, d.bitfield.six);
    for (i = 0; i < 8; i++) {
        printf("%x ", d.byte[i]);
    }
    printf("\n");

    return 0;
}
```

运行结果为：

``` {.example}
[monkey@itlodge test]$ ./t
size of demo:8
values:1 5 513 17 129 24
1b 60 24 10 18 0 0 0 
```

我们知道，byte数组和bitfield是共用存储空间的，所以可见bit-field的内存布局为：
0000 0001 1000 0001 0000 0010 0100 0110 0000 0001 1011 高地址 6组全0
低地址

-   习题：写一个程序，判断平台是小端规则还是大端规则。

有了前面的例子的启发，我写了下面程序来验证：

``` {.c}
#include <stdio.h>

union test {
    int num;
    char byte[2];
};

int main(int argc, char *argv[])
{
    union test t = { 0x1001 };

    if (t.byte[0] == 0x01 && t.byte[1] == 0x10) {
        printf("little endian\n");
    } else if (t.byte[0] == 0x10 && t.byte[1] == 0x01) {
        printf("big endian\n");
    } else {
        printf("error!\n");
    }

    return 0;
}
```

C内联汇编
---------

有的相关平台的指令必须要手写汇编，因为C语言中没有等价的概念，比如x86的端口I/O
使用\_\_asm\_\_("assembly code");来插入汇编语句，多条语句用\n\t分开
完整的内联汇编格式为：

``` {.example}
__asm__(assembly template
        : output operands
        : input operands
        : list of clobbered registers
);
```

除了assembly template，其它的都是可选的 看下面的例子：

``` {.c}
#include <stdio.h>

int main(int argc, char *argv[])
{
    int a, b = 10;

    __asm__("movl %1, %%eax\n\t"
            "movl %%eax, %0\n\t"
            :"=r"(a)
            :"r"(b)
            :"%eax"
        );
    printf("a:%d\n", a);

    return 0;
}
```

其中，为了区分%1这种占位符，eax前面要加两个%号。
这里%0代表的是变量a,%1代表的是变量b，这两条movl指令其实就是把变量b的值
先放到一个寄存器eax里面，然后再把eax里面的值放到变量a里面。
"r"(b)表示分配一个寄存器保存变量b的值，而"=r"(a)表示把寄存器的值输出到
变量a中，最后一个"%eax"告诉编译器在这期间eax要被改写，不要让eax寄存器
保存其它值

volatile限定符
--------------

-   gcc的编译优化选项有-00, -0, -01, -02, -03, -0s

其中-00表示不优化，这是默认的选项，后面的1-3一个比一个优化得多，编译
时间更长，-0s是为了缩小目标文件的尺寸而优化的。

链接详解
========

链接库
------

-   gcc使用下面的方式链接多个文件：

``` {.example}
gcc file1.c file2.c -o file
```

也可以这样：

``` {.example}
gcc -c file1.c
gcc -c file2.c
gcc file1.o file2.o -o file
```

gcc链接的作用其实是把一些段合并到同一个segment中，还要插入一些符号到生成的
脚本中，如果不使用-T选项指定脚本，默认使用ld作为链接脚本。

-   使用ar工具打包静态库，静态库都是以.a作为后缀，表示archive，如：

我现在有3个文件， t.h

``` {.c}
#ifndef _T_H_
#define _T_H_

void hello(void);

#endif /* _T_H_ */
```

t.c

``` {.c}
#include <stdio.h>
#include "t.h"

void hello(void)
{
    printf("hello\n");
}
```

main.c

``` {.c}
#include "t.h"

int main(int argc, char *argv[])
{
    hello();

    return 0;
}
```

先编译成目标文件：

``` {.bash}
gcc -c t.c
```

这时会生成和t.o 现在打包成一个静态库libt.a

``` {.bash}
ar rs libt.a t.o
```

这时会生成libt.a
Note:r表示将后面的文件列表添加到文件包，s用于为静态库建立索引(ranlib有同样功能)
现在把libt.a和main.c编译

``` {.bash}
gcc main.c -L. -lt -I. -o main
```

这里，-L.表示在当前目录(.)找库文件，-lt表示使用libt文件，-I.表示在当前目录找头
文件。

共享库
------

-   共享库只是指定了动态链接器和所需要的库文件，并没有真正做链接，只在运行时做动态

链接，而静态库是真正链接起来的。

-   组成共享库的目标文件要在编译的时候加上-fPIC选项，-f表示后面要加选项，FIC是其中

一种，表示生成位置无关代码(position independent
code)，生成的共享库，各段的 加载地址并不是绝对地址，可以加载到任意位置。

-   使用-shared参数生成共享库，一般以.so结尾。
-   共享库路径的搜索顺序：

1.  首先在环境变量LD\_LIBRARY\_PATH中查找
2.  从缓存文件/etc/ld.so.cache中查找，这个文件是由ldconfig工具读取

/etc/ld.so.conf后生成的。

1.  从默认的系统路径中找(/usr/lib,/lib)

-   当需要加上某个共享库文件时，先要把这个文件的绝对路径加到/etc/ld.so.conf文件中，

再使用ldconfig工具更新缓存文件，这时就能使用了。

-   还有一种方法，把这个文件拷贝到/usr/lib或/lib中，让它能找到并使用
-   共享库的命名惯例

每个共享库有3个文件名，real name, soname, linker name real
name包含完整的版本号，如libzip.so.2.1.0
soname是一个符号链接，包含了主版本号，如libzip.so.2。主版本号一致即可保证库函数
的接口一致。 linker name只在编译链接的时候使用，有的linker
name是库文件的符号链接，有的是 一个链接脚本。

预处理
======

-   转义字符的替换是在预处理阶段完成的
-   \#用于给两边加上双引号，\#\#用于连接参数或字符串
-   书上举了下面这个例子：

``` {.example}
#define VERSION 2
#if defined x || y || VERSION < 3
```

先把VERSION定义为2,再看defined
x，x没有定义，所以替换为0（如果有定义则为1）， 变成了： \#if 0 || y ||
VERSION &lt; 3 然后把VERSION展开，变成了： \#if 0 || y || 2 &lt; 3
再把没有定义的变成0,于是： \#if 0 || 0 || 2 &lt; 3 最后变成了： \#if 1

-   几个特殊的宏

1.  \_~FILE~\_~展开为当前文件的名字~
2.  \_~LINE~\_~展开为当前代码行的行号~。
3.  \_~func~\_~展开为当前函数名~

Makefile基础
============

基本规则
--------

-   Makefile由一组rule组成，每条规则的格式为：

target ... : prerequisites ... command1 command2 ...

注意，每条命令必须以一个tab开头,不能是空格
每条命令，都会创建一个shell进程来执行 看下面的例子：

``` {.example}
main : main.o stack.o
        gcc main.o stack.o -o main
main.o : main.c main.h stack.h
        gcc -c main.c
stack.o : stack.c stack.h
        gcc -c stack.c
```

Makefile先要执行目标main，发现main.o stack.o需要更新，于是去执行main.o，
于是执行目录main.o，再执行目标stack.o，最后执行main

-   在命令前加@，不显示命令本身，只显示它的结果，如：

``` {.example}
@echo "cleaning project"
```

-   在命令前面加-号，即使命令出错，也会继续执行后续命令，通常rm和mkdir命令前面

要加-号，因为rm删除的文件或目录可能不存在，mkdir要创建的目录可能已经存在。

-   如果有一个目标clean，而恰好又有clean这个文件，make就认为它不需要更新，这个

时候就要添加一条特殊规则，把clean声明为一个伪目录：

``` {.example}
clean:
        @echo "clean"
        -rm *.o
.PHONY: clean
```

-   一些约定的目标名字
    -   all，执行主要的编译工作，通常为缺省目标
    -   install，安装，把可执行文件，配置文件，文档等拷到不同的目录下
    -   clean，清理二进制文件
    -   distclean，不仅删除二进制文件，也删除其它生成的文件（如文档）

隐含规则和模式规则
------------------

-   一个目标依赖的条件不一定写在一条规则上，可以分开写：

``` {.example}
main.o main.h stack.h
main.o main.c
        gcc -c main.c
```

注意，其中只有一条规则允许有命令列表，其它规则不能有命令列表，否则会报警并采用
最后一条规则的命令列表

-   隐含规则数据库可以用make -p打印出来，比如可以看到：

``` {.example}
CC = cc
```

cc在/usr/bin中，可以看到它是符号链接，指向gcc

-   \$@取值为规则中的目标，\$&lt;取值为规则中的第一个条件。
-   %.o: %.c是一种模式规则，看下面：

``` {.example}
main: main.o stack.o
        gcc main.o stack.o -o main
main.o: main.h stack.h
stack.o: stack.h
```

上面的例子没有给出main.o的命令列表，这时它会查找隐含规则，发现隐含规则中有
%.o: %.c这样一条模式规则可用，于是就加上了：

``` {.example}
main.o: main.c
        cc -c -o main.o main.c
```

同理，stack.o也是一样。

变量
----

-   变量的值可以推迟到后面定义，如：

``` {.example}
a = $(b)
b = ccc

all:
        @echo $(a)
```

-   通常将CFLAGS定义成一些编译选项，如-g
    -lm等，把CPPFLAGS定义成预处理选项。
-   如果希望在遇到变量定义的时候立即展开，则用:=号，如：

``` {.example}
b := ccc
a := $(b)
all:
        @echo $(a)
```

-   对于?=号，如：

``` {.example}
a ?= $(b)
```

如果a没有定义过，那么?=相当于=，如果定义过，则什么也不做

-   对于+=号，是用于给变量追加值，如：

``` {.example}
objs = main.o
objs += stack.o
```

则objs最后变成了main.o stack.o，它会自动在前面加一个空格
如果变量还没定义过就用+=，则它相当于=

-   常用的特殊变量：
    -   \$@，目标
    -   \$&lt;，第一个条件
    -   \$?，所有比目标新的条件，组成一个列表，以空格分开
    -   \$\^，所有条件

如：

``` {.example}
main: main.o stack.o maze.o
      gcc $^ -o $@
```

这样即使修改了目标的名字或者添加了条件，下面的命令也不需要修改
对于\$?变量，看下面的例子：

``` {.example}
libstack.a: main.o stack.o
        ar rs libstack.a $?
```

这样，只对更新过的条件进行操作，没更新过的目标文件已经打包好了

-   一些常用的隐含变量：

  name             description                 default value
  ---------------- --------------------------- ----------------------------------------------------------------
  AR               static library archives     ar
  ARFLAGS          static library options      rv
  AS               assembly compiler           as
  ASFLAGS          assembly compiler options   null
  CC               c compiler                  cc
  CFLAGS           c compiler options          null
  CXX              c++ compiler                g++
  CXXFLAGS         c++ compiler options        null
  CPP              c preprocessor              \$(CC) -E
  CPPFLAGS         c preprocessor optios       null
  LD               linker                      ld
  LDFLAGS          linker options              null
  TARGET\_ARCH     target dependent options    null
  OUTPUT\_OPTION   output                      -o \$@
  LINK.o           link all .o files           \$(CC) \$(LDFLAGS) \$(TARGET\_ARCH)
  LINK.c           link all .c files           \$(CC) \$(CFLAGS) \$(CPPFLAGS) \$(LDFLAGS) \$(TARGET\_ARCH)
  LINK.cc          link all c++ files          \$(CXX) \$(CXXFLAGS) \$(CPPFLAGS) \$(LDFLAGS) \$(TARGET\_ARCH)
  COMPILER.c       compile .c files            \$(CC) \$(CFLAGS) \$(CPPFLAGS) \$(TARGET\_ARCH) -c
  COMPILER.cc      compile cpp files           \$(CXX) \$(CXXFLAGS) \$(CPPFLAGS) \$(TARGET\_ARCH)
  RM               remove                      rm -f

自动处理头文件的依赖关系
------------------------

-   使用gcc的-M参数可以自动生成目标文件和源文件的依赖关系
-   如果不需要系统的头文件关系，则使用-MM参数
-   make的官方手册建议的写法是这样的：

``` {.example}
all: main
main: main.o stack.o
        gcc $^ -o $@
clean:
        -rm main *.o
.PHONY: clean

sources = main.c stack.c
include $(sources:.c=.d)
%.d %.c
        set -e; rm -f $@; \
        $(CC) -MM $(CPPFLAGS) $< > $@.$$$$; \
        sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
        rm -f $@.$$$$
```

部分解释一下，有的东西太复杂，没搞懂
\$(sources:.c=.d)是一种变量替换，把sources里面所有的.c文件替换成.d文件，
include类型C语言中的\#include，用于包含文件
4个\$读进来变成了2个\$，2个\$\$表示进程的ID

-   常用的命令行选项
    -   -n,只打印要执行的命令，不会真的执行命令，用于检查makefile的执行顺序，

它不是顺序执行的

-   -C，后面接目录，可以用于执行那个目录下的Makefile
-   在make的时候也可以加选项，如：

``` {.example}
make CFLAGS=-g
```

如果Makefile中也定义了CFLAGS，则命令行的值会覆盖掉Makefile中的值

指针
====

-   最初C语言没有通用的指针，就用char
    \*当作通用的指针，需要转换的时候就用

强制转换()，后来引进了void \*，这时候可以隐式转换，不用强制转换了。

-   规定指针不能相加

指针与const
-----------

-   const int \*a; a指向的内存单元不能更改，但是a能更改
-   int const \*a; a指向的内存单元能更改，但是a不能更改
-   const int const \*a; a指向的内存单元不能更改，a也不能更改
-   指向非const变量的指针或者非const变量的地址可以传给指向const变量的指针
-   但是，指向const变量的指针或地址就不能传给指向const变量的指针
-   良好的编程习惯要尽可能多地使用const，因为

1.  读代码的人可以放心地使用，不用但是内存单元会修改
2.  可以依靠编译器来检查出来bug
3.  可以让编译器优化成常量

指针与数组与函数
----------------

-   char \*argv\[\]中的\[\]表示指针而不是数组，等价于char
    \*\*argv,写成\[\]这种形式是

为了告诉读代码的人，它不是指向指针而是指向一个指针数组的首元素

-   指向数组的指针：

``` {.example}
int (*arr)[10];
```

指针arr指向数组arr,这个数组有10个int类型的元素。

-   函数也是一种类型，函数指针，看下例：

``` {.c}
void hello(const char *str)
{
    printf("hello %s\n", str);
}

int main(int argc, char *argv[])
{
    void (*f)(const char *str) = hello;
    f("me");

    return 0;
}
```

定义了一个指向像hello这种函数类型(返回值是void，1个参数，而且第一个参数类型是
const char \*)的指针,然后就可以用f来操作函数了。

类型分类
--------

-   函数类型
-   不完全类型
    -   void
    -   不完全结构体，联合体，数组类型
-   对象类型
    -   非标量类型(struct, union, array)
    -   标量类型
        -   指针类型
        -   算术类型
            -   浮点型(float, double, long double)
            -   整型
                -   (unsigned)char, (unsigned)short, (unsigned)int,
                    (unsigned)long
                -   位域
                -   枚举

C标准库
=======

字符串
------

-   void \*memset(void \*s, int c, size\_t
    n);初始化字符串，把从s开始的n个字节

都填充为c的值。

-   void \*memcpy(void \*dest, const void \*src, size\_t n);

从src拷贝n个字节到dest，如果内存重叠，不能正确拷贝

-   void \*memmove(void \*dest, const void \*src, size\_t n);

其实也是从src拷贝n个字节到dest，但是如果内存重叠，也可以正确拷贝

-   restrict关键字告诉编译器，可以放心优化，传进来的指针不会指向重叠的内存空间。

-   int memcmp(const void \*s1, const void \*s2, size\_t n);

比较内存中的字节，如果前n个字节一样则返回0,如果遇到不一样的字节，返回s1
- s2的值

-   int strcasecmp(const char \*s1, const char \*s2);和

-   int strncasecmp(const char \*s1, const char \*s2, size\_t n);

忽略大小写，但是不是标准C函数，而是POSIX中定义的。

-   char \*strchr(const char \*s, int c);

从左往右查找字符c，找到第一次出现的位置，返回这个位置

-   char \*strrchar(const char \*s, int c);

从右往左查找，r可以理解为reverse

-   char \*strstr(const char \*haystack, const char \*needle);

haystack意为干草堆，needle是针，即大海捞针，表明是从haystack指向的字符串中查找
needle指向的字符串，返回第一次出现的位置。

-   char \*strtok(char \*str, const char \*delim);

这是一个奇怪的函数，根据界定符delim把字符串str分隔开来，每次分隔一个token，返回
这个token的首地址。第一次调用传入str字符串，以后调用都传入NULL，函数内部实现中
使用了静态指针变量，它会记住上次处理到了哪里，但是这样是不好的做法，这叫不可重入的。

-   char \*strtok\_r(char \*str, const char \*delim, char \*\*saveptr);

这个函数，调用者自己维护一个saveptr来记住当前字符串处理到哪里，这种是可重入的，r
代表reentrant，这个函数不属于C标准库，而是POSIX标准库中的。

-   上面这2个函数要改写字符串，分隔符被替换为'\0'，不能用于常量字符串。strtok不是

线程安全的。

I/O库函数
---------

### 文件基本概念

-   文件可分为文本文件（源文件）和二进制文件（目标文件，可执行文件，库文件）。
-   文本文件中的字节都是字符的某种编码（ASCII或UTF-8），二进制文件中的字节表示

其它含义，有的表示指令，有的表示地址。

-   od命令可以dump file in octal or hex format

### 打开关闭文件

-   FILE是结构体类型，它包含了文件描述符，I/O缓冲区和当前读写位置等信息，但是

调用者不必知道FILE里面有哪些成员，调用者只需要把这个文件指针在库函数接口中传
来传去就行了，操作由里面的函数来维护，这种思想叫封装！这种指针叫不透明指针，也
叫句柄(handle)，它就像一个把手，抓住它就可以打开门或抽屉。

-   FILE \*fopen(const char \*path, const char \*mode);

mode是由rwatb+六个字符组合而成的，r是read，w是write，a是append，t是text，
b是binary，对于+号，有下面的约定： r+ 必须先允许读，再允许写 w+
必须先允许写（不存在则创建，已存在则覆盖），再允许读 a+
必须允许追加（不存在则创建），再允许读

### errno和ferror

-   errno是一个整数，当有错误产生时，它会赋值为错误码的number，直接打印只看到

数字，如果使用perror函数，就可以打印出来字符串（错误的原因）了。

-   char \*strerror(int errnum);

传进去错误码，返回错误原因，有的错误码并不存在errno中，这个时候这个函数就非常
有用了，而不是只依赖于perror函数。

### 以字节为单位的I/O函数

-   int fgetc(FILE \*stream);

如果stream是stdin，则和int getchar(void);是等价的，出错或者读到文件结尾
返回EOF

-   为什么要返回int？因为需要返回EOF，为-1，如果读到字节0xff，返回char的话无法

分清是EOF还是0xff，如果为int，则EOF是0xffffffff，而0xff为0x000000ff

### 操作位置的函数

-   int fseek(FILE \*stream, long offset, int whence);

成功返回0,出错返回-1并设置errno whence可以选择下面几个：

1.  SEEK\_SET，从文件开头
2.  SEEK\_CUR，从当前位置
3.  SEEK\_END，从文件结尾

-   long ftell(FILE \*stream);

返回当前读写的位置

-   void rewind(FILE \*stream);

把读写位置移到到文件开头

### 以字符串为单位的函数

-   char \*fgets(char \*s, int size, FILE \*stream);

指定长度，如果不指定长度，stream又是stdin，则和gets功能一样，gets是不推荐使用
的函数，因为用户提供一个缓冲区，却不能指定缓冲区的大小。

-   int fputs(const char \*s, FILE \*stream);

成功返回一个非负整数，出错返回EOF

### 以记录为单位的函数

-   这里的记录指一串固定长度的字节，如一个整数，一个结构体和一个数组。
-   size\_t fread(void \*ptr, size\_t size, size\_t nmemb, FILE
    \*stream);

size是记录的长度，nmemb是要读多少条记录。成功返回记录数nmemb，失败返回0
size\_t fwrite(const void \*pstr, size\_t size, size\_t nmemb, FILE
\*stream);

### 格式化I/O函数

-   int fprintf(FILE \*stream, const char \*format, ...);

打印到指定的流中。

-   int sprintf(char \*str, const char \*format, ...);

打印到指定的缓冲区中，没有指定长度，很容易产生缓冲区溢出错误。

-   int snprintf(char \*str, size\_t size, const char \*format, ...);
-   对于vprintf, vfprintf, vsprintf,
    vsnprintf，和上面的函数的唯一区别就是最后

一个参数不是通过可变参数传进来的，而是通过va\_list传进来的。

-   同样有，scanf, fscanf, sscanf, vscanf, vsscanf, vfscanf

数值字符串转换函数
------------------

-   int atoi(const char \*nptr);

字符串转10进制整数

-   double atof(const char \*nptr);

字符串转浮点数

-   long int strtol(const char \*nptr, char \*\*endptr, int base);

根据base，返回不同进制的整数 endptr返回后面未被识别的第一个字符
如果超过范围，则返回它能表示的最大（最小）数，并设置errno

-   double strtod(const char \*nptr, char \*\*endptr);

是atof的增强版。

分配内存的函数
--------------

-   void \*calloc(size\_t nmemb, size\_t size);

分配nmemb个元素的内存空间，每个元素大小是size，并且负责清0,而malloc是不清零的。

-   void \*realloc(void \*ptr, size\_t size);

内存空间使用一段时间之后需要改变大小，就可以使用这个函数，它会重新找一块足够的内存
空间，把内容复制过来，并释放原来的内存空间。

-   void \*alloca(size\_t size);

这个是在栈上分配空间，不需要free，相当于变长数组，这不是C标准，而是POSIX标准。
