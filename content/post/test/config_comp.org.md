---
date: 2013-03-17
title: 配置测试与兼容性测试
---

@&lt;font color="red"&gt; 本文为"software testing"一书的读书笔记
@&lt;/font&gt;

配置测试
========

其实是有关硬件的测试。需要测试的内容：

-   PC类型（戴尔的？联想的？苹果的？）
-   PC部件（不同的主板，CPU，声卡，显卡，网卡等）
-   外围设备（鼠标，键盘，打印机，摄像头等）
-   接口（USB，PCI，PS/2等）
-   存储量大小（比如显卡有显存大小，内存条有内存大小）
-   设备驱动（该设备的驱动程序有没有？）

在进行配置测试时，可能需要买（或者借）很多种品牌的硬件来测试，比如可能需要
几百种声卡和显卡，要测试完所有的种类是不可能的，这个时候就需要等价划分了，
或者只测试主流的设备。

要了解这些PC的主流硬件，需要去搜索相关资料，有获取信息的途径，比如PC
Magzine 和Mac
World。在同一个等价划分里面的硬件没必要都测试，应该测试那些与众不同的
特性。

怎么获取这些硬件呢？首先，那些经常要用到的硬件应该直接买下来。第二种办法，联系
厂家问他们能不能借来测试。第三种办法，问公司的同事借。当然，你应该要找PM要相关
的经费。

兼容性测试
==========

上面是与硬件的兼容，现在是与其他软件的兼容。

-   向后兼容：该软件与低版本的其他软件兼容。
-   向前兼容：该软件与未来版本的其他软件兼容。

选择要兼容的软件的方法：

-   看这方面软件的流行度
-   看软件的年龄
-   软件的类型
-   厂家

通过这些方法来确定等价划分。