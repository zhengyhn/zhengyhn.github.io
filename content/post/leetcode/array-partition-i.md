---
date: 2017-12-17
title: array-partition-i
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
Given an array of 2n integers, your task is to group these integers into n pairs of integer, say (a1, b1), (a2, b2), ..., (an, bn) which makes sum of min(ai, bi) for all i from 1 to n as large as possible.

Note:
n is a positive integer, which is in the range of [1, 10000].
All the integers in the array will be in the range of [-10000, 10000].
```

#### 思路一
要把整个数组分为一对一对的，然后每对取最小的那个，使最小的加起来值最大。可以想到，小的一定要跟小的配对，因为最小的那个必然要算进去，如果跟它配对的那个比较大，那这个大的就浪费了。所以，最佳的状态就是，整个数组从小到大排好序，然后我们取奇数项相加，得到的和就是结果。时间复杂度是O(nlogn)。

#### 思路一代码
```
    int arrayPairSum(vector<int>& nums) {
      if (nums.size() == 2) {
        return nums[0] < nums[1] ? nums[0] : nums[1];
      }
      std::sort(nums.begin(), nums.end());
      int sum = 0;
      for (int i = 0; i < nums.size(); i += 2) {
        sum += nums[i];
      }
      return sum;
    }
```

#### 思路二
思路一的代码提交上去可以Accept，但是效率太低。原来还有另外一种方法，可以在近似O(n)时间内解决这个问题。

这个方法叫做桶排序(bucket sort)。这其实是一种浪费空间换取时间的方法，并且需要在特定条件下才有用。假设给定一个数组[1, 4, 3, 2]，根据题目要求，数组最大长度是20000, 每个元素的值最小为-10000, 最大为10000，那么我们就可以开一个长度为20000的数组，初始化为0，然后遍历原数组，原数组的值作为新数组的下标，有就加1，然后我们在新数组从头开始遍历，就能直接输出“排好序”的数组了。既然能得到一个排好序的数组，跟思路一一样，直接就能得出答案。

#### 思路二代码
```
    int arrayPairSumBucketSort(vector<int>& nums) {
      vector<int> bucket(10000 * 2 + 1, 0);
      int lower = 10000 * 2;
      for (int i = 0; i < nums.size(); ++i) {
        bucket[nums[i] + 10000] += 1;
        if (nums[i] + 10000 < lower) {
          lower = nums[i] + 10000;
        }
      }

      int count = 0;
      int sum = 0;
      bool skip = false;
      while (count < nums.size() / 2) {
        if (bucket[lower] > 0 && !skip) {
          sum += lower - 10000;
          skip = true;
          ++count;
          bucket[lower]--;
        } else if (bucket[lower] > 0 && skip) {
          skip = false;
          bucket[lower]--;
        } else {
          ++lower;
        }
      }
      return sum;
    }
```
