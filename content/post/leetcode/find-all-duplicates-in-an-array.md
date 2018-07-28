---
date: 2017-12-26
title: find-all-duplicates-in-an-array
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
Given an array of integers, 1 ≤ a[i] ≤ n (n = size of array), some elements appear twice and others appear once.

Find all the elements that appear twice in this array.

Could you do it without extra space and in O(n) runtime?

Example:
Input:
[4,3,2,7,8,2,3,1]

Output:
[2,3]
```

#### 思路
想了好久，终于想出来了。首先，题目有个条件：1 ≤ a[i] ≤ n (n = size of array)，说明方法很有可能跟下标有关。所以，要想办法将元素放到跟它下标对应的位置，如果不能放了，说明是重复的元素，在这个题目中，就是出现两次的元素。

例如：[4,3,2,7,8,2,3,1]

首先看第一个元素4，我们要把它放到第4个位置，现在是7，那2个数交换，变成：[7,3,2,4,8,2,3,1]
继续看第一个元素7，我们要把它放到第7个位置，现在是3，那2个数交换，变成：[3,3,2,4,8,2,7,1]
继续看第一个元素3，我们要把它放到第3个位置，现在是2，那2个数交换，变成：[2,3,3,4,8,2,7,1]
继续看第一个元素2，我们要把它放到第2个位置，现在是3，那2个数交换，变成：[3,2,3,4,8,2,7,1]
继续看第一个元素3，我们要把它放到第3个位置，现在是3，说明3是重复的，记下来，把第一个元素3改成0，变成：[0,2,3,4,8,2,7,1]
继续以同样的方法往下遍历


#### 代码
```
class Solution {
  public:
    vector<int> findDuplicates(vector<int>& nums) {
      vector<int> ret;
      for (int i = 0; i < nums.size(); ++i) {
        if (nums[i] > 0 && nums[i] != (i + 1)) {
          if (nums[nums[i] - 1] == nums[i]) {
            ret.push_back(nums[i]);
            nums[i] = 0;
          } else {
            int temp = nums[i];
            nums[i] = nums[nums[i] - 1];
            nums[temp - 1] = temp;
            --i;
          }
        }
      }
      return ret;
    }
  }
}
```
