---
date: 2022-08-17
title: gtk, widget
tags: ['gtk']
categories: ['GUI']
---

看下面一个装载widget的程序：

``` {.c}
#include <gtk/gtk.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//macros
#define BORDER_WIDTH 10

//global variables
GtkWidget *win;
GtkWidget *btn;
GtkWidget *top_box;
GtkWidget *separator;
GtkWidget *bottom_box;
GtkWidget *label;
GtkWidget *quitbox;

//functions
gint delete_event(GtkWidget *widget, GdkEvent *event, gpointer data);
void make_box(GtkWidget *parent, gboolean homogeneous, gint spacing,
              gboolean expand, gboolean fill, guint padding);
void make_btn(GtkWidget *box, GtkWidget *btn, gboolean expand,
              gboolean fill, guint padding, const char *label);

int main(int argc, char *argv[])
{
    gtk_init(&argc, &argv);

    //for win
    win = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    g_signal_connect(G_OBJECT(win), "delete_event",
                     G_CALLBACK(delete_event), NULL);
    gtk_container_set_border_width(GTK_CONTAINER(win), BORDER_WIDTH);

    //for top_box
    top_box = gtk_vbox_new(FALSE, 0);

    //add a label
    label = gtk_label_new("gtk_hbox_new(FALSE, 0);");
    gtk_misc_set_alignment(GTK_MISC(label), 0, 0);
    gtk_box_pack_start(GTK_BOX(top_box), label, FALSE, FALSE, 0);
    gtk_widget_show(label);

    //add three boxes
    //make_box(parent, homogeneous, spacing, expand, fill, padding);
    make_box(top_box, FALSE, 0, FALSE, FALSE, 0);
    make_box(top_box, FALSE, 0, TRUE, FALSE, 0);
    make_box(top_box, FALSE, 0, TRUE, TRUE, 0);

    //add a separator
    separator = gtk_hseparator_new();
    gtk_box_pack_start(GTK_BOX(top_box), separator, FALSE, TRUE, 5);
    gtk_widget_show(separator);

    //add a label again
    label = gtk_label_new("gtk_hbox_new(TRUE, 0);");
    gtk_misc_set_alignment(GTK_MISC(label), 0, 0);
    gtk_box_pack_start(GTK_BOX(top_box), label, FALSE, FALSE, 0);
    gtk_widget_show(label);

    //add two boxes
    make_box(top_box, TRUE, 0, TRUE, FALSE, 0);
    make_box(top_box, TRUE, 0, TRUE, TRUE, 0);

    //add a separator again
    separator = gtk_hseparator_new();
    gtk_box_pack_start(GTK_BOX(top_box), separator, FALSE, TRUE, 5);
    gtk_widget_show(separator);

    //add a box to quit
    quitbox = gtk_hbox_new(FALSE, 0);
    btn = gtk_button_new_with_label("quit");
    g_signal_connect_swapped(G_OBJECT(btn), "clicked",
                             G_CALLBACK(gtk_main_quit),G_OBJECT(win));
    gtk_box_pack_start(GTK_BOX(quitbox), btn, TRUE, FALSE, 0);
    gtk_widget_show(btn);

    gtk_box_pack_start(GTK_BOX(top_box), quitbox, FALSE, FALSE, 0);
    gtk_widget_show(quitbox);          //quitbox end

    gtk_container_add(GTK_CONTAINER(win), top_box);
    gtk_widget_show(top_box);         //top_box end

    gtk_widget_show(win);       //win end

    gtk_main();

    return 0;
}

gint delete_event(GtkWidget *widget, GdkEvent *event, gpointer data)
{
    gtk_main_quit();

    return FALSE;
}

void make_box(GtkWidget *parent, gboolean homogeneous, gint spacing,
              gboolean expand, gboolean fill, guint padding)
{
    GtkWidget *box;
    GtkWidget *btn;
    char padstr[80];

    box = gtk_hbox_new(homogeneous, spacing);

    make_btn(box, btn, expand, fill, padding, "(gtk_box_pack, ");
    make_btn(box, btn, expand, fill, padding, "box, ");
    make_btn(box, btn, expand, fill, padding, "button, ");
    if (expand == TRUE) {
        make_btn(box, btn, expand, fill, padding, "TRUE, ");
    } else {
        make_btn(box, btn, expand, fill, padding, "FALSE, ");
    }
    sprintf(padstr, "%d)", padding);
    make_btn(box, btn, expand, fill, padding, padstr);

    gtk_box_pack_start(GTK_BOX(parent), box, FALSE, FALSE, 0);
    gtk_widget_show(box);
}

void make_btn(GtkWidget *box, GtkWidget *btn, gboolean expand,
              gboolean fill, guint padding, const char *label)
{
    btn = gtk_button_new_with_label(label);
    gtk_box_pack_start(GTK_BOX(box), btn, expand, fill, padding);
    gtk_widget_show(btn);
}
```

这里用到一些陌生的函数：

``` {.c}
void gtk_box_pack_start(GtkBox *box, GtkWidget *child,
                        gboolean expand, gboolean fill, guint padding);
```

把child装进去box里面，expand表明是否展开，如果为FALSE则收缩。在expand为TRUE的情况
下，如果fill为TRUE，则会填充，容器之间没有间隔，如果为FALSE则有间隔。padding是
widget两边的的间隔。 还有一个函数：

``` {.c}
GtkWidget *gtk_hbox_new(gboolean homogeneous, gint spacing);
```

spacing为widget之间的间隔，homogeneous表明是否均匀。琢磨一下上面的例子，运行一下
就会明白了。
