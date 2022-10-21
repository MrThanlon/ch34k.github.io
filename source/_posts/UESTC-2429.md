---
title: UESTC 2429
tags:
  - 动态规划
  - 状压DP
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 状压DP
abbrlink: 7768
date: 2020-05-23 10:25:20
---


## 题目

和[UESTC-2428](https://blog.ch34k.xyz/2020/05/23/UESTC-2428/)基本相同，区别在于只能相邻的两个进行交换，询问最少交换次数。

## 题解

有一说一，这题我没怎么搞懂，基本上全靠出题人的PPT。

这题比较直白了，直接就是$1\sim 20$的数字。继续使用状态压缩，变量$s$二进制表示状态，$dp[s]$表示状态$s$需要的最少交换次数，$w[i][j]$表示原排列i在j前面的对数，状态转移方程
$$
dp[s]=\min\{dp[s-(1<<i)]+\sum_{j\in s \&j\ne i}w[i][j]\}
$$
显然$dp[0]=0$。

$w$直接暴力就行，题解上给的做法也是暴力。（学姐真是太良心了）

## 代码

虽然没有一发入魂（某个循环的边界写错了），但是调了几下就过了，所以也没有写测试例生成和暴力验证......

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

unsigned int s = 1;

int a[MAX];
int cnt[27];
int w[27][27];

const unsigned int full = (1u << 20u) - 1;
long long dp[full + 7];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i] -= 1;
    }
    for (int i = 1; i <= n; i++) {
        cnt[a[i]]++;
        for (int j = 0; j < 20; j++) {
            w[j][a[i]] += cnt[j];
        }
    }
    for (; s <= full; s++) {
        dp[s] = 0x7fffffffffffffffll;
        for (int i = 0; i < 20; i++) {
            if (!(s & (1 << i))) {
                continue;
            }
            long long ans = 0;
            for (int j = 0; j < 20; j++) {
                if (i == j || !(s & (1 << j))) {
                    continue;
                }
                ans += w[i][j];
            }
            dp[s] = min(dp[s], dp[s - (1 << i)] + ans);
        }
    }
    cout << dp[full] << endl;
    return 0;
}

```