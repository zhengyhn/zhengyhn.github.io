---
date: 2018-01-21
title: N皇后问题
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other.

Given an integer n, return all distinct solutions to the n-queens puzzle.

Each solution contains a distinct board configuration of the n-queens' placement, where 'Q' and '.' both indicate a queen and an empty space respectively.

For example,
There exist two distinct solutions to the 4-queens puzzle:

[
 [".Q..",  // Solution 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // Solution 2
  "Q...",
  "...Q",
  ".Q.."]
]
```

#### 我的思路
本来这是很经典的问题，有很多人分享解决办法，不应该再发出来。但是，这次是我想了很久才想出来的，写下来纪念一下。

转化成图，n * n的图，第一行有n个位置可以摆放，一共有n * n种状态，找出互相无法攻击的那些状态(上下左右对角线没有皇后)。

直接使用深度优先搜索，遍历一棵n层的树，除了节节点，每个节点都有n个子节点，遇到无法遍历下去的，直接剪枝。递归可以解决，这里使用栈来实现非递归。

#### 代码
```
class Solution {
  public:
    vector<vector<string>> solveNQueens(int n) {
      vector<vector<string>> ret;
      if (n <= 0) {
        return ret;
      }
      int stack[n];
      int top = -1;
      stack[++top] = -1;

      while (top >= 0) {
        ++stack[top];
        if (stack[top] >= n) {
          --top;
          continue;
        }
        // this->print(stack, top + 1, n);
        bool isValid = this->isValidBoard(stack, top + 1, n);
        // std::cout << isValid << std::endl;
        if (isValid) {
          if (top == n - 1) {
            ret.push_back(this->toString(stack, top + 1, n));
          } else {
            stack[++top] = -1;
          }
        }
      }
      return ret;
    }
  private:
    bool isValidBoard(int *stack, int stackLen, int boardLen) {
      if (stackLen == 1) {
        return true;
      }
      int it = stack[stackLen - 1];
      if (it >= boardLen) {
        return false;
      }
      for (int i = 0; i < stackLen - 1; ++i) {
        if (it == stack[i]) {
          return false;
        }
        int left = it - stackLen + 1 + i;
        if (left >= 0 && left == stack[i]) {
          return false;
        }
        int right = stackLen - i - 1 + it;
        if (right < boardLen && right == stack[i]) {
          return false;
        }
      }
      return true;
    }

    vector<string> toString(int *stack, int stackLen, int n) {
      vector<string> ret(n);

      for (int i = 0; i < n; ++i) {
        string item(n, '.');
        if (i < stackLen) {
          item[stack[i]] = 'Q';
        }
        ret[i] = item;
      }
      return ret;
    }

    void print(int *stack, int stackLen, int n) {
      vector<string> ret = this->toString(stack, stackLen, n);
      for (auto row: ret) {
        for (auto item: row) {
          std::cout << item;
        }
        std::cout << std::endl;
      }
      std::cout << "--------" << std::endl;
    }
};

```
