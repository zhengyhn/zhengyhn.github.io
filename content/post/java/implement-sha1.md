---
date: 2020-02-23
title: 实现一下SHA1算法
tags: ['Java', 'SHA1', 'SHA', 'Security']
categories: ['Java']
---

由于工作中经常会接触到数字签名，摘要算法等知识。现在准备实现一个最基础的摘要算法: SHA1。

### rfc3174

算法的文档在[RFC3174](https://tools.ietf.org/html/rfc3174)。这一份 standard 里面定义了很多规范，我们一个一个看。

#### 输入内容长度限制和输出内容的长度

```
When a message of any length < 2^64 bits is input, the SHA-1 produces a 160-bit output called a message digest.
```

输入的内容转成二进制之后，长度必须小于 2^64 位。由于这是一个 Hash 算法，输出的长度是固定，SHA1 的输出长度是 160 位的二进制。

#### 输入和输出

对于使用方，我们一般是输入一段字符串给 SHA1，结果是得到一串固定长度的十六进制数字组成的字符串。
由于输入的字符串是带字符编码的，所以我们需要先根据编码转成字节数组，再交给 SHA1 处理。
从 SHA1 的视角看，只认字节，它会对输入的字节数组进行一系列的处理，输出一个 20 个字节的数组，一共 160 位。

最后，通过这 20 个字节的数组，每个字节转换成十六进制的字符串，拼接进来，就是最终的结果。

```

public String digest(String text) {
    byte[] result = hash(text.getBytes(StandardCharsets.UTF_8));
    StringBuilder stringBuilder = new StringBuilder();
    for (byte b : result) {
        stringBuilder.append(Integer.toHexString(b & 0xff));
    }
    return stringBuilder.toString();
}

```

#### Padding 算法

SHA1 算法中 512 位称为一个 block，padding 算法就是为了将输入的数据补全为 512 位的 n 倍。

比如：输入 a, 一共 1 个字节，1 \* 8 = 8 位，不足 512 位，所以后面要补 512 - 8 = 504 位

至于要补什么内容，规则是这样的:

- 先在内容的后面补一个 1，这个 1 是指二进制的 1，补的是 1 位，所以转成字节就是 1000 0000, 十六进制就是 0x80
- 再在末尾的 64 位补上输入内容的长度

以刚刚 a 的例子来说，a 对应的 ascii 码为 97，转成二进制为 01100001, 所以初始内容为 01100001,

后面补上二进制 1 之后，为 01100001 10000000,

长度是 8 位，转换为二进制就是 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00001000

最后的结果就是一个 512 位的二进制，01100001 10000000 ... 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00001000

中间的...都是补 0.

有 2 个特殊 case:

- 如果内容刚好是 512 位的倍数，需要 padding 吗？答案是需要的，需要再往后面补多 512 位
- 如果内容的长度还差 1 ~ 9 位才到达 512 位的倍数，怎么办？答案是，需要往后面再补多 512 位

实现的代码如下：

```

long bits = originalBytes.length * 8;
int newLength = ((originalBytes.length / 64) + 1) * 64;
if ((newLength - originalBytes.length) < 9) {
    newLength += 64;
}
byte[] bytes = new byte[newLength];
System.arraycopy(originalBytes, 0, bytes, 0, originalBytes.length);
bytes[originalBytes.length] = (byte) 0x80;
for (int i = bytes.length - 8; i < bytes.length; ++i) {
    bytes[i] = (byte) ((bits >>> (56 - (i - (bytes.length - 8)) * 8)) & 0xff);
}

```

### 处理算法

文档中有 2 种方法，我选择的是算法一。

先定义几个算法中会用到的函数:

- S^n(X), 对 X 进行循环左移 n 位的操作

```

S^n(X)  =  (X << n) OR (X >> 32-n)

```

- f(t;B,C,D), 根据 t 的不同取值，对 B, C, D 进行相应位运算

```

f(t;B,C,D) = (B AND C) OR ((NOT B) AND D)         ( 0 <= t <= 19)

f(t;B,C,D) = B XOR C XOR D                        (20 <= t <= 39)

f(t;B,C,D) = (B AND C) OR (B AND D) OR (C AND D)  (40 <= t <= 59)

f(t;B,C,D) = B XOR C XOR D                        (60 <= t <= 79)

```

- K(t), 根据不同的 t，返回相应的魔法数字

```

  K(t) = 5A827999         ( 0 <= t <= 19)

  K(t) = 6ED9EBA1         (20 <= t <= 39)

  K(t) = 8F1BBCDC         (40 <= t <= 59)

  K(t) = CA62C1D6         (60 <= t <= 79)

```

算法描述是这样的:

对 Padding 过后的数据，每 512 位执行下面的操作:

- 初始化 5 个魔法数字: H0 = 67452301, H1 = EFCDAB89, H2 = 98BADCFE, H3 = 10325476, H4 = C3D2E1F0
- 将 512 位分成 16 份，每份包含 1 个字(word)，即 32 位，记为 W[0] ~ W[15]
- 对于 t = 16 到 t = 79, 执行下面的操作
  - W(t) = S^1(W(t-3) XOR W(t-8) XOR W(t-14) XOR W(t-16)),
- 令 A = H0, B = H1, C = H2, D = H3, E = H4
- 对于 t = 0 到 t = 79, 执行下面的操作
  - TEMP = S^5(A) + f(t;B,C,D) + E + W(t) + K(t);
  - E = D; D = C; C = S^30(B); B = A; A = TEMP;
- 令 H0 = H0 + A, H1 = H1 + B, H2 = H2 + C, H3 = H3 + D, H4 = H4 + E

最终的结果是 H0 H1 H2 H3 H4 拼成 5 个字(word)，一共 160 位。

全部的代码实现如下:

```

public byte[] hash(byte[] originalBytes) {
    long bits = originalBytes.length * 8;
    int newLength = ((originalBytes.length / 64) + 1) * 64;
    if ((newLength - originalBytes.length) < 9) {
        newLength += 64;
    }
    byte[] bytes = new byte[newLength];
    System.arraycopy(originalBytes, 0, bytes, 0, originalBytes.length);
    bytes[originalBytes.length] = (byte) 0x80;
    for (int i = bytes.length - 8; i < bytes.length; ++i) {
        bytes[i] = (byte) ((bits >>> (56 - (i - (bytes.length - 8)) * 8)) & 0xff);
    }
    int[] hash = new int[]{0x67452301, 0xEFCDAB89, 0x98BADCFE, 0x10325476, 0xC3D2E1F0};
    int[] words = new int[80];
    for (int i = 0; i < bytes.length; ) {
        for (int t = 0; t < 16; ++t) {
            words[t] = (bytes[i] << 24) | ((bytes[i + 1] << 16) & 0xff0000) +
                    ((bytes[i + 2] << 8) & 0xff00) + (bytes[i + 3] & 0xff);
            i += 4;
        }
        for (int t = 16; t < 80; ++t) {
            words[t] = circularLeftShift(1, words[t - 3] ^ words[t - 8] ^ words[t - 14] ^ words[t - 16]);
        }
        int[] newHash = Arrays.copyOf(hash, hash.length);
        for (int t = 0; t < 80; ++t) {
            int temp = circularLeftShift(5, newHash[0]) + func(t, newHash[1], newHash[2], newHash[3]) +
                    newHash[4] + words[t] + key(t);
            newHash[4] = newHash[3];
            newHash[3] = newHash[2];
            newHash[2] = circularLeftShift(30, newHash[1]);
            newHash[1] = newHash[0];
            newHash[0] = temp;
        }
        for (int j = 0; j < hash.length; ++j) {
            hash[j] += newHash[j];
        }
    }
    byte[] result = new byte[hash.length * 4];
    int i = 0;
    for (int h : hash) {
        result[i++] = (byte) ((h >>> 24) & 0xff);
        result[i++] = (byte) ((h >>> 16) & 0xff);
        result[i++] = (byte) ((h >>> 8) & 0xff);
        result[i++] = (byte) (h & 0xff);
    }
    return result;
}

@Override
public String digest(String text) {
    byte[] result = hash(text.getBytes(StandardCharsets.UTF_8));
    StringBuilder stringBuilder = new StringBuilder();
    for (byte b : result) {
        stringBuilder.append(Integer.toHexString(b & 0xff));
    }
    return stringBuilder.toString();
}

private int circularLeftShift(int n, int x) {
    return (x << n) | (x >>> (32 - n));
}

private int func(int t, int b, int c, int d) {
    if (t >= 0 && t < 20) {
        return (b & c) | ((~b) & d);
    } else if (t < 40) {
        return b ^ c ^ d;
    } else if (t < 60) {
        return (b & c) | (b & d) | (c & d);
    } else {
        return b ^ c ^ d;
    }
}

private int key(int t) {
    if (t >= 0 && t < 20) {
        return 0x5A827999;
    } else if (t < 40) {
        return 0x6ED9EBA1;
    } else if (t < 60) {
        return 0x8F1BBCDC;
    } else {
        return 0xCA62C1D6;
    }
}

```

### 加到 GUI 工具

实现了算法之后，加到我以前写的一个 GUI 工具上面去，效果是这样的:

![效果图](/images/sha1.png)

### 源码

https://github.com/zhengyhn/crypto-tool

### 总结

跟着文档写其实不难，难在阅读理解，我在实现过程中被 int 和 byte 之间互转的问题卡了很久，实际上还是基础不扎实，最后看了 open jdk 的实现以及 debug 它和我的的实现的不同才最终解决所有问题。

不过 SHA1 现在已经不推荐使用了，因为出现了相同数据产生同样输出的碰撞情况，更推荐使用 SHA2 和 SHA256 等算法。不过，这依旧是哈希算法中最基础的算法，对我很有帮助。
