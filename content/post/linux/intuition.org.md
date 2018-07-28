---
date: 2013-03-28
title: linux系统初识
---

常识
====

-   当命令太长时，可以使用反斜杠\来防止按RET执行命令，能在下一行继续输入
-   可以用分号;来分隔命令，因为可以在同一行输入多个命令
-   显示日历：cal

man
===

man中查询的项后面的数字的含义：

  数字   含义
  ------ -----------------------
  1      shell命令或可执行文件
  2      系统调用
  3      C函数库
  4      设备文件
  5      配置文件
  6      游戏
  7      协议
  8      管理员可用的命令
  9      和kernel有关的文件

如：man fstab, man NULL

其中1，5，8特别重要，但是怎么记住呢？man man就行了！

man -f参数能够显示要查找的东西的简短解释, whatis也行。

info这货好，man是用vim的快捷键的，而info则是用Emacs的快捷键的，正好适合我\^\_\^.

关机
====

ps -u 可以查看当前user的进程 ps -au 查看所有user的进程

sync 可以将内存中的数据同步进硬盘。

shutdown,
halt和poweroff的区别，引用自[serverfault](http://serverfault.com/questions/191537/shutdown-what-is-difference-between-power-off-and-halt)

-   shutdown -H now 相当于halt,会关机，最后显示器显示"System halted"
-   shutdown -P now 相当于poweroff，会关机会关闭电源
-   shutdown -h now 则由BIOS的设定而halt或者poweroff

run level

一共有7种执行等级，常见的4种为：

-   run level 0:关机
-   run level 3:纯文本模式
-   run level 5: 图形接口模式
-   run level 6: 重启

因此，可以使用命令

``` {.example}
init 0
```

来关机。

文件管理
========

用户的相关信息放在了/etc/passwd里面，密码放到了/etc/shadow。所有的组的信息都
放在了/etc/group里面。

ls -l出来的信息，例如：

``` {.example}
drwxr-xr-x  2 monkey sudo     4096 Feb 24 14:43 beijing
-rw-r--r--  1 monkey sudo    40914 Mar 21 20:39 caiji.jpg
```

第一部分是权限类型。以beijing那个目录为例，第一个是文件类型，有下面各种：

  符号   类型
  ------ ---------------------
  d      directory目录
  -      普通文件
  l      link链接文件
  b      block存储型设备文件
  c      char字符型设备文件
  p      pipe数据输送文件
  s      socket网络端口文件

接下来的字符中3个为一组，第一组为owner的权限，第二组为所有群组的用户权限，
第三组为其它人的权限。

第二部分是一个数字，表示连接到该inode中。

第三部分是owner,第四部分是group,第四部分是大小，单位为字节，第五部分为修改
日期，最后一部分是文件名。

chown可以同时改变用户和组，如：

``` {.example}
sudo chown monkey:sudo caiji.jpg
```

就OK了，再也不用打2个命令了。另外使用-R选项还能递归修改整个目录的用户了组。

chmode的另外一种非数字的改变权限方法：

  ------- --- --- --- ----------
  chmod   u   +   r   filename
          g   -   w   
          o   =   x   
          a           
  ------- --- --- --- ----------

如：

``` {.example}
[monkey@itlodge pictures]$ ls -l caiji.jpg 
-rw-r--r-- 1 monkey sudo 40914 Mar 21 20:39 caiji.jpg
[monkey@itlodge pictures]$ chmod u=rwx,g=rw,o=r caiji.jpg 
[monkey@itlodge pictures]$ ls -l caiji.jpg 
-rwxrw-r-- 1 monkey sudo 40914 Mar 21 20:39 caiji.jpg
```

文件的权限：

-   r，可读取内容
-   w，可编辑，增加，修改，但是不能删除
-   x，可执行

目录的权限：

-   r，可ls
-   w, 可新建文件，删除文件，重命名文件，移动文件
-   x，进入目录

除了文本文件和二进制文件，居然还有一种数据文件，由程序指定的特定格式。
last命令可读取整个系统登陆的情况的数据，其实读取的是/var/log/wtmp
这个文件，用cat是读取不了的。

还有2种奇葩文件：socket和FIFO/pipe文件。

根目录下的/srv是service的缩写，用于放服务文件的。/sys和/proc差不多，都是记录
与内核相关的信息。

/usr不是user的缩写，是unix software resource的缩写。

pwd,
如果当前目录是一个链接目录的话，则显示当前目录，加上-P则显示链接到的目录

``` {.example}
[monkey@itlodge mail]$ pwd
/var/mail
[monkey@itlodge mail]$ pwd -P
/var/spool/mail
```

mkdir, 加上-p可以建立新目录里面的新目录。

``` {.example}
[monkey@itlodge tmp]$ mkdir a/b/
mkdir: cannot create directory ‘a/b/’: No such file or directory
[monkey@itlodge tmp]$ mkdir -p a/b/
[monkey@itlodge tmp]$ ls
a
```

cp, 加上-a可以把文件的所有属性都一起复制过去，是真正的复制。

居然还有一个rename工具，可以用来指重命名。

od可以用来查看二进制文件。

每个文件都会记录三种时间：

1.  修改时间mtime
2.  状态时间ctime
3.  访问时间atime

ls默认显示的是mtime，cat等查看工具会更新atime，修改了权限等状态会更新ctime.

文件居然还有隐藏属性，使用chattr可以改变属性，比如+i会让文件不能被删除。lsattr
可以列出隐藏属性。

file命令可以查看文件的属性。

type能查询可执行指令的类型（alias还是builtin还是可执行文件）
