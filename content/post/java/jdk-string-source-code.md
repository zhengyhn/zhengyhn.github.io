---
date: 2019-12-30
title: 初步学习JDK的String类源码
tags: ['Java', 'Java8', 'String', 'JDK']
categories: ['Java']
---

最近有空看了部分 String 类的源码，下面只挑重点部分记录一下。

### 为什么要用 final 修饰类

第一个比较特别的就是，String 类是用 final 修饰的，表明这个类不可被继承。

```
public final class String
```

为什么要防止被继承呢？经过一番研究和思考之后，目前比较靠谱的解析是：String 类是设计成不可变的。假设可以被继承，我们就可以写一个子类继承 String，然后覆盖它的部分方法，使得不可变性遭到破坏。最后，我们把这个子类传给使用方，使用方预期是一个不可变的 String 类，实际得到的是一个可变的子类，这样就出问题了。

### 使用 char value[]存储字符

```
    private final char value[];
```

跟预期一样

### 继承了 Serializable, Comparable 和 CharSequence 接口

IO 数据时字符串很常用，继承了 Serializable，就可以序列化成字节和从字节反序列回字符串。

字符串是可以比较的，实现了 Comparable 接口。看了一下实现，有个地方跟我想象中的不一样。

```
    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```

如果某个字符串 A 长度小于另外一个字符串 B，而前面的字符又相等，那是认为 A 小于 B 的。举个例子，"abc"小于"abcd".

看了一下 CharSequence 接口，有下面几个核心方法:

```
    int length();
    char charAt(int index);
    CharSequence subSequence(int start, int end);
    public String toString();
```

阅读 subSequence 方法的时候发现了很多宝藏。

首先 subSequence 很少用，原来 String 类让它直接调用了 substring 方法。

```
    public CharSequence subSequence(int beginIndex, int endIndex) {
        return this.substring(beginIndex, endIndex);
    }
```

在 substring 方法里面，我之前以为不管怎么样一定会返回一个新的 String 对象，其实并不是。当子子字符串就是原字符串时，返回本身，不然才新创建了一个对象。

```
return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
```

看到有一些边界检查，会抛出常见的 StringIndexOutOfBoundsException 异常，这里发现，原来每一个异常类都应该要加上 serialVersionUID，因为基类 Throwable 就实现了 Serializable，本意是希望 IO 过程中可以将异常出转成字节。

```
public class Throwable implements Serializable {
    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -3042686055658047285L;
```

这个之前在项目中自定义异常时没有注意到这一点，一直都没加 serialVersionUID.

最后，StringBuilder 和 StringBuffer 也实现了 CharSequence 接口

### Trim 方法到底去掉哪些字符？

我以为只会去掉空格和 TAB, 回车，换行这些，原来不是这样的，而是 ascii 码表里面小于等于空格的字符都去掉，其实就是不可见字符。

```
        while ((st < len) && (val[st] <= ' ')) {
            st++;
        }
        while ((st < len) && (val[len - 1] <= ' ')) {
            len--;
        }
        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
```

附一张 ascii 码表:
![ascii](/images/asciifull.gif 'ascii')

先学习这么多, 更深入的内容，下次再看。
