---
title: C++-sort-浅识
date: 2022-05-03 13:44:28
tags: 
typora-root-url: ./..
---
  
今天在做题时候发现自己对于C++自定义排序方法`sort` 和 `stable_sort` 的理解一直有问题。
  
<!--more-->
  
先说最粗暴简单的理解: **`sort` 和 `stable_sort` 定义了我们希望的"<"比较，就是把我们认为小的放在前面**。
  
# 题目
  
<img src="/imgs/C++-sort-浅识/题目-1677565078882-1.png"/>
  
代码: 
  
```cpp  
vector<string> reorderLogFiles(vector<string>& logs) {  
    stable_sort(logs.begin(), logs.end(), [&](const string& log1, const string& log2) {  
        int pos1 = log1.find_first_of(' ');  
        int pos2 = log2.find_first_of(' ');  
        bool isDigit1 = isdigit(log1[pos1 + 1]);  
        bool isDigit2 = isdigit(log2[pos2 + 1]);  
        if (isDigit1 && isDigit2)  
        {  
            return false;        // 我的问题  
        }  
        if (!isDigit1 && !isDigit2)  
        {  
            string str1 = log1.substr(pos1);  
            string str2 = log2.substr(pos2);  
            if (str1 != str2)  
                return str1 < str2;  
            return log1 < log2;  
        }  
        return isDigit1 ? false : true;  
        });  
    return logs;  
}  
```
  
我的问题就是出现在数字log的处理部分:
  
先说说一开始的**错误理解**: `s1`和 `s2` 作为两个比较对象传入，`s1` 在 `s2` 前面，如果返回值为 `true`，不做修改；如果返回值为 `false`，二者交换位置。 
  
嘿，这看上去完全就是很合理的嘛，因为依照我们的经验来看就是这样，**然而**实际却相去甚远。
  
## 我的反思
  
<img src="/imgs/C++-sort-浅识/sort.png"/>
  
这是在说什么呢？就是说我们自定义的 `sort` 其实实现的是 `<` 比较。  
关于返回值的理解就是，如果返回值为 `true` 就是 `s1 < s2`，那么返回值为 `false`就很好解释了，利用一点离散知识, `!<` 就是 `>=` 。
  
> 如果对于离散有疑问，那换一个简单粗暴的说法：  
> true: 就是 s1 < s2  
>   
> 那么， s1 = s2 和 s1 > s2 就都是 false
  
# 总结
  
我需要纠正的是 `sort` 函数注重的是 `cmp` 而不是 `swap`，就是只是实现比较谁更“小”，而不是他们的位置关系。
  
相关链接:  
[Why must std::sort compare function return false when arguments are equal?](https://stackoverflow.com/questions/45929474/why-must-stdsort-compare-function-return-false-when-arguments-are-equal)  