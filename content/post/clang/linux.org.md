---
date: 2013-02-27
title: linux系统编程
---

@&lt;font color="red"&gt; 本文为"linux c编程一站式学习"一书的笔记
@&lt;/font&gt;

文件与I/O
=========

再来一个hello world
-------------------

-   我也来写一段汇编的hello world

``` {.example}
.data
msg:
        .ascii "yuanhang zheng\n"
        len = . - msg

.text
.global _start

_start:
        movl $len, %edx
        movl $msg, %ecx
        movl $1, %ebx     #1 is stdout
        movl $4, %eax     #4 is sys_write
        int $0x80

        movl $0, %ebx    #0 is exit code
        movl $1, %eax    #4 is sys_exit
        int $0x80

```

运行结果为：

``` {.example}
[monkey@itlodge asm]$ as -o hello.o hello.s
[monkey@itlodge asm]$ ld -o hello hello.o
[monkey@itlodge asm]$ ./hello
yuanhang zheng
```

对上面程序的几点说明：

1.  汇编中.ascii定义的字符串末尾没有隐含'\0'
2.  len的值是. - msg，即当前地址用.号表示，这是一种神奇的求长度方法！

-   用C语言实现刚刚的程序

``` {.c}
#include <unistd.h>

#define LEN 15

char msg[LEN] = "yuanhang zheng\n";

int main(int argc, char *argv[])
{
    write(1, msg, LEN);
    _exit(0); 

    return 0;
}
```

事实上write和\s~exit就包装了上面两段代码~。其中头文件unistdh意思是unix
standard。

-   POSIX,portable operationg system
    interface,用于统一各种UNIX的函数接口。

open/close
----------

-   在linux系统编程中，我们可以使用标准C函数库里面的fopen和fclose，也可以使用系统的

接口open/close

-   open函数如下：

``` {.c}
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```

事实上是这样声明的：

``` {.c}
int open(const char *pathname, int flags, ...);
```

flags有很多常数值选择，多个常数用按位或连接起来，这些宏定义都以O\_开头（大写o），
表示or。

1.  必选项（必选一个且只能选一个）

  O\_RDONLY   read only
  ----------- ----------------
  O\_WRONLY   write only
  O\_RDWR     read and write

1.  可选项（部分）

  O\_APPEND   append
  ----------- ---------------------------------------------------
  O\_CREAT    create if not exist, should give parameter "mode"
  O\_TRUNC    truncate as zero byte if exist, read and write

open与fopen的一些区别：

-   以可写的方式fopen时，如果文件不存在会自动创建，而用open必须要使用O\_CREAT，否则

出错。

-   以可写的方式fopen时，如果文件已经存在则截断为0字节，而用open必须要使用O\_TRUNC,

否则出错。
mode指定文件权限，这里学习到了权限字符与数字表示之间怎么计算，比如：

``` {.example}
-rwxr-xr-x
```

这是一个很常见的权限，我们要把它分为3部分：

``` {.example}
u/g/o rwx rwx rwx
```

记住下面的字符数字对应关系：

  u   g   o   r   w   x
  --- --- --- --- --- ---
  4   2   1   4   2   1

好，开始计算，第一个为-，表示留空，即为0。第二段为rwx，即4 + 2 + 1 = 7。
第三段和第四段为rw-，即4 + 1 = 5，最后整个权限为0755。
作者在这里提到文件权限由open的mode参数和当前进程的umask掩码共同决定。
通过下面的命令查看shell进程的umask掩码：

``` {.bash}
[monkey@itlodge ~]$ umask
0022
```

我的是0022。根据作者的推理，

-   当用touch创建一个文件时，创建权限是0666,

而touch继承了shell进程的umask掩码，变成了0666 & \~0022 = 0644。

-   当用gcc编译生成一个可执行文件时，创建权限是0777,而最后是

0777 & \~0022 = 0755 下面来做个实验，写个程序：

``` {.c}
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char *argv[])
{
    int fd = open("abc.c", O_WRONLY | O_CREAT, 0666);
    close(fd);

    return 0;
}
```

运行结果为：

``` {.bash}
[monkey@itlodge c]$ ls -l abc.c
-rw-r--r-- 1 monkey sudo 0 Feb 24 20:15 abc.c
```

可以看到，权限最终为0644。

-   习题

1.  查找flags和mode参数用到的宏定义，我在fcntl-linux.h中找到如下：

``` {.c}
#define O_RDONLY         00
#define O_WRONLY         01
#define O_RDWR           02
#ifndef O_CREAT
# define O_CREAT       0100 /* Not fcntl.  */
#endif
```

以0开头，都是8进制数，而且按位或之后各自占的位不影响。
有关mode的宏则没找到。

read/write
----------

``` {.example}
ssize_t read(int fd, void *buf, size_t count);
```

成功返回读取的字节数，出错返回-1并设置errno。这里ssize\_t表明signed，
文件的当前位置是记录在内核里面的而不是用户空间的I/O缓冲区。

``` {.example}
ssize_t write(int fd, const void *buf, size_t count);
```

读常规文件是不会阻塞的，而从终端设备和网络中读就可能会阻塞。
来看一下阻塞读终端的例子：

``` {.c}
#include <unistd.h>
#include <stdlib.h>

#define BUFFER_SIZE 10

int main(int argc, char *argv[])
{
    char buf[BUFFER_SIZE];
    int bytes_read;

    bytes_read = read(STDIN_FILENO, buf, BUFFER_SIZE);
    if (bytes_read < 0) {
        perror("read STDIN_FILENO");
        exit(-1);
    }
    write(STDOUT_FILENO, buf, bytes_read);

    return 0;
}
```

输出结果如下：

``` {.bash}
[monkey@itlodge c]$ ./main 
hello
hello
[monkey@itlodge c]$ ./main 
hello world
hello worl[monkey@itlodge c]$ bash: d: command not found
```

在第二次，因为缓冲区只有10个字节，而hello
world一共11个字符，所以还剩下最后一个
d没有读，留在了内核的终端设备缓冲区里面，当main进程退出时，轮到shell进程执行，
继续读取用户输入，读取到了d和换行符，把它当成一条命令。 @&lt;font
color="red"&gt; 至于书上说的非阻塞I/O，我不太明白，不敢下笔。
@&lt;/font&gt;

lseek
-----

off\_t lseek(int fd, off\_t offset, int whence);
可以想像，它和fseek类似。偏移量允许超过文件末尾，这种情况，下一次写操作
将延长文件，中间空洞部分读出来的都是0.
和fseek的一点区别是，fseek成功时返回0,失败时返回-1,而lseek成功时返回
当前偏移量失败时返回-1。

fcntl
-----

``` {.c}
int fcntl(int fd, int cmd);
int fcntl(int fd, int cmd, long arg);
int fcntl(int fd, int cmd, struct flock *lock);
```

-   可以想像，它是用可变参数实现的，可变参数的类型和个数取决于cmd，常用的cmd

有F\_GETFL(拿到flags),F\_SETFL(设置flag)。

-   将一个标志flag与O\_ACCMODE做与运算，可以拿到它的读写位，与O\_APPEND做与

运算，可以判断是否是append，与O\_NONBLOCK做与运算，可以判断是否阻塞。

-   学习到部分重定向语法：

1.  &gt;&gt; 是追加
2.  在重定向符号左边加数字，表示打开文件描述符为这个数字的文件，比如：

``` {.example}
2>> temp.txt
```

表示将标准错误输出以追加的方式重定向到temp.txt中

1.  如果要重定向到标准输入/输出/错误呢？用符号&，比如：

``` {.example}
2>&1
```

将标准错误输出重定向到标准输出，注意&gt;后面不能有空格。

-   /dev/null设备文件只有一个作用，往它里面写任何数据都被直接丢弃。这种用法

可以让命令安静地执行而不打印任何信息。

ioctl
-----

-   向设备发送的命令，有些命令也需要读写数据，但是这些数据有的不能用read/write

来操作，而需要用ioctl，这些数据称为out-of-band数据，而用read/write来操作
的数据称为in-band数据。

``` {.c}
int ioctl(intd, int request, ...);
```

下面的例子用于获取终端窗口大小：

``` {.c}
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/ioctl.h>

int main(int argc, char *argv[])
{
    struct winsize size;

    if (isatty(STDOUT_FILENO) == 0) {
        exit(1);
    }
    if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &size) < 0) {
        perror("ioctl TIOCGWINSZ error");
        exit(1);
    }
    printf("%d * %d\n", size.ws_row, size.ws_col);

    return 0;
}
```

其中isatty用于判断是不是在tty中执行这个程序，如果是说明没有终端图形窗口。

mmap
----

``` {.c}
void *mmap(void *addr, size_t len, int prot, 
           int flag, int filedes, off_t off);
int munmap(void *addr, size_t len);
```

-   如果addr为NULL，则内核会自己在进程地址空间中选择合适的地址映射。
-   strace可以用于跟踪程序执行过程中用到的所有系统调用。我的系统原来没有

这个工具，后来自己装的。

文件系统
========

-   硬链接的建立，直接用ln
-   符号链接文件里面保存的其实就是它指向的文件的路径
-   ln只能创建目录的符号链接，但是不能创建目录的硬链接
-   /dev/zero是一个特殊的设备文件，它没有磁盘数据块，对它进行读操作传给设备

号为1,5的驱动程序,它可以看作是无穷大的。

进程
====

-   环境变量

打印环境变量：

``` {.c}
#include <stdio.h>

int main(int argc, char *argv[])
{
    extern char **environ;
    int i = 0;

    while (environ[i]) {
        printf("%s\n", environ[i]);
        i++;
    }

    return 0;
}
```

可以使用getenv取得环境变量，用setenv修改环境变量，unsetenv删除某个环境变量。

``` {.c}
char *getenv(const char *name);
int setenv(const char *name, const char *value, int rewrite);
void unsetenv(const char *name);
```

-   fork函数

``` {.c}
pid_t fork(void);
```

用于创建一个新进程，

-   exec函数

用于进程运行时调用另外一个程序。

-   wait和waitpid

用于得到某进程的退出状态并彻底清除这个进程。

``` {.c}
pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
```

-   pipe

``` {.c}
int pipe(int filedes[2]);
```

调用它时会在内核中开辟一块缓冲区（管道）用于通信，传进去的参数需要满足，

``` {.example}
filedes[0]是读端，filedes[1]是写端
```

管道是用环形队列实现的，数据从写端流入，从读端流出。

shell脚本
=========

-   内建命令builtins，反是用which查不到的程序都是内建命令，如cd命令。
-   source命令也是内建命令，不会创建子shell，而是直接执行脚本中的命令。
-   export命令可以把本地变量导出为环境变量。
-   命令代换也可以使用\$(命令)，这样免得打\`号（不好打）
-   \$(())用于算术代换，如：

``` {.bash}
a=5
echo $((a + 3))
```

-   交互shell指在提示符下输命令的shell，即用户登录后得到的shell，注意不是

进入桌面后打开终端。这时，会这样：

1.  首先执行/etc/profile
2.  然后依次查找当前用户的.bash~profile~ .bash~login和~.profile，sh规定

了.profile，而bash规定的则是以.bash~开头的~，如果没有则执行.profile。

1.  退出登录时会执行.bash~logout~（如果有）

-   交互非登录shell启动，如进入桌面后打开终端（我经常这样干）,或者在shell提示

符下再输入bash。这种情况下启动会执行\~/.bashrc。如果你打开.bash\_profile，
你会发现在里面通常会调用.bashrc。

-   命令test或\[可以测试一个条件是否成立。其中\[是一个命令，它需要\]作为参数，各

参数之间必须用空格分开。

-   常见测试命令：

  \[ -d DIR \]         if DIR exist and is a directory
  -------------------- ---------------------------------
  \[ -f FILE \]        if FILE exist and is a file
  \[ -z STRING \]      if length of STRING is zero
  \[ -n STRING \]      if length of STRING is not zero
  \[ STR1 = STR2 \]    if STR1 is equals to STR2
  \[ STR1 != STR2 \]   if STR1 is not equals to STR2
  \[ ARG1 OP ARG2      ARG1,ARG2 are integer

上面的op可以为-eq, -ne, -lt, -gt, -le, -ge。
！是非，-a是与(and)，-o是或(or)。

-   下面的代码：

``` {.bash}
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi
```

这里其实是3条命令：

1.  if \[ -f \~/.bashrc \]
2.  then source \~/.bashrc
3.  fi

-   调试方法

``` {.example}
-n 读一遍脚本但不执行
-v 一边执行，一边打印到标准错误输出
-x 提供跟踪执行信息，打印第一条命令的结果
```

正则表达式
==========

-   sed是stream editor
-   awk不仅可以以行为单位处理文件，还能以列为单位处理。

信号
====

-   启动一个前台进程，按下ctrl-c，会产生一个硬件中断，终端驱动程序会将ctrl-c

解释成一个SIGINT信号。

-   用kill -l可以查看系统定义的信号列表。
-   ctrl-\产生SIGQUIT，ctrl-z产生SIGSTP
-   SIGINT默认是终止进程，SIGQUIT默认是终止进程并core dump
-   当一个进程异常终止时，可以选择把进程的用户空间内存数据全部保存在磁盘上，文件名

通常为core，这个动作称为core
dump，以便调试。一个进程允许产生多大的core文件取 决于resource
limit，可以用ulimit命令来改变resource limit，默认是不产生core 文件的。

终端、作业控制与守护进程
========================

-   在/dev中，ptyXX是主设备，ttyXX是从设备，伪终端数目取决于内核配置
-   网络登录通过伪终端进行
-   守护进程不受用户登录注销的影响
-   ps
    axj命令列出来的进程中，用\[\]括起来的是内核线程，udevd负责/dev里面的设备，

acpid用于电源管理，syslogd用于日志管理，守护进程通常以d结尾，表示deamon

-   必须要给ps加上x才能看到守护进程

线程
====

创建线程
--------

``` {.c}
int pthread_create(pthread_t *restrict thread,
        const pthread_attr_t *restrict attr,
        void *(*start_routine)(void *), 
        void *restrict arg);
```

thread是线程号的指针，attr是线程的属性，详细不清楚，传入NULL表示取默认属性。
start\_routine是一个传入参数为void \*返回值为void
\*的函数的指针。新线程执行
的代码就是从这个函数中开始的，这个函数退出时，新线程也就退出了。后面的arg参数
是传进回调函数start\_routine的参数。 成功返回0,错误返回相应的错误号。
看下面的例子：

``` {.c}
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

pthread_t ntid;

void * routine(void *name);
void print_ids(const char *name);

int main(int argc, char *argv[])
{
    int errno;

    errno = pthread_create(&ntid, NULL, routine, "new:");
    if (errno) {
        fprintf(stderr, "can't create thread: %s\n", strerror(errno));
        exit(1);
    }
    print_ids("main:");
    sleep(1);

    return 0;
}

void * routine(void *name)
{
    print_ids(name);

    return NULL;
}

void print_ids(const char *name)
{
    pid_t pid;
    pthread_t tid;

    pid = getpid();
    tid = pthread_self();
    printf("%s pid: %u tid: %u\n", name, pid, tid);
}
```

getpid()用于获取当前进程的id，pthread\_self()用于获取当前线程的id。
pthread\_create的错误码不保存在errno中，不能用perror直接打印，需要用strerror
将错误码转换成错误信息之后再打印。

终止线程
--------

有3种方法：

1.  从线程函数中return，但是这样对主线程不适用。
2.  一个线程可以调用pthread\_cancel终止同一进程中的另一个线程。
3.  pthread~exit可以终止自己~。

网络socket编程
==============

-   下面的函数可以用于网络字节序和主机字节序之间的转换。

``` {.c}
#include <arpa/inet.h>
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

-   有关socket的基本数据类型和函数

ipv4和ipv6的地址格式定义在netinet/in.h中，ipv4地址用sockaddr\_in结构体表示，
ipv6则用sockaddr\_in6结构体，ipv4的地址类型定义为常数AF\_INET，ipv6的为AF\_INET6。

-   最简单的TCP网络程序

server.c

``` {.c}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define MAXLINE 80
#define SERVER_PORT 8080
#define MAX_CONN_CNT 20

int main(int argc, char *argv[])
{
    struct sockaddr_in server_addr, client_addr;
    int listenfd, connfd;
    char buf[MAXLINE];
    char ipaddr[INET_ADDRSTRLEN];
    int i, bytes_read, client_addr_len;

    listenfd = socket(AF_INET, SOCK_STREAM, 0);

    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    server_addr.sin_port = htons(SERVER_PORT);

    bind(listenfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    listen(listenfd, MAX_CONN_CNT);

    printf("accepting\n");
    while (1) {
        client_addr_len = sizeof(client_addr);
        connfd = accept(listenfd, (struct sockaddr *)&client_addr,
                        &client_addr_len);
        bytes_read = read(connfd, buf, MAXLINE);

        inet_ntop(AF_INET, &(client_addr.sin_addr),ipaddr, sizeof(ipaddr));
        printf("from %s:%d\n", ipaddr, ntohs(client_addr.sin_port));

        for (i = 0; i < bytes_read; i++) {
            buf[i] = toupper(buf[i]);
        }
        write(connfd, buf, bytes_read);
        close(connfd);
    }

    return 0;
}
```

socket函数，ipv4需使用AF\_INET，TCP使用SOCK\_STREAM，表示面向流的协议。
listen函数，第二个参数为最大连接数。
inet\_ntop函数，传入地址，返回点分式的IP，放到ipaddr中。

client.c

``` {.c}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define MAXLINE 80
#define SERVER_PORT 8080
#define SERVER_IP "127.0.0.1"

int main(int argc, char *argv[])
{
    struct sockaddr_in server_addr;
    char buf[MAXLINE];
    int sockfd, bytes_read;
    char *msg;

    if (argc != 2) {
        fprintf(stderr, "usage: ./client message\n");
        exit(1);
    }
    msg = argv[1];
    sockfd = socket(AF_INET, SOCK_STREAM, 0);

    bzero(&server_addr, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    inet_pton(AF_INET, SERVER_IP, &server_addr.sin_addr);
    server_addr.sin_port = htons(SERVER_PORT);

    connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    write(sockfd, msg, strlen(msg));

    bytes_read = read(sockfd, buf, MAXLINE);
    printf("from server:\n");
    write(STDOUT_FILENO, buf, bytes_read);
    printf("\n");
    close(sockfd);

    return 0;
}
```

客户端的端口号由内核自动分配。
