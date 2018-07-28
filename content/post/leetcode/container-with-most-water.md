---
date: 2018-05-27
title: container-with-most-water
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.
```

#### 思路一
一开始我理解错了，以为是求相邻两条线段之间的面积最大，很快写了个方法提交上去，结果是Wrong Answer. 一看，原来是要任意两条线段之间的面积。

直观的想法，就是暴力，遍历所有的两两线段，计算出面积，取面积最大的，提交上去，结果是Time Exceed Limit. 下面是暴力的代码:
```
class Solution {
  public:
    int maxArea(vector<int>& height) {
      int max = 0;
      for (int i = 0; i < height.size() - 1; ++i) {
        for (int j = i + 1; j < height.size(); ++j) {
          int min = this->min(height[i], height[j]);
          // std::cout << "min:" << min << " " << min * (j - i) << " " << max << std::endl;
          if (min * (j - i) > max) {
            max = min * (j - i);
          }
        }
      }
      return max;
    }

    int min(int left, int right) {
      return left < right ? left : right;
    }
};

```

看来不能暴力，暴力的时间复杂度是O(n^2)。难道可以动态规划？但是想了一下，又没有重复子问题，看来这个方向不对。

#### 思路二
看来这个问题，在暴力的方法中，肯定存在一些步骤是没有必要的，只能通过观察。我尝试画了一下，举了几个例子，发现:

- 每次取两条线段，面积的大小取决于最短的那一条，这叫做短板效应。

想起来查理芒格那句话: 反过来想，总是反过来想。所以初始的时候，取最左边和最右边的两条线段，是不是当前最大的面积呢？之前我思考的时候总是取最前面的几条。

最重要的发现是:

- 如果取两条线段，计算了面积之后，我们移动较长的那条线段，不管怎么移动，后面算出来的面积都不会比初始的面积大
因为面积的大小等于宽度乘以高度，一开始的宽度最大，由于面积的大小取决于最短的那一条, 所以后面取的线段，最短的高度一定是小于等于一开始的线段的高度，
可是宽度又变小了，那么面积肯定变小。

如果两条线段长度相等, 验证了一下，发现也符合这个规律。

那这样就好办了，每次移动线段时，找到最短的线段，往中间移动，因为没必要再移动最长的线段了，这样可以大大减少不必要的计算，
理论上O(n)的时间复杂度下就可以解决这个问题。

下面是这种思路的代码:
```
class Solution {
  public:
    int maxArea(vector<int>& height) {
      int max = 0;
      int i = 0;
      int j = height.size() - 1;
      while (i < j) {
        int min = this->min(height[i], height[j]);
        // std::cout << "min:" << min << " " << min * (j - i) << " " << max << std::endl;
        if (min * (j - i) > max) {
          max = min * (j - i);
        }
        if (min == height[i]) {
          ++i;
        } else if (min == height[j]) {
          --j;
        }
      }
      return max;
    }

    int min(int left, int right) {
      return left < right ? left : right;
    }
};

```

提交上去，居然打败了96%的人!

一大早起来刷道题，还是很有成就感的，不说废话了，收拾东西上班去。
