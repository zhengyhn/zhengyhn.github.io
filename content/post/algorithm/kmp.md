---
date: 2022-07-17
title: KMP算法-我的理解
tags: ['Algorithm']
categories: ['Algorithm']
---

# 动机

看了一本叫做《C/C++面试题》的电子书，里面提到找子字符串的算法，最好的是KMP，
于是开始了KMP之旅！

在网上看了好几篇中文文章，没一篇看得懂，最后找到了2篇英文文章，深有启发：

-  [searching-for-patterns-set-2-kmp-algorithm](http://www.geeksforgeeks.org/searching-for-patterns-set-2-kmp-algorithm/)
-  [the-knuth-morris-pratt-algorithm-in-my-own-words](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)

特别是第2篇，让我深刻理解了KMP中的预处理，而第1篇则让我学会了如何用高效的方法
实现预处理。

于是，我也来写一篇文章，用自己的理解来实现KMP算法。

# 在哪里优化？

如果要让我写一个程序来找到子串，唯一能想到的方法就是一个一个找，这叫做暴力算法。
KMP算法的作者，他其实是发现了下面的规律。比如，有字符串str：

``` {.example}
ababaabcbab
```

需要找的子字符串，称为 **模式** ，pattern，为：

``` {.example}
ababab
```

匹配的时候，会发现，就差最后一个a就匹配成功了，太可惜了！按照传统的做法，我们会
从str右移一下，再从头比较，但是很明显，str的第二个字母b和pattern的第一个
字母a不同，我们应该直接移2下，因为str中的第3个字母到第5个字母为aba，而
pattern开头的3个字母为aba，这样匹配就不用重复比较前面已经比较过的地方了。这是偶然？还是真理?

考虑str的前面6位，当pattern匹配到最后的a时，发现不匹配，于是应该往前退回
3步，正好跳过最前面的ab，到达abaa。这样就省去了一次查找。这正是KMP算法对
普通算法的优化之处。可是这个3是怎么算出来的呢？下面来探索一下。

# 部分匹配表

这个名字是*the-knuth-morris-pratt-algorithm-in-my-own-words* 这篇文章的
作者起的，我觉得比较好理解，于是拿过来了。

对每个模式，它自身的内容有一些规律，下表就是所谓的partial match table:


   pattern |  a |  b |  a |  b |  a |  b |  c |  a 
  :--------|:---|:---|:---|:---|:---|:---|:---|:---
  index    | 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  
  value    | 0  | 0  | 1  | 2  | 3  | 4  | 0  | 1  


可以看到，第5个字符a，它的value是3，这个3就是上面要回退的步数，所以，这个算法的核心是计算出来这个部分匹配表。
要计算这个表中的value，必须先理解下面的概念：

1.  **真前缀** :
    一个字符串，忽略最后一个字符，前面的任意长度的子字符串。如
    "monkey"的真前缀有"m", "mo", "mon", "monk", "monke"。
2.  **真后缀** :字符串，忽略第一个字符，后面的任意长度的子字符串。如
    "monkey"的真后缀有"y", "ey", "key", "nkey", "onkey"。

那么部分匹配表中的value就是：

``` {.example}
以当前位置结束的前面子串的所有真后缀和真前缀中，公共的那些里面，最长的子串的长度
```

比较难懂，解释一下，如果要求第5个字符a的value，则我们只关心前5个字符
ababa，它的真前缀有a, ab, aba, abab真后缀有a, ba, aba, baba, 则真前缀
和真后缀一样的有：a, aba，而最长的是aba，它的长度是3,于是value为3。

那凭什么这样子算出来的value为3，就是要回退的步数呢？原理是这样的：

对于模式ababab的子串ababa，它的真前缀为下面加粗的部分:

**a**baba

**ab**aba

**aba**ba

**abab**a

真后缀为下面加粗部分:

abab**a**

aba**ba**

ab**aba**

a**baba**

真前缀和真后缀一样的有a和aba，放在一起加粗显示来看:

**a**baba abab**a**

**aba**ba ab**aba**

回到我们的例子中，有字符串ababaabcbab，现在要用模式ababab来匹配，一开始是这样的：

```
ababaabcbab
|||||
ababab
```

前面5个都匹配上了，现在我们要往后移动进行下一次匹配。暴力的方法是往后移动一位。而实际上我们观察到，由于前5个字符是匹配上的，
所以原字符串前5个字符组成的子串跟模式的前5个字符组成的子串是一样的，那我们当成这2个子串都是模式，看一下他们相同的真前缀和真后缀。

长度为3的相同真前缀和真后缀:

ab**aba**abcbab

**aba**bab

长度为1的相同真前缀和真后缀:

abab**a**abcbab

**a**babab

现在有两种移动方法：

这样:

ab**aba**abcbab

\_\_**aba**bab


和这样:

abab**a**abcbab

\_\_\_\_**a**babab

当然是选择第一种，因为第一种离起点更近，选择第二种会漏掉第一种匹配的情况，所以我们才要选择最长的公共真前缀和真后缀。

通过查看公共的真前缀和真后缀，我们知道，跳去哪个位置能够让下一次匹配开始，并且事先知道已经匹配了多少个字符。


# 如何快速计算部分匹配表

如果按照上面所说的办法，把真前缀和真后缀穷举出来，然后看哪些相等，再看哪个最长
的话，那这个算法的效率就太低了，我们有更好的办法来计算。

先看看上面的那个表的每一个value是怎么算出来的。

   pattern |  a |  b |  a |  b |  a |  b |  c |  a 
  :--------|:---|:---|:---|:---|:---|:---|:---|:---
  index    | 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  
  value    | 0  | 0  | 1  | 2  | 3  | 4  | 0  | 1  


1.  第一个必然是0,因为没有真前缀和真后缀。
2.  第二个就要看第一个的情况，因为第一个是0,而b又不等于a,于是还是0。
3.  第三个就看第二个，第二个是0,而最后一个a等于第一个a，于是加1。
4.  第四个就看第三个，因为是1,说明第3个a和第一个a相等这个事实已经在
    上一步确定了，我们不用再浪费时间来确定，我们只需要比较第4个b和第
    2个字符是不是一样的，发现是一样的，加1。
5.  同理，加1。
6.  同理，加1。
7.  第7个字符是c，上一步算出来是4,于是我们还是很贪心，先试试可不可以再增加，
    即看一下ababc行不行，发现c和a不相等，不行了，那就变小吧，这个时候就要
    往前看，即试试babc行不行，明显最后一个c和abab的最后一个b不匹配,再往前
    看，看一下abc行不行，很可惜，和aba的最后一个a不匹配，同理，bc不行，单独
    的c也不行。于是这里是0。
8.  因为上一步是0,
    那长度比1大的肯定不用比较了，直接用a和第一个a比较，发现
    一样，则值为1。

这就是计算部分匹配表的算法，参考自[这篇文章](http://www.geeksforgeeks.org/searching-for-patterns-set-2-kmp-algorithm/)
，这个表其实就是这个算法的关键。

# 查找

光计算出来那个表是没用的，重要的是利用这个表来快速查找子串。方法为：

``` {.example}
假设部分匹配表为part_tab
- 当不匹配时，直接往后搜索
- 当前面匹配了n个，中间某个位置i不匹配时，则i要回退part_tab[n - 1]个位置，
  接着匹配下一个
- 当全部匹配时，匹配成功，返回起始位置
```

为什么要回退part\_tab\[n - 1\]个位置呢？假定从位置k就开始匹配，则这个位置就是
pattern拥有前n个字符的子串的真前缀位置，part\_tab\[n - 1\]中保存的就是pattern
拥有前n个字符的子串的最长前后前缀的长度，在i位置，回退part\_tab\[n - 1\]就刚好
跳过了无用的字符，到达下一个和位置k拥有一样的真前缀的位置。这样开始下一轮的查找
才有意义。

# 代码实现

下面是我的简单实现，只经过少量测试。

``` {.c}
#include <stdio.h>
#include <string.h>

// if found PATTERN in str, return the first position
// else return -1
int index_of(char *str, char *pattern);

int main(int argc, char *argv[])
{
     char *str = "aababaacaabaa";
     char *pattern = "aabaa";

     printf("%d\n", index_of(str, pattern));

     return 0;
}

int index_of(char *str, char *pattern)
{
     int len = strlen(str);
     int plen = strlen(pattern);
     int part_tab[plen];
     int i, j, longest, match_len;

     // calculate the partial table
     part_tab[0] = 0;
     longest = 0;
     for (i = 1; i < plen; ) {
      if (pattern[i] == pattern[longest]) {
           longest++;
           part_tab[i] = longest;
           i++;
      } else {
           if (longest == 0) {
            part_tab[i] = 0;
            i++;
           } else {
            longest = part_tab[longest - 1];
           }
      }
     }

     // find the matching string position
     i = j = 0;
     match_len = 0;
     while ((i + plen - j) <= len) {
      if (j == plen - 1 && str[i] == pattern[j]) {
           return i - plen + 1;
      }
      if (str[i] == pattern[j]) {
           i++;
           j++;
           match_len++;
      } else {
           if (match_len > 0) {
            i = i - part_tab[match_len - 1];
            match_len = 0;
           } else {
            i++;
           }
           j = 0;
      }
     }
     return -1;
}
```

# 总结

这个算法，关键于是理解它是怎样从O(n \* m)优化成O(n + m)的，理解在哪里优化了。
并且要知道辅助数组part\_tab的用处及计算方法，后面的查找就很简单了。
