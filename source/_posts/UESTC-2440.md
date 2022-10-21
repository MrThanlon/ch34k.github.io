---
title: UESTC 2440
tags:
  - 背包DP
  - 动态规划
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 背包DP
abbrlink: 47259
date: 2020-05-23 10:25:30
---


## 题目

和[UESTC-2430](https://blog.ch34k.xyz/2020/05/23/UESTC-2430/)完全一致，数据量增大。

## 题解

我觉得没写啥的必要了，和[UESTC-2430](https://blog.ch34k.xyz/2020/05/23/UESTC-2430/)完全一致，加上二进制优化就行。

## 代码

### AC代码

这是debug之后了的，注意到打开了`OPTIMIZE`宏。

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
#define OPTIMIZE 1

using namespace std;

class Thing {
public:
    long long weight, value, number;
};

class Special {
public:
    long long x, y, z;

    long long getValue(long long weight) {
        if (weight == 0) {
            return max(0ll, z);
        }
        return x * weight * weight + y * weight + z;
    }
};

Thing things[10007 * 15];
int cnt = 0;
Special specials[6];

long long dp0[10007];
long long dp[10007];

const int pow2[] = {1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384};

# if OPTIMIZE

// 二进制优化
int optimize() {
    int st = cnt;
    int d = 0;
    while (things[cnt].number > pow2[d]) {
        things[cnt].number -= pow2[d];
        // things[d + 1].number = 1;
        things[cnt + d + 1].weight = things[cnt].weight * pow2[d];
        things[cnt + d + 1].value = things[cnt].value * pow2[d];
        d += 1;
    }
    things[cnt].weight = things[cnt].weight * things[cnt].number;
    things[cnt].value = things[cnt].value * things[cnt].number;
    // things[cnt].number = 1;
    cnt += d + 1;
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
        optimize();
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
        memcpy(dp0, dp, sizeof(long long) * (c + 3));
#endif
    }
    for (int i = 0; i < cnt; i++) {
        for (int j = 1; j <= c; j++) {
            if (j < things[i].weight) {
                continue;
            }
            dp[j] = max(dp0[j], dp0[j - things[i].weight] + things[i].value);
        }
        memcpy(dp0, dp, sizeof(long long) * (c + 3));
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
        memcpy(dp0, dp, sizeof(long long) * (c + 3));
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

测试例生成（Python）

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