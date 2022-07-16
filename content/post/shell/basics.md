---
date: 2022-07-16
title: shell基础知识
tags: ['Shell']
categories: ['DevOps']
---

本文整理自我多年前读Advanced Bash-Scripting Guide一书的笔记



# 特殊字符
## \#

-   在双引号和单引号里面的\#不会被认为是注释开始，经\转义的\#不会认为是注释开始。

``` {.bash}
echo "#这里不是注释"
echo '#这里不是注释'
echo \#这里不是注释
echo #这里不是注释
```
## ;

-   和C语言不同，这里分号用于在同一行中分隔两条命令，这一点和pascal很相似

``` {.bash}
if [ -z $1 ]; then
    echo "-->must with paramether 'start' or 'stop'"
elif [ $1 == "start" ]; then
    echo "starting..."
    sudo systemctl start smbd nmbd
    echo "done!"
elif [ $1 == "stop" ]; then
    echo "stoping..."
    sudo systemctl stop smbd nmbd
    echo "done!"
else
    echo "-->error parameter!"
fi
```

上面的分号就是用于分隔前面的判断和后面的then，分号后面最好加个空格，好区别一点。

## ;;

-   两个分号用于case语句中选项的结束符

``` {.bash}
case "$1" in
    "abc") echo "\$1 = abc" ;;
    "xyz") echo "\$1 = xyz" ;;
esac    
```

-   原来case的结构是这样的，和pascal的很相似，最后的结束esac，就是case倒过来，

我发现了规律，因为if的结束是fi，因此我猜其它的也是这个规律。

## .

-   当用于命令时，它和source是一样的功能。source是什么功能呢？以前看到修改了.bashrc

总是要source一下才会使配置生效，今天看了这本书，才发现，原来source就相当于C语言中
的\#include指令，用于给脚本包含一个文件。

-   在正则表达式中，dot匹配单个字符。

## " and '

-   "表示partial quoting,'表示full quoting

## ,

-   逗号可以用于分隔数学表达式，和C语言一样，只返回最后一个逗号后面的表达式的值。如：

let "t = ((a = 9, 15 / 3))"

## \`

-   这个符号，在我的键盘里，是和波浪号～在一个键的，它的作用是命令替换，即\`command\`

得到的是执行command这个命令之后的结果，它的英文名是backquote或者backtick

## :

-   冒号的英文名叫colon，在bash
    shell里面起空命令的作用，相当于汇编里面的NOP。
-   它还表示true。比如：

``` {.bash}
while :
do
    echo "wow"
done
```

相当于

``` {.bash}
while true
do
    echo "wow"
done
```

-   它的退出状态是0

``` {.bash}
:
echo $?
```

结果是：

``` {.example}
0
```

-   在if/then结构中，用于空操作，相当于C语言中的分号。

``` {.bash}
if condition
then 
    :
else
    ...
fi
```

-   它还有一个巧妙的用法，就是防止当作命令使用。比如：

``` {.bash}
: ${username=`whoami`}
echo $username
```

这里，如果不用 **:** ，则后面拿到的用户名会当作命令来用。

-   它还用作路径的分隔符，在\$PATH中就会看到。

## ! 

-   一般用于“非”，用法和C语言一样
-   两个!表示上一个命令，一般打某个命令忘了打sudo，则下一个命令可以使用sudo
    !!

## \*

-   通配符来的，通配符的英文是wild card，不查我还真不明白wild
    card到底是什么意思！
-   在正则表达式中，它匹配0个或多个字符。
-   在数学表达式中，一个\*表示乘号，两个\*表示幂运算。
-   两个\*还有一个特殊功能，它表示递归的文件匹配。看下面的例子：

``` {.bash}
#!/bin/bash

shopt -s globstar

for file in **
do
    echo $file
done    
```

但是，需要开启globstar，因为这个是bash 4里面的新加的功能。

## ?

-   像C语言中的三目运算符一样，可以和?:一起构成if/then/else结构，但是必须要用双括号。

``` {.bash}
a=1
b=2
((min = $a < $b ? $a : $b))
echo $min       
```

## \$

-   最普遍的应用是用于变量替换。
-   在正则表达式中，\$表示字符串的结束。
-   \$?取得结束状态(exit status)
-   \$\$取得当前脚本的进程ID

## ()

-   第一个作用是命令组，有点像C语言中的区块的概念。

``` {.bash}
a=123
(a=321;)
echo $a
```

结果为：

``` {.example}
123
```

-   第二个作用用于数组初始化。

## {}

-   这个才是真正的区块，但是奇怪的是，这里的区块里面的变量外面却是可见的。

``` {.bash}
a=123
{ a=321; }
echo $a
```

结果为：

``` {.example}
321
```

注意{}里面要加空格。

## &gt; and &&gt; &gt;& and &gt;&gt;
-   &gt;用于把stdout重定向到文件
-   &&gt;用于把stdout和stderr重定向到文件
-   &gt;&用于把stdout重定向到stderr
-   &gt;&gt;和&gt;的不同之处在于，&gt;&gt;是append到文件的结尾。

# 控制字符

## Ctrl-c

-   break用于结束前台作业

## Ctrl-d

-   退出
-   EOF

## Ctrl-h

-   相当于退格键，但是要比退格键快

## Ctrl-L

-   相当于clear命令。

## Ctrl-t

-   当前光标所在的字符与上一个字符交换，打错的时候特别有用。

## Ctrl-u

-   清除一行命令（再也不用按那么多退格键了）
