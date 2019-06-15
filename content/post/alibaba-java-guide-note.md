---
title: 阿里java开发者手册读书笔记
date: "2019-04-08"
tags: ["java"]
categories: ["java"]
---

### POJO 类中布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误
- 心得：这个之前我写的代码的确会下意识地就用isXXX，现在遵守这个规范，因为一些RPC框架在反向解析的时候，“误以为”对应的属性名称是 deleted，导致属性获取不到，进而抛 出异常。

### 杜绝完全不规范的缩写，避免望文不知义
- 心得：反例举了2个例子，AbstractClass“缩写”命名成 AbsClass；condition“缩写”命名成 condi，此类随意缩写严重降低了代码的可阅读性。好吧，我就是把抽象类写成AbsXXX的，现在遵守这个规范，写全。

### Service/DAO 层方法命名规约
1. 获取单个对象的方法用 get 做前缀 
2. 获取多个对象的方法用 list 做前缀，复数形式结尾如：listObjects。
3. 获取统计值的方法用 count 做前缀。 
4. 插入的方法用 save/insert 做前缀。 
5. 删除的方法用 remove/delete 做前缀。 
6. 修改的方法用 update 做前缀。
- 心得：记录一下这些命名规范，之前是部分遵守

### 避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成本，直接用类名来访问即可
- 心得：虽然没看源码直接证明，但是我猜，使用对象访问静态方法或静态变量，先要根据对象找到类，再调用类的静态方法或静态变量。另外，直接用类名，其实可读性更好，因为看变量还得去看这个变量是什么类型的。

### Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals
- 正例："test".equals(object);
- 反例：object.equals("test");
- 说明：推荐使用 java.util.Objects#equals（JDK7 引入的工具类）
- 心得：新学习了Objects.equals，并知道了这样子判断equals可以避免空指针

### 所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较
- 对于 Integer var = ? 在-128 至 127 范围内的赋值，Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行 判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑， 推荐使用 equals 方法进行判断。
- 心得：这个的确是经常会忘记的

### 关于基本数据类型与包装数据类型的使用标准如下： 
1） 【强制】所有的 POJO 类属性必须使用包装数据类型。 
2） 【强制】RPC 方法的返回值和参数必须使用包装数据类型。
3） 【推荐】所有的局部变量使用基本数据类型。
- 比如显示成交总额涨跌情况，即正负 x%，x 为基本数据类型，调用的 RPC 服务，调用 不成功时，返回的是默认值，页面显示为 0%，这是不合理的，应该显示成中划线
- 心得：POJO类使用包装类型，是因为上面所说的基本类型的默认值，应该要遵守。局部变量使用基本数据类型，是因为性能更好

### ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常
- 说明：：subList 返回的是 ArrayList 的内部类 SubList，并不是 ArrayList 而是 ArrayList 的一个视图，对于 SubList 子列表的所有操作最终会反映到原列表上。
- 心得：学习了subList这一点，之前没遇到过

### 使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常
- 说明：asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList
体现的是适配器模式，只是转换接口，后台的数据仍是数组
- 心得：学习了asList这一点，之前有写过，但是没有注意这一点

### 使用 entrySet 遍历 Map 类集合 KV，而不是 keySet 方式进行遍历
- 说明：keySet 其实是遍历了 2 次，一次是转为 Iterator 对象，另一次是从 hashMap 中取出 key 所对应的 value。而 entrySet 只是遍历了一次就把 key 和 value 都放到了 entry 中，效率更高。如果是 JDK8，使用 Map.foreach 方法。
- 心得：学习了这一点，entrySet效率更高

### SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为static，必须加锁，或者使用 DateUtils 工具类
- 说明：可以使用synchronized或ThreadLocal来实现同步，或者用第三方类库。如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar， DateTimeFormatter 代替 SimpleDateFormat
- 心得：知道了SimpleDateFormat的问题，并要学会用Java8的新特性

### 避免 Random 实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一seed 导致的性能下降。
- 说明：在 JDK7 之后，可以直接使用 API ThreadLocalRandom，而在 JDK7 之前，需要编
码保证每个线程持有一个实例。
- 知道了Random的问题，并要学会用Java7的新特性

### volatile 解决多线程内存不可见问题。对于一写多读，是可以解决变量同步问 但是如果多写，同样无法解决线程安全问题。如果是 count++操作，使用如下类实现： AtomicInteger count = new AtomicInteger(); count.addAndGet(1); 如果是 JDK8，推荐使用 LongAdder 对象，比 AtomicLong 性能更好（减少乐观锁的重试次数）
- 心得：学习了volatile的一写多读和多写的适应场景

### 获取当前毫秒数 System.currentTimeMillis(); 而不是 new Date().getTime()
- 说明：因为System.currentTimeMillis()速度更快

### finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch
- 说明：如果 JDK7 及以上，可以使用 try-with-resources 方式
- 心得：学习了JDK7的新特性，以及异常屏敝，参考文章：https://juejin.im/entry/57f73e81bf22ec00647dacd0

### 对 trace/debug/info 级别的日志输出，必须使用条件输出形式或者使用占位符的方式。
- 说明：logger.debug("Processing trade with id: " + id + " and symbol: " + symbol);如果日志级别是 warn，上述日志不会打印，但是会执行字符串拼接操作，如果 symbol 是对象， 会执行 toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。使用占位符方式不会浪费资源：logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);
- 心得：学习了这种方式会浪费系统资源

### 表达是与否概念的字段，必须使用
is_xxx的方式命名，数据类型是unsigned tinyint（1表示是，0表示否）。
- 表达逻辑删除的字段名is_deleted，1表示删除，0表示未删除
- 心得：学了一招，立马用到项目中

### 主键索引名为 pk_字段名；唯一索引名为 uk_字段名；普通索引名则为 idx_字段名
- 心得：之前没注意索引命名的规范，学习了

### 小数类型为 decimal，禁止使用 float 和 double
- 说明：float 和 double 在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不 正确的结果。如果存储的数据范围超过 decimal 的范围，建议将数据拆成整数和小数分开存储
- 心得：例如金额

### 不得使用外键与级联，一切外键概念必须在应用层解决
- 说明：以学生和成绩的关系为例，学生表中的 student_id 是主键，那么成绩表中的 student_id 则为外键。如果更新学生表中的 student_id，同时触发成绩表中的 student_id 更新，即为 级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻 塞，存在数据库更新风暴的风险；外键影响数据库的插入速度
- 心得：以前我建表都是会指定外键，现在学精了
