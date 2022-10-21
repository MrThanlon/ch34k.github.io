---
title: UESTC 2430
tags:
  - 背包DP
  - 动态规划
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 背包DP
abbrlink: 34969
date: 2020-05-23 10:25:24
---


## 题目

有$n$种普通物品，第$i$种物品的体积为$V_i$，价值$W_i$，数量$D_i$，有$m$件神奇物品，神奇物品的价值由分配的体积$V_i$决定，计算公式为
$$
W_i=x_iV_i^2+y_iV_i+z_i
$$
给一个体积为$C$的背包，求最大价值。

## 题解

> 多重背包+泛化物品。

先说多重背包的部分。

其实背包问题没啥好讲的，考虑0-1背包，$dp[i][j]$表示前$i$个物品在容量为$j$的背包下的最大价值，有状态转移方程
$$
dp[i][j]=\max(dp[i-1][j],dp[i-1][j-V_i]+W_i)
$$
这题数据量比较小，直接朴素做法就行，把多重背包考虑为多个相同重量和价值的物品，就可以转变为0-1背包，时间复杂度$O(nCD)$（讲道理这已经达到1e9了，居然没超时）。

再来说一下泛化物品。

参照背包九讲中关于泛化物品的部分。计算完多重背包的部分之后，我们得到了一个在没有神奇物品情况下，任意容量背包能装的最大价值。背包九讲中对于多个泛化物品的操作是两两合并，这样就只有一个泛化物品了，但是我觉得太麻烦，其实泛化物品就相当于有$C$个一般物品，并且将它们放到一个组（也就是只能拿一个，因为泛化物品只有一个），这样就转变为分组背包，遍历容量然后求最大值就行，同样考虑$dp[i][j]$为前$i$个神奇物品在容量为$j$的背包下的最大价值，转移方程为
$$
dp[i][j]=\max\{dp[i-1][j-v]+x_iv^2+y_iv^2+z_i|v\in[1,j]\}
$$
注意到$j$是两重循环，所以是这部分复杂度是$O(mC^2)$。

## 代码

<del>暴力测试的程序代码找不到了，嘤嘤嘤。</del>

### AC代码

`OPTIMIZE`是二进制优化，在写这题的时候没有并没有完成这一部分，朴素暴过去的。

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
#define OPTIMIZE 0

using namespace std;

class Thing {
public:
    int weight, value, number;

};

class Special {
public:
    int x, y, z;

    int getValue(int weight) {
        if (weight == 0) {
            return max(0, z);
        }
        return x * weight * weight + y * weight + z;
    }
};

Thing things[1007 * 15];
int cnt = 0;
Special specials[6];

int dp0[1007];
int dp[1007];

const int pow2[] = {1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024};

# if OPTIMIZE
// 二进制优化
int optimize() {
    int st = cnt;
    int d = 0;
    while (things[cnt].number > pow2[d]) {
        things[cnt].number -= pow2[d];
        things[d + 1].number = 1;
        things[d + 1].weight = things[cnt].weight * pow2[d];
        things[d + 1].value = things[cnt].value * pow2[d];
        d += 1;
    }
    things[cnt].weight = things[cnt].weight * things[cnt].number;
    things[cnt].value = things[cnt].value * things[cnt].number;
    things[cnt].number = 1;
    // 考虑不拿的情况
    cnt += d + 2;
    return st;
}
#endif

#if DISPLAY_A
int sp[1007];
#endif

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m, c;
    cin >> n >> m >> c;
    for (int i = 1; i <= n; i++) {
        cin >> things[cnt].weight >> things[cnt].value >> things[cnt].number;
        // 多重背包
#if OPTIMIZE
        // 二进制优化
        auto st = optimize();
        for (int k = st; k < cnt; k++) {
            for (int j = 1; j <= c; j++) {
                if (j < things[k].weight) {
                    continue;
                }
                dp[j] = max(dp[j], dp0[j - things[k].weight] + things[k].value);
            }
            memcpy(dp0, dp, sizeof(int) * c + 3);
        }
#else
        for (int j = 1; j <= c; j++) {
            // 不拿的情况(k=0)已经包含了
            for (int k = 1; k <= things[cnt].number; k++) {
                int weight = k * things[cnt].weight;
                if (j < weight) {
                    break;
                }
                dp[j] = max(dp[j], dp0[j - weight] + things[cnt].value * k);
            }
        }
        memcpy(dp0, dp, sizeof(int) * (c + 3));
#endif
    }
    for (int i = 1; i <= m; i++) {
        cin >> specials[i].x >> specials[i].y >> specials[i].z;
        // 计算神奇物品
        for (int j = 0; j <= c; j++) {
            for (int k = 0; k <= j; k++) {
#if DISPLAY_A
                if (dp[j] < dp0[j - k] + specials[i].getValue(k)) {
                    dp[j] = dp0[j - k] + specials[i].getValue(k);
                    sp[i] = k;
                }
#else
                dp[j] = max(dp[j], dp0[j - k] + specials[i].getValue(k));
#endif
            }
        }
        memcpy(dp0, dp, sizeof(int) * (c + 3));
    }
#if DISPLAY_A
    for (int i = 1; i <= m; i++) {
        cout << sp[i] << " ";
    }
    cout << endl;
#endif
    cout << dp[c] << endl;
    return 0;
}
```

### 测试例生成（Python）

```Python
def G(n, m, c, max=1e3, filename='/Volumes/RD/in'):
    n = int(n)
    m = int(m)
    c = int(c)
    f = open(filename, 'w')
    f.write("{} {} {}\r\n{}\r\n{}".format(
        n, m, c, "\r\n".join([
            "{} {} {}".format(random.randint(1, max), random.randint(1, max), random.randint(1, max))
            for i in range(n)
        ]),
        "\r\n".join([
            "{} {} {}".format(random.randint(-max, max), random.randint(-max, max), random.randint(-max, max))
            for i in range(m)
        ])
    ))
    f.close()
```