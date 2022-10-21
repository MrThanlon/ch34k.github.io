---
title: UESTC 2431
tags:
  - LCS
  - 动态规划
categories:
  - ICPC训练
  - 动态规划
  - LCS
abbrlink: 18520
date: 2020-05-23 10:25:27
---


## 题目

给任意一个字符串，求最少插入多少字符可以得到回文串。

## 题解

参照[leetcode 1312](https://leetcode-cn.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)。

求该字符串与它的反序的最长公共子序列（LCS）长度，然后用字符串长度减去这个长度，输出答案完事，时间复杂度为字符串长度的平方。签到题。

## 代码

因为一发入魂，就没写测试例生成和暴力验证。

### AC代码

```C++
// @author: hzy
//#pragma G++ optimize("O3")

#include <algorithm>
#include <cmath>
#include <cstdio>
#include <ctime>
#include <cstring>
#include <iostream>
#include <queue>
#include <map>
#include <set>
#include <string>
#include <list>
#include <forward_list>
#include <stack>
#include <unordered_set>
#include <vector>
#include <limits.h>

#define DISPLAY_A 0

using namespace std;

const int MAX = 5e3 + 7;

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    cin >> n;
    string s;
    cin >> s;
    string inv = s;
    reverse(inv.begin(), inv.end());
    // LCS
    vector<vector<int>> dp(n + 1, vector<int>(n + 1));
    dp[0][0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            if (s[i - 1] == inv[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = max(dp[i][j - 1], dp[i - 1][j]);
            }
        }
    }
    cout << n - dp[n][n] << endl;
    return 0;
}
```