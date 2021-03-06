+++
categories = ["编程"]
date = "2016-05-27T20:40:43+08:00"
tags = ["编程"]
title = "improve code readability"

+++

这是我读 "the art of readable code" 一书做的笔记

简化循环和逻辑
==============

making control flow easy to read
--------------------------------

下面的代码:

    if (length >= 10)

要比:

    if (10 <= length)

更容易看懂。这是很显然的。而在C语言中，有的人为了避免=与==的错误，常常把
代码写成:

    if (10 == length)

这种做法是为了避免错误的。其实在我看来，这明显是在掩饰自己区分不了=与==的
弱点，如果你真的理解赋值与相等的含义，就从来不会在比较相等的时候写成=，以
我使用C语言这么多年来，我从来没有犯过这种错误。上面的代码对于一个正常人来
说，很难理解，因为程序员首先是一个人，自然的语言是“长度等于10”，而不是
“10等于长度”，所以为了使代码更可读，我建议使用下面这种:

    if (lenght == 10)

上面的是有数字，有常量的比较，下面这个:

    while (bytes_received < bytes_expected)

要比:

    while (bytes_expected > bytes_received)

更容易看懂，因为第一个的阅读顺序符合人类的自然语言。

-   对于三目运算符?:，当表达式很长时不要使用。
-   不要使用do/while。当你阅读do/while的时候，因为你刚开始不知道条件，你会
    把循环的主体阅读两次。我几乎不写do/while，而且我非常讨厌看do/while代码。
-   尽量不要使用嵌套的if/else，想尽办法使得只有一个层次。

breaking down giant expressions
-------------------------------

利用宏来简化代码。看下面的例子:

    void AddStats(const Stats &add_from, Stats *add_to)
    {
        add_to->set_total_memory(add_from.total_memory() + 
            add_to->total_memory());
        add_to->set_free_memory(add_from.free_memory() + 
            add_to->free_memory());
        add_to->set_swap_memory(add_from.swap_memory() + 
            add_to->swap_memory());
    }

不管是谁，看到这样的代码都会头晕，但是你会发现其实它们都在做同一件事:

    add_to->set_XXX(add_from.XXX + add_to->XXX);

于是，通过定义带参数的宏，可以简化成这样:

    void AddStats(const Stats &add_from, Stats *add_to)
    {
        #define ADD_FIELD(field) \
            add_to->set_#field(add_from.#field() + add_to->#field())
        ADD_FIELD(total_memory);
        ADD_FIELD(free_memory);
        ADD_FIELD(swap_memory);
    }

这样不仅视觉上看起来舒服，而且理解起来非常容易。

variables and readability
-------------------------

-   可去除一些多余的变量
-   尽量缩小变量的范围，即使是全局变量，这样才能让程序更清晰。
-   在C++中，有这样一个例子:

        int size = list.size();
        if (size > 0) {
            cout << size << endl;
        }

假设后面再也没有用到size了，但是阅读代码的人会一直把这个变量记住，因为
他以为后面还会用到这个变量。我们可以把它改成这样:

    if ((int size = list.size()) > 0) {
        cout << size << endl;
    }

当然，在C语言中，这需要C99的支持，当读者看完这段代码时，就会忘记这个变量，
因为后面已经用不上了。
作者是这样说的，但是我觉得这不太可能，因为写代码的人不可能预知未来，它总是
喜欢先把变量缓存起来，说不定以后还会用到，所以我不太赞成这种写法，除非是非常
简单，很明显后面用不上的变量。

-   尽量使变量只能改变一次(prefer write-once varaibles)

改善外层代码
============

packing informations into names
-------------------------------

### choose specific words

书上说到getPage(url);这个函数的名字，根据我以前的看法，这绝对是一个好名字，
但是作者却批评这种命名方式。作者说，get太模糊了，我们看不出来它是从缓存中get还是
从数据库中get，还是从互联网上get。如果是从互联网上get,则应该使用fetchPage或者
downloadPage()。我觉得这个说法非常好，我以后给变量或者函数起名字的时候也要注意
这方面的东西了。

### avoid generic names like tmp and retval

作者批评了tmp,retval,foo这种词语，虽然我没用过retval和foo这种奇怪的名字，但是
我却用过tmp这东西，后来想了一下，tmp这种东西的确看不出来任何含义，就算时间紧迫，我
也不会用tmp这种变量了。但是，就像作者所说的一样，对于下面的代码:

    void swap(int *a,int *b)
    {
        int tmp = *a;
        *a = b;
        *b = tmp;
    }

这时候tmp则用得恰到好处，tmp在这里生命周期非常短，而且它的作用刚好是作为临时来用的。

对于计数器变量，我们经常使用i,j,k,x,y,z,a,b,c什么的，但是，正如作者所说，当有
多个计数器变量时，这种东西就经常会让人很难看懂了。看下面的例子:

    for(int i = 0; i < N; i++){
        for(int j = 0; j < M; j++){
            for(int k = 0; k < C; k++){
                if(school[i].teacher[k] == user[j]){
                    ...
                }
            }
        }
    }

school，teacher,user这种变量命名是很好的，但是i,j,k这种东西就很难懂了。我们怎么
知道i,j,k分别对应哪个数组的下标呢？我以前也经常写这种代码，也经常看到其他人写这种代码，
每当我看到这种代码时（就算是我自己的代码），我都觉得非常头疼，现在看了这本书，马上
醒悟过来，以后再也不写这种代码了！转而使用si,ti,ui，这样
school\[si\].teacher\[ti\] == user\[ui\] 就非常清楚了。

### prefer concrete names over abstract names

意思是不要抽象的名字，而是要具体的名字。以书上的一个例子说，
serverCanStart是抽象的名字，而canListenOnPort则是一个具体的名字

### attaching extra information to name,by using a suffix or prefix

如果我们有一个变量:

    string id;

而这个id必须是十六进制，这时，直接使用id就很不好，因此你看不出来它必须使用十六进制，
使用hex\_id代替就很明确了。

单位数值, 使用书上的例子，看下面的js代码:

    var start = (new Date()).getTime();
    var elapsed = (new Date()).getTime() - start;
    document.writeln("time is:" + elapsed + " seconds");

如果对js比较熟悉的，会知道，getTime()返回的是ms，而不是s，因此这样的命名会很容易
产生bug，不是每个程序员都会记得那么清楚getTime()返回的单位是什么。把这2个变量改成
start\_ms和elapsed\_ms就很清楚了！
看到这里，我决定，以后遇到这种有单位的变量，都要带个单位的后缀，以写出可读的代码。

### deciding how long a name should be

在较短的域里面可以使用较短的变量名，比如:

    if(i != k){
        int t = a[i];
        a[i] = a[k];
        a[k] = t;
    }

现在，打一个长变量名已经不是问题了。

我以前很恶心java的类名方法名很长，看起来不简洁，现在看多了也习惯了，而且现在打长的变量
名的确很简单了，因此现在很多编辑器或IDE都有自动补全的功能。比如我现在使用的Emacs就自带
有补全的功能（Alt +
/），我居然还不知道，因此我都是用auto-complete的，看来当它不起作用
时，我就可以手动地补全了。

有关缩略词,
作者说，对于工程项目的代码，最好不要写缩略词，因为新加进来的成员可能看不懂缩略词的意思，
而一些最常见的缩写，比如evaluation写成eval,string写成str，document写成doc，则写
成缩写比较好。其实，我觉得，像linux系统这么大的工程都使用了很多缩写，有的时候缩写还是
非常必要的，可能是我的个人原因，我不喜欢又臭又长的代码，我喜欢简洁的代码，对于unix哲学
中的缩写规则，我很感兴趣，在程序中还是要尽量使用缩写。

### using name formating to pack extra information

不同类型的实体命名特点, 在google
c++的规范中，类名首字母大写，使用驼峰式;宏常量名全部大写，使用下划线分隔;
const变量则首字母大写，驼峰式，区分宏常量;类的方法首字母大写，驼峰式;类的变量全小写，
最后要跟一个下划线;其它局部变量则全部小写，后面不跟下划线。
在html/css中，id一般使用下划线分隔，而class使用中线（dash）来分隔。

names that can't be misconstructed
----------------------------------

-   关键思想是：不断地问自己，这个名字别人会不会认为是其他意思？
-   很多情况对变量的命名是有包含与不包含的意思的。这里列举出几种很常用的用法。
    -   包含的情况，使用min和max，比如说min\_items和max\_students
    -   两端包含，first和last。

<table>
<colgroup>
<col width="11%" />
<col width="5%" />
<col width="5%" />
<col width="9%" />
<col width="5%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left">a</td>
<td align="left">b</td>
<td align="left">c</td>
<td align="left">d</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">first</td>
<td align="left"></td>
<td align="left"></td>
<td align="left">last</td>
<td align="left"></td>
</tr>
</tbody>
</table>

> -   包含前面，不包含后面，使用begin和end。

<table>
<colgroup>
<col width="11%" />
<col width="5%" />
<col width="5%" />
<col width="5%" />
<col width="8%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left">a</td>
<td align="left">b</td>
<td align="left">c</td>
<td align="left">d</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">begin</td>
<td align="left"></td>
<td align="left"></td>
<td align="left"></td>
<td align="left">end</td>
</tr>
</tbody>
</table>

-   在给布尔型变量命名时，注意不要带负面的意思，如不要使用:

        bool disable_ssl = false;

而要使用:

    bool use_ssl = true;

-   一般会使用is,can,has前缀命名布尔型变量。
-   函数的名字一定要和里面的操作相符，书上举了一个例子，比如函数getSum，这个函数的
    实现是计算一大堆数据的和，但是一般程序员第一眼看过去的时候就会以为仅仅是返回和，并
    没有想到里面会有代价很多的计算，很有可能会经常调用这个函数，这样就会使程序变得很慢。
    使用computeSum会使人更容易明白。
-   作者还举了STL里面的list::size()方法来批评。这个方法会一个节点一个节点地计算链
    表的长度，O(n)的速度很慢。看下面的例子:

        while (list.size() > max_size) {
            ...
        }

连写STL的程序员也有不规范的时候，一般的程序员会以为size()是O(1)的速度，直接返回
链表的长度，这样就会使得程序的速度非常慢了。如果改成countSize()会好很多，
但是幸运的是，作者说了，最新版的STL已经把size()变成了O(1)速度。

aesthetics
----------

举的第一个例子让我震惊！看下面的代码:

    public class PerformanceTester {
            public static final TcpConnectionSimulator wifi =
                    new TcpConnectionSimulator(
                        500,  /* Kbps */
                        80,   /* millisecs */
                        200,  /* jitter */
                        1     /* packet loss % */);
            public static final TcpConnectionSimulator t3_fiber =
                    new TcpConnectionSimulator(
                        4500,  /* Kbps */
                        10,    /* millisecs */
                        0,     /* jitter */
                        0      /* packet loss % */);
            public static final TcpConnectionSimulator cell =
                    new TcpConnectionSimulator(
                        100,  /* Kbps */
                        400,  /* millisecs */
                        250,  /* jitter */
                        5     /* packet loss % */);
    }

这里，缩进是对齐了，注释也对齐了，但是占用的行数太多，而且注释重复了3遍。
改成这样就好看多了:

    public class PerformanceTester {
            // TcpConnectionSimulator(throughput, lantency, jitter, packet_loss)
            //                          [Kbps]     [ms]      [ms]    [percent]

            public static final TcpConnectionSimulator wifi =
                    new TcpConnectionSimulator(500, 80, 200, 1);
            public static final TcpConnectionSimulator t3_fiber =
                    new TcpConnectionSimulator(4500, 10, 0, 0);
            public static final TcpConnectionSimulator cell =
                    new TcpConnectionSimulator(100, 400, 250, 5);
    }

作者建议代码要列对齐，我之前在programming windows一书中看到过这种
漂亮的代码，但是一直做不到，因为要打很多空格。后来发现emacs有通过正则表达式
对齐的功能align-regexp，爽极了！

knowing what to comment
-----------------------

### when should not comment？

看注释会浪费阅读代码的人的时间，注释还会占用屏幕的地方，导致读者时常要
翻页，所以，没有价值的注释不要写。下面的代码的注释都没有价值:

    // The class definition for Account
    class Account {
    public:
            // Constructor
            Account();

            // Set the profit member to a new value
            void SetProfit(double  profit);

            // Return the profit from this Account
            double GetProfit();
    };

因为，注释的含义从代码中已经可以看出来了，注释没有提供额外的信息，其实是和
代码重复了，不仅浪费地方，而且浪费写代码的人的时间和读代码的人的时间。
记住:

    good code > bad code + good comment

### what should comment be？

-   写自己的想法
-   写自己的代码的缺点，比如:

        // TODO:use a faster algorithm

有一些和TODO一样的很流行的标签

<table>
<colgroup>
<col width="12%" />
<col width="47%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left">Marker</td>
<td align="left">Typical meaning</td>
</tr>
<tr class="even">
<td align="left">TODO</td>
<td align="left">things to finish</td>
</tr>
<tr class="odd">
<td align="left">FIXME</td>
<td align="left">known-broken code here</td>
</tr>
<tr class="even">
<td align="left">HACK</td>
<td align="left">inelegant solution to a problem</td>
</tr>
<tr class="odd">
<td align="left">XXX</td>
<td align="left">danger!major problem here</td>
</tr>
</tbody>
</table>

-   解释常量为什么是那个值

making comments precise and compact
-----------------------------------

-   函数注释中，可使用举例说明，如:

        //Example:strip("abba/a/ba", "ab") returns "/a/"

-   要写明你的代码的意图。看下面例子:

        // Iterate through the list in reverse order
        for (i = SIZE; i >= 0; i--) {
            ...
        }

上面的注释写了和没写差不多，改成这样就非常好了:

    // display each price, from highest to lowest
    for (i = SIZE; i >= 0; i--) {
        ...
    }

-   调用函数的时候也可以注释，按照书上的例子，这样的代码:

        Connect(/* timeout_ms = */ 10, /* use_encryption = */ false);

要比这样的代码:

    Connect(10, false);

更容易看懂。

重构代码
========

extracting unrelated subproblems
--------------------------------

-   要封装一些和本功能无关的子问题到另外一个函数上。
-   对于一些工具的类或函数（字符串操作，哈希表等），就分离出来。

one task at a time
------------------

每个函数应该只实现一个功能，不要实现多个功能。

turning thoughts into code
--------------------------

记住爱因斯坦的这句话:

    You do not really understanding something unless you can explain it to
    your grandmother.               
                                                --Albert Einstein
    代码应该用易懂的英语来写。

writing less code
-----------------

-   一定要熟悉现存的库，这样可以减少代码量，多重用代码，少写代码。
-   使用Unix工具(shell命令），而不是自己写代码

写出易懂的选择排序
==================

以选择排序为例，我以前是直接把它记住的（当然是在理解的前提下），其中的一些
下标i,j,k，直接记住人家的代码的，如果要我重新写，我还是会用i,j,k，造成了
硬性的思维，现在看来，只要记住算法，用自己的方式来写出的代码才是好的代码。
下面的选择排序是经典的课本上的代码:

    void select_sort(int *arr,int len)
    {
        int i,j,k,temp;

        for(i = 0; i < len - 1; i++){
                k = i;
                for(j = i + 1; j < len; j++){
                        if(arr[j] < arr[k]){
                                k = j;
                        }
                }
                if(i != k){
                        temp = arr[k];
                        arr[k] = arr[i];
                        arr[i] = temp;
                }
        }
    }

我觉得，有经验的程序员看上面这份代码当然是没问题，但是给初学者来看，i,j,k分别
代表什么意思，他就会摸不着头脑了，所以，我们需要给变量的名字赋予意义，更容易
理解，于是我写了下面的代码:

    void select_sort(int *arr,int len)
    {
        int current,next,smallest,temp;

        for(current = 0; current < len - 1; current++){
                smallest = current;
                for(next = current + 1; next < len; next++){
                        if(arr[next] < arr[smallest]){
                                smallest = next;
                        }
                }
                if(current != smallest){
                        temp = arr[smallest];
                        arr[smallest] = arr[current];
                        arr[current] = temp;
                }
        }
    }

这样的话，单从变量名字就可以理解整个算法的思想了。虽然变量的名字是长了一点，
但是现在的编辑器或者IDE的自动补全功能这么强大，变量名太长这个已经不是问题了。
