---
title: UESTC 2428
tags:
  - 动态规划
  - 状压DP
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 状压DP
abbrlink: 56985
date: 2020-05-23 10:25:15
---


## 题目

给一段长度为$n$的序列，包含$1\sim m$的数字，可以任意交换两个数字的位置，目标是让所有相同的数字相邻，问最多可以有几个数字的位置不被交换。

## 题解

参照出题人的PPT。

把“已经排列好的数字种类的集合”视为状态，不考虑集合中数字的顺序，他们的总数是固定的，记状态为$s$，已经排列的人数为$f[s]$，这个可以在输入的时候直接求出来。$m$最大只有20，直接位运算来实现集合。$dp[s]$表示状态$s$最多能固定的数字数量。

枚举$s$，在每次枚举的时候枚举数字$i$，考虑

- $i$不在$s$中，则$dp[s|(1<<i)]=\max\{dp[s]+sum[r][i]-sum[l-1][i]|i\in[0,m-1]\}$
- $i$在$s$中，则$dp[s]=\max\{dp[s\oplus(1<<i)]+sum[r][i]-sum[l-1][i]|i\in[0,m-1]\}$

注意我为了方便位运算将输入序列全部减去1。

最后输出$dp[(1<<m)-1]$即可。

## 代码

因为一发入魂，所以没有写测试例生成和暴力验证。

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

const int MAX = 1e5 + 7;

unsigned int s;

int n, m;
int dp[1 << 21];
int f[1 << 21];
int a[MAX + 1];
int sum[MAX + 1][20 + 1];

int getSum(int c) {
    int k = 0;
    int ans = 0;
    for (; c; c >>= 1, k++) {
        ans += c & 1 ? sum[n][k] : 0;
    }
    return ans;
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        // 从0开始
        a[i] -= 1;
        for (int j = 0; j < m; j++) {
            sum[i][j] = sum[i - 1][j] + (j == a[i]);
        }
    }
    const int full = (1 << m) - 1;
    for (int i = 0; i <= full; i++) {
        f[i] = getSum(i);
    }
    while (s <= full) {
        for (int i = 0; i < m; i++) {
            int l = f[s] + 1;
            int r = f[s | (1 << i)];
            if (s & (1 << i)) {
                dp[s] = max(dp[s], dp[s - (1 << i)] + sum[r][i] - sum[l - 1][i]);
            } else {
                dp[s | (1 << i)] = max(dp[s | (1 << i)], dp[s] + sum[r][i] - sum[l - 1][i]);
            }
        }
        s++;
    }
    cout << dp[full] << endl;
    return 0;
}

```