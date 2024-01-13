---
title: ZumaGame
typora-root-url: ./..
date: 2023-03-03 15:47:23
tags:
---


# ZumaGame

经典的困难题
BFS或者记忆化搜索

<!--more-->

## 题目

<img src="/imgs/ZumaGame/question.png">

## 例子

* **输入**：board = "WRRBBW", hand = "RB"

* **输出**：-1
* **解释**：无法消除所有球，最好解是
  -插入‘R’，WRRRBBW->WBBW
  -插入‘B’，WBBBW->WW
  没有球，结束。
