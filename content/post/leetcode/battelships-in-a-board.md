---
date: 2017-12-23
title: battelships-in-a-board
tags: ["leetcode", "算法"]
categories: ["leetcode"]
---

#### 题目
```
Given an 2D board, count how many battleships are in it. The battleships are represented with 'X's, empty slots are represented with '.'s. You may assume the following rules:
You receive a valid board, made of only battleships or empty slots.
Battleships can only be placed horizontally or vertically. In other words, they can only be made of the shape 1xN (1 row, N columns) or Nx1 (N rows, 1 column), where N can be of any size.
At least one horizontal or vertical cell separates between two battleships - there are no adjacent battleships.
Example:
X..X
...X
...X
In the above board there are 2 battleships.
Invalid Example:
...X
XXXX
...X
This is an invalid board that you will not receive - as battleships will always have a cell separating between them.
Follow up:
Could you do it in one-pass, using only O(1) extra memory and without modifying the value of the board?
```

#### 思路
一个二维的板上有很多个X，其中部分X属于一组，求的是一共有多少组。这是并查集的思想，可以遍历整个二维数组，初始每个X自身为一个组，当两个组可以合并时，组数就减一，最终的组数就是答案。根据题意，行相连或者列相连的为同一组，这可以用来判断两个并查集是否可以合并。

#### 代码
```
#include <iostream>
#include <vector>

using namespace std;

class Solution {
public:
    int adjacentCount(vector<vector<char>>& board, int i, int j) {
      int count = 0;
      if ((i - 1) >= 0 && board[i - 1][j] == 'X') {
        ++count;
      }
      if ((j - 1) >= 0 && board[i][j - 1] == 'X') {
        ++count;
      }
      return count;
    }

    int countBattleships(vector<vector<char>>& board) {
      int count = 0;
      for (int i = 0; i < board.size(); ++i) {
        for (int j = 0; j < board[i].size(); ++j) {
          if (board[i][j] == 'X') {
            ++count;
            if (this->adjacentCount(board, i, j) > 0) {
              --count;
            }
          }
        }
      }
      return count;
    }
};

int main() {
  Solution sln;
  vector<vector<char>> board = {{'X', '.', 'X'}};
  // vector<vector<char>> board = {{'X', '.', '.', 'X'}, {'.', '.', '.', 'X'}, {'.', '.', '.', 'X'}};
  // vector<vector<char>> board = {{'X', 'X', 'X', 'X'}, {'.', '.', '.', '.'}, {'X', '.', 'X', '.', 'X'}, {'X', '.', 'X', '.', 'X'}};

  std::cout << sln.countBattleships(board) << std::endl;

  return 0;
}
```
