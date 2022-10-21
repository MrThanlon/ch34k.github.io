---
title: UESTC 2427
tags:
  - 区间DP
  - 动态规划
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 区间DP
abbrlink: 56025
date: 2020-05-23 10:25:12
---


## 题目

给一组长度为$n$序列，相邻的数字可以合并为一个数字，合并后的数字等于合并前的数字+1，问合并后的数字最少是多少个。

## 题解

参照出题人PPT，是区间DP没错了。

$dp[l][r]$表示区间$[l,r]$在合并过之后有多少个数，$ag[l][r]$表示区间$[l,r]$合并成一个（如果可以）的数。

不好解释，直接看代码吧。。。

```C++
for (int mid = l; mid <= r; mid++) {
  if (dp[l][mid] == 1 && dp[mid + 1][r] == 1 &&
      ag[l][mid] == ag[mid + 1][r] && ag[l][mid] != 0) {
    // merge
    dp[l][r] = 1;
    ag[l][r] = ag[l][mid] + 1;
  }
  dp[l][r] = min(dp[l][r], dp[l][mid] + dp[mid + 1][r]);
}
```

对于区间$[l,r]$，将其分为$[l,mid]$和$[mid+1,r]$，看能否合并，然后最小值。

然后枚举长度和$l$即可，复杂度$O(n^3)$。

## 代码

因为一发入魂，就没写测试例生成和暴力程序了。

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

const int MAX = 500 + 7;

int arr[MAX];
int ag[MAX][MAX];
int dp[MAX][MAX];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = n;
        }
    }
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
        dp[i][i] = 1;
        ag[i][i] = arr[i];
    }
    for (int len = 1; len <= n; len++) {
        for (int l = 1; l + len <= n; l++) {
            int r = l + len;
            for (int mid = l; mid <= r; mid++) {
                if (dp[l][mid] == 1 && dp[mid + 1][r] == 1 &&
                    ag[l][mid] == ag[mid + 1][r] && ag[l][mid] != 0) {
                    // merge
                    dp[l][r] = 1;
                    ag[l][r] = ag[l][mid] + 1;
                }
                dp[l][r] = min(dp[l][r], dp[l][mid] + dp[mid + 1][r]);
            }
        }
    }
    cout << dp[1][n] << endl;
    return 0;
}
```
