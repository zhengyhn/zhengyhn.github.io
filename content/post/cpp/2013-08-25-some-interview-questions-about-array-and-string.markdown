+++
date = "2016-05-14T16:35:19+08:00"
draft = false
title = "Some interview questions about array and string"
+++

Hash Table
------------
In C++, the namespace``std::tr1`` contains lots of hash tables. We can play
with it.

{% include_code careerup-array/careerup-hash.cpp %}

Questions
-----------
**1.1.** Implement an algorithm to determine if a string has all unique
characters. What if you can not use additional data structures?

I have come up with 2 methods.

* Iterate the string and compare each character to other character. This is the
common way to solve this problem. It will cost (O ^ 2) time.

* Use the ``std::tr1::unordered_set`` data structure so that we can only iterate
the string once.

Here is the code.

{% include_code careerup-array/careerup-1.1.cpp %}

However, there are two more effecient methods.

* If the string contains only ASCII(Ask the interviewer!), we can use an array
of size 256 to mark every character. The time complexity is O(n). But this is
to sacrifice space for time, just as the unordered_set method.

* If the string contains only letters, we can use a 4-bytes integer to mark the
characters in the string.

Here is the code.

{% include_code careerup-array/careerup-1.1-answer.cpp %}

**1.2.** Write code to reverse a C-Style String(C-String means that "abcd" is
represented as five characters, including the null character)

The approach I come up with is described as follows.

1. Get the length of the string, which needs iterate the whole string.

2. According to the property of continuing, we can manipulate the string
from the end. So we can swap the first character with the last character,
and swap the second character with the last but not least character, ...
This needs iterate half of the string.

In sumary, this will need O(1.5n) -> O(n) time complexity. Here is the code.

{% include_code careerup-array/careerup-1.2.cpp %}

However, the answer is using pointer instead of index. But the algorithm and
the time complexity are the same. Here is the code.

{% include_code careerup-array/careerup-1.2-answer.cpp %}

This code has several problems. First, the variable ``end`` and ``tmp`` should
be defined in the ``if`` block. Otherwise, if ``str`` is ``NULL``, the
definition of the two variables will be wasteful. Second, using the pointer with
the ``++`` and ``--`` is error-prone.

**1.3** Design an algorithm and write code to remove the duplicate characters
in a string without using any additional buffer. NOTE: One or two additional
variables are fine. An extra copy of the array is not.

FOLLOW UP

Write the test cases for this method.

I am not so smart that I can only come up with the straightforward method. Just
iterate the whole string and check from the former characters to find duplicate
character. If there is duplicate, remove it by moving the rest characters front.
Here is the code.

{% include_code careerup-array/careerup-1.3.cpp %}

The algorithm is ok but the program is too slow. In fact, the last ``for`` loop
is not necessary. Here is the improved version according to the answer.

{% include_code careerup-array/careerup-1.3-answer.cpp %}

**1.4** Write a method to decide if two strings are anagrams or not.

From [Wikipedia](http://en.wikipedia.org/wiki/Anagram), An anagram is a type
of word play, the result of rearranging the letters of a word or phrase to
produce a new word or phrase, using all the original letters exactly once.

For example, ``"abc"``, ``"acb"``, ``"bac"``, ``"bca"``, ``"cab"`` and ``"cba"``
are all anagrams.

I come up with two approaches.

1. Sort them and compare them. This will require at least O(nlogn) * 2 + O(n)
time complexity.

2. Use an extra array to record the occurrence of each character when compare
the two string.

I implemented the last method here.

{% include_code careerup-array/careerup-1.4.cpp %}

**1.5** Write a method to replace all spaces in a string with '%20'.

There is a ``replace`` method in class ``std::string`` , so I can use it
directly. But it's too slow(O(n^2)). There is another method with the time
complexity O(n). Count the number of the spaces and resize the string and
copy the string from tail to head. If encounter the space, replace it with
``%20``. Here is the code of the two methods.

{% include_code careerup-array/careerup-1.5.cpp %}

**1.6** Given an image represented by an NxN matrix, where each pixel in the
image is 4 bytes, write a method to rotate the image by 90 degrees  Can you
do this in place?

The first time I looked at this problem, I think it's very simple and write
down an equation ``a[i][j] = a[n - j - 1][i]``. When I implemented the program
using this equation, I found that I was totally **wrong**!

Since you can't request another space to store the matrix that was unmodified,
the modified value will affect the value to be changed. So we should think about
other solutions. One effective solution is to divide the whole matrix into
``N / 2`` parts and rotate each parts by swapping the values four times.

Here is the code.

{% include_code careerup-array/careerup-1.6.cpp %}

**1.7** Write an algorithm such that if an element in an MxN matrix is 0, its
entire row and column is set to 0.

I made the same mistake as problem **1.6**. I think we should just iterate the
whole matrix and set its entire row and row to 0 when we met an element that is
0. But this is **wrong**! It will also affect the elements that haven't been
iterated.

One solution is to use another matrix to record the original matrix. But this
will need too much space.

Actually, we just need two array of length ``M`` and ``N`` respectively and
record whether there are an ``0`` in that row or column. If there are, set the
entire row or column to 0.

Here is the code.

{% include_code careerup-array/careerup-1.7.cpp %}

**1.8** Assume you have a method isSubstring which checks if one word is a
substring of another. Given two strings, s1 and s2, write code to check if
s2 is a rotation of s1 using only one call to isSubstring (i.e., "waterbottle"
is a rotation of "erbottlewat").

I am not so smart that I can't come up with the method to solve this problem
after a long time. The answer amazed me.

Just concatenate ``s1`` and check if ``s2`` is a substring of it. For example,
``waterbottle`` is a substring of ``erbottlewaterbottlewat``, so it's a rotation
of ``erbottlewat``.

Here is the code.

{% include_code careerup-array/careerup-1.8.cpp %}
