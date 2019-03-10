---
date: 2019-03-10
title: String permutations
tags: ["Algorithm"]
categories: ["Algorithm"]
---

There is an interview problem. Given a string without duplicate characters, return all permutations of the string.

#### First Try
The straightforward idea is using recursive algorithms.
We enum all characters for the first position and concatate the permutations of the rest substring.

However, the speed is too slow when the string is a little logger, like "abcdefghi".

So, we have to try the iterative solution.

There is an algorithm call "Next Permutation".

### Next Permutation
The idea of this algorithm is great!

First, from end to start, it find the position **i** where the former character is less than the current character.

Second, from end to the position **i** found just now, it find the position **j** where the character at **i - 1** is less than the character at **j**.

Then swap them.

After that, the character from **i** to the end must have the order Bigger to Smaller. And we now reverse them.

The result now is the next permutation of the string.

When we find all next permutations of the given string until the next permutation become the string itself.

Now we have all the permutations of the string.

#### Show me the code
```
    public String nextPermutation(String string) {
        char[] chars = string.toCharArray();
        int i = chars.length - 1;
        for (; i > 0; --i) {
            if (chars[i] > chars[i - 1]) {
                break;
            }
        }
        if (i > 0) {
            int j = chars.length - 1;
            for (; j >= i; --j) {
                if (chars[j] > chars[i - 1]) {
                    break;
                }
            }
            swapChar(chars, j, i -1);
        }
        int end = chars.length - 1;
        while (i < end) {
            swapChar(chars, i++, end--);
        }
        return new String(chars);
    }

    public List<String> permuteStringIter(String str) {
        List<String> list = new ArrayList<>();
        if (str.length() <= 0) {
            return list;
        }
        String string = str;
        list.add(string);
        while (true) {
            String nextString = this.nextPermutation(string);
            if (nextString.equals(str)) {
                break;
            }
            list.add(nextString);
            string = nextString;
        }
        return list;
    }
    private void swapChar(char[] nums, int i, int j) {
        if (i == j) {
            return;
        }
        char temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
```
#### Test
I test the string "abcdefgh" with two algorithms. The iterative one cost 69ms, but the recursive one cost 13322 ms.

#### Reference
- [leetcode](https://leetcode.com/problems/next-permutation/)
