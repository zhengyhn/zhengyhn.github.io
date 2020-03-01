---
date: 2013-02-21
title: gtk, introduction
---

简介
====

-   GTK(GIMP
    ToolKit)，是用于构建GUI的库来的。取这个名字，是因为GTK是用于开发GIMP

(GNU image manipulation program)的，现在也用来开发GNOME(GNU Network
Object Model Environment),它建立在GDK(GIMP Drawing Kit)的基础上。

弹一个窗口的程序
================

-   代码

``` {.c}
#include <gtk/gtk.h>

int main(int argc, char *argv[])
{
    GtkWidget *win;

    gtk_init(&argc, &argv);

    win = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    gtk_widget_show(win);

    gtk_main();

    return 0;
}
```

-   所有的程序都要使用&lt;gtk/gtk.h&gt;这个头文件。
-   gtk~init~(gint \*argc, gchar
    \*\*\*argv)，原来这个库把基本的类型都重新定义了

一下，前面都加了g

-   gtk~init这个函数~，设置了一些基本的东西（比如默认的可视化界面，颜色图等，初始化

了一些默认的设置）

-   gtk~windownew函数用于创建窗口~，GTK~WINDOWTOPLEVEL参数表示我们想让这个窗口~

经过窗口管理器的装饰。这里默认的大小是200×200

-   gtk~widgetshow用于显示窗口~
-   gtk~main用于进入gtk的主函数循环~，在这个循环里，程序会一直等待事件的产生。

Hello World
===========

-   代码

``` {.c}
#include <gtk/gtk.h>

void hello(GtkWidget *widget, gpointer data)
{
    g_print("my name is yuanhang zheng\n");
}

gint delete_event(GtkWidget *widget, GdkEvent *event, gpointer data)
{
    g_print("delete event\n");

    return TRUE;
}

void destroy(GtkWidget *widget, gpointer data)
{
    g_print("destroy\n");
    gtk_main_quit();
}

int main(int argc, char *argv[])
{
    GtkWidget *window;
    GtkWidget *button;

    gtk_init(&argc, &argv);

    window = gtk_window_new(GTK_WINDOW_TOPLEVEL);

    g_signal_connect(G_OBJECT(window), "delete_event", G_CALLBACK(
                         delete_event), NULL);
    g_signal_connect(G_OBJECT(window), "destroy", G_CALLBACK(destroy), NULL);

    gtk_container_set_border_width(GTK_CONTAINER(window), 10);
    button = gtk_button_new_with_label("yuanhang zheng");
    g_signal_connect(G_OBJECT(button), "clicked", G_CALLBACK(hello), NULL);
    g_signal_connect_swapped(G_OBJECT(button), "clicked",
                         G_CALLBACK(gtk_widget_destroy), G_OBJECT(window));
    gtk_container_add(GTK_CONTAINER(window), button);

    gtk_widget_show(button);
    gtk_widget_show(window);

    gtk_main();

    return 0;
}
```

-   编译的时候，我的是这样的：

``` {.example}
gcc -O2 hello.c -lm -o hello `pkg-config --cflags gtk+-2.0` 
`pkg-config --libs gtk+-2.0`
```

pkg-config是一个小工具，如果系统没有则要安装。 在终端执行下面的命令：

``` {.bash}
pkg-config --cflags gtk+-2.0
```

会输出gtk+-2.0程序需要使用到的头文件目录,比如我的是这样的：

``` {.example}
-pthread -I/usr/include/gtk-2.0 -I/usr/lib/gtk-2.0/include
 -I/usr/include/pango-1.0 -I/usr/include/atk-1.0 -I/usr/include/cairo
 -I/usr/include/pixman-1 -I/usr/include/libpng15
 -I/usr/include/gdk-pixbuf-2.0 -I/usr/include/libpng15
 -I/usr/include/pango-1.0 -I/usr/include/harfbuzz
 -I/usr/include/pango-1.0 -I/usr/include/glib-2.0
 -I/usr/lib/glib-2.0/include -I/usr/include/freetype2
```

在终端执行下面的命令：

``` {.bash}
pkg-config --libs gtk+-2.0
```

会输出gcc编译gtk所需要的库参数，比如我的是这样的：

``` {.example}
-lgtk-x11-2.0 -lgdk-x11-2.0 -lpangocairo-1.0 -latk-1.0 -lcairo
 -lgdk_pixbuf-2.0 -lgio-2.0 -lpangoft2-1.0 -lpango-1.0 -lgobject-2.0
 -lglib-2.0 -lfreetype -lfontconfig
```

-   有关这些库的说明：
    -   -lgtk,是GTK的库，主要是widget的库
    -   -lgdk,是GDK的库，主要是Xlib的wrapper
    -   -lgdk~pixbuf~,是gdk~pixbuf的库~，主要包含了图像处理的操作
    -   -lpango,是Pango库，用于国际化，即对各国语言的支持
    -   -lgobject,是gobject库，包含了gtk里面的类型系统
    -   -lgmodule，是gmodule库，用于加载运行时的扩展
    -   -lglib, 是GLib库，包含了一些乱七八糟的函数，比如g~print~
    -   -lX11，是Xlib库，GDK需要用到
    -   -lXext，是Xext库，包含了共享内存的pixmap的代码和其它的X扩展
    -   -lm，这个不用说，是数学函数库
-   有关信号signal的概念

调用了gtk~main后~，程序会睡眠，等待事件的发生，当有事件（如按下键盘一个键）
发生时，发送者会发送信号，然后执行某一个操作。g~signalconnect的原型为~：

``` {.c}
gulong g_signal_connect(gpointer *object, const gchar *name,
                        GCallback func, gpointer func_data);
```

其中object为信号的发送者，name为信号的名字，func是回调函数，即要执行的
操作，func~data是给函数传递的数据~。 回调函数的一般原型为：

``` {.c}
void callback(GtkWidget *widget, gpointer data);
```

第一个参数widget为发送信号的widget，第二个参数data为传递过来的数据。
我刚开始对g~signalconnectswapped函数有很多疑问~，看了书上的说法，终于
明白了，它和g~signalconnect的作用几乎一样~，只是它的回调函数只接收一个
参数，如：

``` {.c}
void callback(GtkObject *object);
```

因为gtk里面的很多库函数没有传送数据，所以才出现了这个东西专门用来回调那些
不传递数据的函数。

-   有关事件event的概念

``` {.c}
g_signal_connect(G_OBJECT(button), "button_press_event",
                 G_CALLBACK(button_press_callback), NULL);
static gint button_press_callback(GtkWidget *widget,
                                  GdkEventButton *event,
                                  gpointer data);
```

可以通过上面这种方式使用回调函数。
GdkEvent是一个联合体，是很多事件的名字的集合。

深入探索Hello World
===================

-   delete~event函数~，返回TRUE表示，不产生destroy信号，窗口不会关闭，如果返回

FALSE，则会关闭窗口。

-   G~OBJECT和GCALLBACK是用来强制转换类型并检查类型的宏~。
-   一个widget可以绑定多个回调函数，按照绑定的顺序来触发，这里，先绑定hello函数，

再绑定gtk~widgetdestroy~，因为没有数据传递，所以使用了g~signalconnectswapped~
函数，最后一个参数表明这个按钮button是放在window上面的。

-   gtk~widgetshow的顺序要注意~，先把button显示，再显示window，这样就不会出现

先弹出窗口，再显示按钮这种奇怪的事情了。

-   gtk的设计是面向对象的。
-   修改过的hello world

``` {.c}
#include <gtk/gtk.h>

void hello(GtkWidget *widget, gpointer data)
{
    g_print("%s\n", (gchar *)data);
}

gint delete_event(GtkWidget *widget, GdkEvent *event, gpointer data)
{
    gtk_main_quit();

    return FALSE;
}

void destroy(GtkWidget *widget, gpointer data)
{
    gtk_main_quit();
}

int main(int argc, char *argv[])
{
    GtkWidget *window;
    GtkWidget *button;
    GtkWidget *box;

    gtk_init(&argc, &argv);

    window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    gtk_window_set_title(GTK_WINDOW(window), "yuanhang");
    gtk_container_set_border_width(GTK_CONTAINER(window), 20);
    g_signal_connect(G_OBJECT(window), "delete_event", G_CALLBACK(
                         delete_event), NULL);
    g_signal_connect(G_OBJECT(window), "destroy", G_CALLBACK(destroy), NULL);

    box = gtk_hbox_new(FALSE, 0);
    gtk_container_add(GTK_CONTAINER(window), box);

    button = gtk_button_new_with_label("one");
    g_signal_connect(G_OBJECT(button), "clicked", G_CALLBACK(hello),
                     (gpointer)"one");
    gtk_box_pack_start(GTK_BOX(box), button, TRUE, TRUE, 0);
    gtk_widget_show(button);

    button = gtk_button_new_with_label("two");
    g_signal_connect(G_OBJECT(button), "clicked", G_CALLBACK(hello),
                     (gpointer)"two");
    gtk_box_pack_start(GTK_BOX(box), button, TRUE, TRUE, 0);
    gtk_widget_show(button);

    gtk_widget_show(box);
    gtk_widget_show(window);

    gtk_main();

    return 0;
}
```

这里加了一个box,把2个button加到box里面，传递回调函数的数据时，传了一个字符串，
在回调函数里面把这个字符串打印出来。
