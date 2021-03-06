+++
categories = ["Design"]
date = "2017-03-31T10:50:49+08:00"
tags = ["Design"]
title = "浅谈后端业务系统设计"

+++

这里将说说个人对设计的想法，必然会有争议之处，内容包括系统之间设计，系统内部模块之间设计，细到每个函数的设计。

#### 系统设计
##### 不要为了拆系统而拆系统
一般我们会因为下面几个原因拆分系统：
* 这个模块被几个系统共用
* 这个模块压力很大，独立出来可以扩展分流，优化
* 这个模块代码量太大，需要很多人维护，独立出来，易于管理

拆分系统之后一般会产生下面的问题：
* 原系统调用独立出来的模块之间通讯会变慢，内存寻址 -> 网络通讯
* 开发联调成本增大，开发效率降低
* 团队沟通成本变大，开发效率降低

所以，在你的团队人员没有填充，工具方法上没有提高的情况下，不要随意拆分系统

##### 系统的资源应该保证私有
最常见的资源就是数据库数据了，数据库的实例可以各个系统共用，但是表一定要独立。
如果一张表可以被多个系统读写，说明这两个系统就不应该拆分，设计上有问题。
数据库表必须私有，如果其他的系统想要读写，必须通过接口的方式进行，这也是面向对象里面的封装思想。

曾经有家公司，其中一个系统A直接读了另外一个系统B的数据库表，一个月后，系统B上线了新功能，改造了表的结构，导致系统A出现严重bug，
直接经济损失7000多万，两个团队都有10多人，团队之间很少沟通，谁也不知道对方做了什么事情。

在团队很小的时候，可能任何改动大家都知道，一旦团队扩充了，如果资源不封装，就会产生这样的问题。

#### 系统内模块设计
##### 面向对象
世界是面向对象的，现代计算机是面向过程的。目前能接触到的计算机都是基于冯.诺依曼体系架构（计算，存储，输入，输出），很久很久以前有面向函数的Lisp机，
可是时代证明，它只适合做计算，而且性能并不高效，冯.诺依曼体系架构是经过时间的考验沉淀下来，因为世界并不全是计算，特别是互联网业务系统，大部分是输入输出和存储。
所以，函数式编程仅仅适用于很少量的计算中。
面向过程编程适合一个简单的模块，并不适合复杂的情景，在复杂的业务下，面向过程式编程将变得难以维护。

我以前维护过一个汇款模块，纯面向过程，1000多行，全局变量全部定义在前面100行，用户可以从四端(柜台，电脑版，网页版，手机银行）发起汇款，每端都有自己特殊的业务逻辑，
一开始写这份代码的时候只有两个端（柜台和网页），那个时候还是2000年左右，看起来写得比较简洁，问题不大，后来有了电脑版，手机银行，就有了很多if channel == xxx的代码，
越来越难维护，一个全局变量，有可能被下面的很多个过程改变。
其实，上面的例子，遇到的if else问题，在面向对象里面是可以通过多态来解决的，全局变量的问题，是可以通过封装来解决的。

###### 封装
**封装的本质是为了提高系统的可维护性**。如果一个私有方法被另外一个模块引用了，当你要改变你的私有方法的逻辑时，你就必须要考虑到使用到的地方会不会受影响，这增大了维护成本，
在面向对象里面，区分公有和私有，就是强制规定，公有方法，它是固定的，既然公开了，就不能改变，私有的方法，你内部可以灵活变化，这样就可以降低维护成本，提高开发效率。

###### 继承，组合
世界是继承的，组合的，一个产品组合包含了若干个产品，每个产品包含了多个资产，每个资产有它的利率，金额等，世界就是这样的，如果你的代码依照这个规律来组织，
任何一个团队新成员，都能在短时间内熟悉，并接手项目。

###### 多态
世界是多态的，上面的汇款就是一个例子。简单地说，多态就是用于减少if else，使得模块组织更清晰，更简洁，更容易维护。

###### 高内聚，低耦合
模块之间的交互尽量少，模块内部灵活调用。模块之间交互越多，系统复杂度越高，越难以维护。

###### 层级设计，杜绝循环依赖
各个模块之间的调用应该是单向的，如果出现双向，说明本身设计上就有问题。建议每增加一个模块时都由项目负责人审核。

#### 模块内设计
##### 不要为了拆函数而拆函数
一个函数做一件事情，一件事情这是个有争议的话题，一行代码也是一件事情，但是如果一个函数只有一行代码，抽出来的意义是什么呢？
函数越多，阅读代码的难度越大，因为你需要不断地跳转，归位，理解，合并。

##### 关键位置写上一点点注释
并不是所有人都能从函数名或变量名中看出来你的代码含义，请从他人的角度思考，而不要从自己的角度看待这个问题。
在比较特殊或一般人很难看懂的地方，写上注释，对自己负责，也对他人负责。代码在写在时候是给自己看的，剩余的所有时间都是给其他人（包含你自己）看的。

##### 写任何代码时都要考虑基本的性能问题
基本的性能问题，主要是执行速度，内存占用等。举例：
* 有没有多余的循环代码，O(n)的执行速度有没有可能换个算法变成O(logn)
* 有没有多余的内存占用，比如拷贝内存占用过大的对象，大数组等等
* 数据库操作时，会不会出现数据量很大的情况，会不会崩溃，有没有返回多余的数据等等

##### 写任何代码时都要考虑将来接手的人
比如，我这样写，会不会给看代码的人带来阅读上的困难？拆分这么多函数看代码的人会不会看起来很麻烦？代码没有一个换行，看代码的人会不会看得不清晰？

一个团队的效率高不高，很多时候体现在代码质量上，代码质量高，维护成本低，bug少，时间都花费在完成需求和提高质量上面，效率怎么会不高呢。
不像隔壁团队，天天在修bug，做个需求要一天，测试测出来一堆问题，一堆没有考虑到的依赖问题。


