---
typora-root-url: ./..
---

一个字符串删除n位，使结果最小，相对位置不变，不能有前导零。
例子：90001删除2位，结果900
Removes n bits of a number so that it is minimal
[类似的题目](https://www.geeksforgeeks.org/build-lowest-number-by-removing-n-digits-from-a-given-number/)

## [戳气球(数组、动态规划)](https://blog.51cto.com/zhanjq/6054073)

有 n 个气球，编号为0 到 n - 1，每个气球上都标有一个数字，这些数字存在数组 nums 中。
现在要求你戳破所有的气球。戳破第 i 个气球，你可以获得 nums[i - 1] * nums[i] * nums[i + 1] 枚硬币。 这里的 i - 1 和 i + 1 代表和 i 相邻的两个气球的序号。如果 i - 1或 i + 1 超出了数组的边界，那么就当它是一个数字为 1 的气球。
求所能获得硬币的最大数量。

示例 1：
输入：nums = [3,1,5,8] 输出：167 解释： nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> [] `coins = 3*1*5 + 3*5*8 + 1*3*8 + 1*8*1 = 15 + 120 + 24 + 8 = 167`

### [Leetcode题解](https://leetcode.cn/problems/burst-balloons/)

`dp[i][j]表示填满开区间(i,j)能得到的最大数`

```cpp
#include <cstdio>
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    int n;
    cin >> n;
    vector<int> nums(n);
    for(int i = 0; i < n; ++i)
    {
        cin >> nums[i];
    }
    vector<int> tmp(n+2);
    for(int i = 0; i < n; ++i)
    {
        tmp[i+1] = nums[i];
    }
    tmp[0] = tmp[n+1] = 1;
    vector<vector<int>> dp(n+2,vector<int>(n+2));
    for(int k = 1; k <= n; ++k)
    {
        for(int i = 1; i <= n - k + 1; ++i)
        {
            int j = i+k-1;
            for(int x = i; x <= j; ++x)
            {
                dp[i][j] = max(dp[i][j], 
                dp[i][x-1] + tmp[i-1]*tmp[x]*tmp[j+1]+dp[x+1][j]);
            }
        }
    }
    cout << dp[1][n];
    return 0;
}
```

# 状态压缩DP

# [祖玛游戏](https://leetcode.cn/problems/zuma-game/solutions/)



# [省份数量](https://leetcode.cn/problems/number-of-provinces/description/)



# [N皇后]()

规则和N皇后一样，但是输入一个棋盘上有若干旗子，问最多还能放多少棋子
