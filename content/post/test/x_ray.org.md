---
date: 2013-03-17
title: 带上X光眼镜测试软件
---

@&lt;font color="red"&gt; 本文为"software testing"一书的读书笔记
@&lt;/font&gt;

动态白盒测试
============

显然，动态即要运行，白盒即要看到代码。dynamic white-box testing，
也叫structural testing.

这听起来和调试很相似，但是它和调试还是有很大区别的。

-   动态白盒测试是要找到bug,而调试是要fix the bug

解决一个问题的过程就包含了动态白盒测试和调试，因此两者之间有一部分重合，

分块测试
========

-   单元测试(unit
    testing)：把整个软件分成很多的小模块，对每个小模块的测试
-   集成测试(integration
    testing)：把一些模块集成在一起，对一组模块进行测试
-   系统测试(system testing)：把所有的模块合并成一个系统进行测试

主要有两种测试方法：自底向上测试(bottom-up
testing)和自顶向下测试(top-down
testing).对于bottom-up，需要自己写测试模块，称为测试驱动(test driver).
