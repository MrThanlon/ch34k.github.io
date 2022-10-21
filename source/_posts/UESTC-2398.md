---
title: UESTC 2398
tags:
  - 动态规划
  - 斜率优化
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 斜率优化
abbrlink: 12079
date: 2020-05-23 10:24:55
---


## 题目

给定$m$个牛头人和$p$个老哥，老哥向一个方向移动，遇到牛头人时牛头人停止活动，现在给出牛头人出现的时间距离，让所有牛头人加起来的活动时间最短，安排老哥的出发时间。

## 题解

参照出题人PPT的状态转移方程。

首先简化问题，假设所有牛头人都出现在起始位置，直接用时间减去距离即可，为了方便后面的操作再排序一下，设为$a[]$，$dp[i][j]$表示第$i$个老哥在$a[j]$时刻出发前$j$牛头人的最少活动时间，于是有状态转移方程
$$
dp[i][j]=\min\{dp[i-1][k]+\sum_{x=k+1}^{j}(a[j]-a[x])|k\in[1,j]\}
$$
考虑用前缀和，令
$$
s[j]=\sum_{x=1}^ja[x]
$$
方程可以变为
$$
dp[i][j]=\min\{(dp[i-1][k]+s[k])-k\cdot a[j]+(j\cdot a[j]-s[j])|k\in[1,j]\}
$$
考虑$i\in[1,p]$，$j\in[1,m]$，直接DP的话复杂度依然高达$O(pm^2)$，因为式子中有一项同时包含$k$和$j$，所以无法用单调队列优化，我们考虑斜率优化。

设$k_1<k_2$且$k_2$比$k_1$更优，即
$$
(dp[i-1][k_2]+s[k_2])-k_2\cdot a[j]+(j\cdot a[j]-s[j])<
(dp[i-1][k_1]+s[k_1])-k_1\cdot a[j]+(j\cdot a[j]-s[j])
$$

$$
(dp[i-1][k_2]+s[k_2])-k_2\cdot a[j]<
(dp[i-1][k_1]+s[k_1])-k_1\cdot a[j]
$$

$$
\frac{(dp[i-1][k_2]+s[k_2])-(dp[i-1][k_1]+s[k_1])}{k_2-k_1}<a[j]
$$

<del>（虽然我不知道变换出这个式子之后有什么用，总之就放这吧）</del>

其实对于斜率优化，我的理解是对于不同的自变量（这里是$k$），取到的值不同，而当$k_1<k_2$且从$k_1$到$k_2$的斜率小于这里的$a[j]$（注意$a[j]$单调不减）时，则后续的状态转移一定不会有$k_1$，因为$k_2$一定比$k_1$更优，同时能取到的点集会有斜率单调不减的特征，也就是一个下凸包，同时那个最优的点就是凸包上第一个斜率大于等于$a[j]$的点，由于$a[j]$单调不减，所以在取了这点之后可以直接将这个点以前的点全部移除。

之后就是代码实现了，我用一个数组来维护下凸包，准确说是一个双向队列（两个变量`head`和`tail`，但是STL的`deque`效率不得行，所以是手写的。

## 代码

### AC代码

这题`int`会溢出，我吐了。

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

long long a[MAX];
long long sum[MAX];

long long dp[100 + 7][MAX];

float grad(const long long *arr, long long x1, long long x2) {
    return ((float) (arr[x2] + sum[x2] - arr[x1] - sum[x1])) / ((float) (x2 - x1));
}

int ndeq[MAX];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int m, p;
    cin >> m >> p;
    int tmp;
    for (int i = 1; i <= m; i++) {
        cin >> tmp >> a[i];
        a[i] -= tmp;
    }
    sort(a + 1, a + m + 1);
    // 前缀和
    for (int i = 1; i <= m; i++) {
        sum[i] = sum[i - 1] + a[i];
    }
    for (int j = 1; j <= m; j++) {
        dp[1][j] = a[j] * j - sum[j];
    }
    for (int i = 2; i <= p; i++) {
        // 斜率优化
        int head = 0, tail = 0;
        // 必然吧
        dp[i][1] = 0;
        ndeq[tail] = 0;
        tail++;
        for (int j = 2; j <= m; j++) {
            // 准备插入j，维护斜率递增，左侧斜率小于右侧斜率
            while (tail - head >= 2 &&
                   grad(dp[i - 1], ndeq[tail - 2], ndeq[tail - 1]) >=
                   grad(dp[i - 1], ndeq[tail - 1], j)) {
                // 出栈
                tail--;
            }
            // 这一点应该是一定会加入的
            ndeq[tail] = j;
            tail++;
            // 找不到情况队处理
            dp[i][j] = dp[i - 1][j];
            // TODO: 二分查找
            // 大概直接查找就完事了
            for (int k = head; k < tail - 1; k++) {
                // 斜率递增，故左斜率<=a[j]&&右斜率>=a[j]时最优
                if (dp[i - 1][ndeq[k + 1]] + sum[ndeq[k + 1]] - dp[i - 1][ndeq[k]] - sum[ndeq[k]] >=
                    a[j] * (ndeq[k + 1] - ndeq[k]) &&
                    dp[i - 1][ndeq[k]] + a[j] * (j - ndeq[k]) - (sum[j] - sum[ndeq[k]]) <= dp[i][j]) {
                    // 是这点了
                    dp[i][j] = dp[i - 1][ndeq[k]] + a[j] * (j - ndeq[k]) - (sum[j] - sum[ndeq[k]]);
                    // 修改队首
                    head = k;
                    break;
                }
            }
            head = min(head, tail - 1);
        }
    }
    cout << dp[p][m] << endl;
    return 0;
}

```

### 测试例生成（Python）

为了方便直接将位置都设为0了。

```Python
def e(m, p, max=1e9, filename='in'):
    m = int(m)
    p = int(p)
    fi = open(filename, 'w')
    fi.write("{} {}\r\n{}\r\n".format(
        m, p, "\r\n".join([
            "0 {}".format(random.randint(0, max))
            for i in range(m)
        ])
    ))
    fi.close()
```

### 暴力验证

只是没有斜率优化，其他一样。

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

long long a[MAX];
long long sum[MAX];

long long dp[100 + 7][MAX];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int m, p;
    cin >> m >> p;
    int tmp;
    for (int i = 1; i <= m; i++) {
        cin >> tmp >> a[i];
        a[i] -= tmp;
    }
    sort(a + 1, a + m + 1);
    // 前缀和
    for (int i = 1; i <= m; i++) {
        sum[i] = sum[i - 1] + a[i];
    }
    for (int j = 1; j <= m; j++) {
        dp[1][j] = a[j] * j - sum[j];
    }
    for (int i = 2; i <= p; i++) {
        for (int j = 1; j <= m; j++) {
            dp[i][j] = dp[i - 1][1] + a[j] * (j - 1) - (sum[j] - sum[1]);
            for (int k = 1; k <= j; k++) {
                dp[i][j] = min(dp[i][j], dp[i - 1][k] + a[j] * (j - k) - (sum[j] - sum[k]));
            }
        }
    }
    cout << dp[p][m] << endl;
    return 0;
}

```
