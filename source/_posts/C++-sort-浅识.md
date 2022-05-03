---
title: C++-sort-浅识
date: 2022-05-03 13:44:28
tags: 
---

今天在做题时候发现自己对于C++自定义排序方法`sort` 和 `stable_sort` 的理解一直有问题。

<!--more-->

先说最粗暴简单的理解: **`sort` 和 `stable_sort` 定义了我们希望的"<"比较，就是把我们认为小的放在前面**。

相关链接:
[Why must std::sort compare function return false when arguments are equal?](https://stackoverflow.com/questions/45929474/why-must-stdsort-compare-function-return-false-when-arguments-are-equal)
