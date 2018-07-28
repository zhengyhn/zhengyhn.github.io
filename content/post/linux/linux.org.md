---
date: 2013-03-22
title: 鸟哥的linux私房菜笔记：linux是什么&主机规划
---

@&lt;font color="red"&gt;
本文为"鸟哥的linux私房菜基础学习篇(第三版)"一书的笔记 @&lt;/font&gt;

概念
====

原来BSD是指Berkeley Software Distribution.

System V是AT&T自家的Unix

LSB, Linux Standard Base

FHS, File system Hierarchy Standard

常识
====

常见设备的文件名：

  设备                文件名
  ------------------- --------------------------
  IDE硬盘             /dev/hd\[a-d\]
  SCSI/SATA/USB磁盘   /dev/sd\[a-p\]
  软盘                /dev/fd\[0-1\]
  25针打印机          /dev/lp\[0-2\]
  USB打印机           /dev/lp\[0-15\]
  USB鼠标             /dev/usb/mouse\[0-15\]
                      我的是：
                      /dev/input/mouse\[0-15\]
  PS2鼠标             /dev/psaux
  当前CDROM/DVDROM    /dev/cdrom

IDE
===

一般的主机会提供2个IDE接口，通常称为IDE1（primary）和IDE2（secondary），
每个IDE接口可以连接2个IDE装置，通常称为master和slave，于是最多可连接4个IDE
装置。如下表：

  IDE    master   slave
  ------ -------- -------
  IDE1   hda      hdb
  IDE2   hdc      hdd

磁盘
====

磁盘第一个扇区记录了两个重要信息：

-   Master Boot
    Recorder，即大名鼎鼎的MBR，用于安装系统引导程序，占446字节
-   分区表(Partition Table)，记录整个磁盘的分区情况，共64字节

分区表
------

磁柱(cylinder)是文件系统的最小单位，也是分区的最小单位。

第一个扇区中的64字节的分区表，只能记录4个分区记录，每个记录16字节，记录了起始
磁柱号和结束磁柱号。

是不是只有分4个区呢？不是的，有人想出了很好的方法，可以分很多区。使用的是
extended partition的方法。

``` {.example}
比如有400个磁柱，那么按照传统的方法，分成4个区为sda1:1-100, sda2:101-200,
sda3:201-300, sda4:301-400，4个都是primary partition.
如果使用extended partition的方法，则为sda1:1-100, sda2:101-400,sda2中
没有直接保存分区表信息，而是指向另外的扇区，比如一共要分6个区，sda1已经是一个
了，sda2中没有，sda3和sda4这2个名字是保留给主分区用的，这里已经不存在了，于是
下一个是sda5:101-160, sda6:161-220, sda7:221-280, sda8:281-340,
sda9:341-400。
```

晒一下我现在的分区情况：

``` {.example}
Disk /dev/sda: 160.0 GB, 160041885696 bytes, 312581808 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xc3ffc3ff

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *          63    61448624    30724281    7  HPFS/NTFS/exFAT
/dev/sda2        61448686   312578047   125564681    f  W95 Ext'd (LBA)
/dev/sda5        61448688    92903766    15727539+  83  Linux
/dev/sda6        92907520   310310911   108701696   83  Linux
/dev/sda7       310312960   312578047     1132544   82  Linux swap / Solaris

```

可以看到sda2记录的是61448686-312578047, sda5又从61448688开始，sda7结束
位置为312578047.

扩展分区只能有1个,由扩展分区分出来的区称为逻辑分区，扩展分区无法格式化。

MBR
---

CMOS是记录硬件各项参数且嵌入在主板上的存储器

BIOS是一个写入主板的固件(firmware)，即一个写入硬件的软件程序，它是开机时执行
的第一个软件程序。BIOS会根据设定从某个磁盘（硬盘，软盘，光盘，U盘）启动，然后
从磁盘中的第一个扇区读取MBR，MBR中包含了bootloader，bootloader可以读取
操作系统内核，于是就开始启动系统了。

每个分区都有自己的启动扇区，在安装引导（比如grub)时，可以选择安装到MBR还是某个
分区的启动扇区。
