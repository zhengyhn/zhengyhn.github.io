---
date: 2013-03-21
title: 鸟哥的linux私房菜笔记：计算机概论
---

@&lt;font color="red"&gt;
本文为"鸟哥的linux私房菜基础学习篇(第三版)"一书的笔记 @&lt;/font&gt;

计算机常识
==========

原来Eeepc就是现在所说的上网本netbook.

原来作者所说的卡片阅读器就是读卡器(card reader)

读卡器要读SD卡，SD卡是什么呢？原来是secure digital memory card,
安全数码卡。

计算机的五大组成单元
====================

1.  输入单元
2.  算术逻辑单元
3.  控制单元
4.  存储单元
5.  输出单元

当然，这是基于冯诺依曼体系结构的。

CPU的种类
=========

-   RISC，reduced instruction set computing,精简指令集

这种CPU主要代表是Sun的SPARC,
IBM的PowerPC与大名鼎鼎的ARM系列。SPARC用于大型
工作站和一些服务器，PowerPC用于sony著名的PS3上，而ARM则不用说了，用在各种嵌入式
设备上。

-   CISC, complex instruction set computing,复杂指令集

主要代表是Intel, AMD, VIA的芯片。原来VIA是威盛来的，他们也是生产芯片的。

度量
====

这是最基础的，不仅要会写，而且要会读。下面的内容引自[百度百科](http://baike.baidu.com/view/60408.htm)

  KB   Kilobyte
  ---- ------------
  MB   Megabyte
  GB   Gigabyte
  TB   Terabyte
  PB   Petabyte
  EB   Exabyte
  ZB   Zettabyte
  YB   Yottabyte
  BB   Brontobyte

上面的是中国人的理解。而事实上，是这样的：

1 KiB(kibibyte) = 1024 byte

1 KB(kilobyte) = 1000 byte

1 MiB(mebibyte) = 1024 KiB

1 MB(megabyte) = 1000 KB

1 GiB(gibibyte) = 1024 MiB

1 GB(gigabyte) = 1000 MB

硬盘厂商是以1000算的，而我们的操作系统是用1024算的，所以如果你买了8G的U盘，实际
系统检测出来是小于8G的。

外围接口
========

PCI,原来是personal computer interface, 也有称为peripheral component
interconnect.它是由Intel推出的局部总线标准。

主板是由很多芯片组组成的，芯片组通常由2个网桥来控制各个组件的沟通。

-   北桥：负责连接速度较快的CPU，内存，显卡等
-   南桥：负责连接速度较慢的外围设备，包括硬盘，键盘，鼠标，USB，网卡等

当然，上面的是Intel的架构。而于是AMD的来说，主要的区别是，CPU与内存的沟通
不通过北桥，而是直接相连。

北桥的总线称为 **系统总线** ，南桥就是所谓的 **I/O总线**
.北桥支持的频率称为前端 总线速度(front side bus, FSB),
每次传送的位数称为总线宽度，而 **总线带宽**
就是频率乘以总线宽度，即每秒能传送的最大数据量。

CPU的知识
=========

外频：CPU与外部组件进行数据传输的速度

倍频：CPU内部用于加速工作效能的一个倍数

主频：外频 \* 倍频

显示
====

VGA, video graphics array, 显示适配器。

以前的VGA的运算交给CPU来运行，后来显卡厂商直接在显卡里面加了一个3D加速的
芯片，称为GPU(graphics process unit)。

磁盘
====

当是复习数据库。将磁盘的一个平面看成很多个同心圆，再由圆心发射很多射线分割，
得到的小扇形状的就叫扇区(sector)，每个扇区是512bytes（一定要记住），这
一圈的扇区组成的一个环，就叫磁道(track)，所有平面上同一个磁道就组成了一个
磁柱(cylinder)，磁柱是分割硬盘的最小单位。每个平面称为磁面，每个盘片(platter)
都有2个磁面，每个磁面有1个磁头(header)。

*\~/pictures/disk.png*

以前老是看到有IDE, SATA之类的，不知道是什么，现在该知道了。

它们其它是一种磁盘数据传输接口。

-   IDE, integrated drive
    electronics，电子集成驱动器，早期的计算机多采用这种

接口，速度较慢。

-   SATA, serial advanced technology
    attachment，串行高级技术附件，速度较快，

现在的计算机几乎都弃用IDE而采用SATA了。

-   SCSI, small computer system interface,
    速度非常快，但是也非常贵，主要用

于服务器。

目前主要的硬盘制造商（引用自[维基](http://zh.wikipedia.org/wiki/%E7%A1%AC%E7%9B%98#.E7.A1.AC.E7.9B.A4.E8.A3.BD.E9.80.A0.E5.95.86)
）：西部数据(western digital)，希捷(seagate), 东芝(toshiba)。

BIOS
====

basic input/output system，用于系统设置信息，开机自检程序和自启程序。
目前主流的BIOS有Award BIOS, AMI BIOS, Phoenix BIOS, Insyde BIOS。
