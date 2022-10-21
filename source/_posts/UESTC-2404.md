---
title: UESTC 2404
tags:
  - 动态规划
  - 斜率优化
  - 刷题
categories:
  - ICPC训练
  - 动态规划
  - 树形DP
abbrlink: 48024
date: 2020-05-23 10:25:03
---


## 题目

给一颗$n$节点的树，1节点为根节点，求最少减去多少条边可以得到一颗$p$节点的树。

## 题解

参照[洛谷P1272](https://www.luogu.com.cn/problem/P1272)。

基本上是原题了。

$dp[i][j]$表示以$i$为根节点时获得节点数为$p$的树最少需要的修建次数。状态转移方程
$$
dp[i][j]=\min\{dp[i][j-k]+dp[son][k]-1|k\in[1,j]\}
$$
$son$是$i$的子节点。

考虑到不一定1为根节点（$dp[1][p]$）才是最优，所以我们可以遍历每一个节点作为根节点的情况。注意到除了根节点之外，其他节点为根节点的话需要断开自己与父节点的边，所以遍历的时候+1即可。

还有一个坑是输入。每行输入的是$u$和$v$表示$u$和$v$之间有一条边，然而并不知道哪个是父节点，所以得专门做一次处理，我懒得搞骚操作直接暴力了，速度还行吧。

## 代码

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

const int MAX = 200 + 7;

set<int> nodes[201];

int dp[201][201];

int dfs(int root) {
    int sum = 1;
    for (auto son:nodes[root]) {
        sum += dfs(son);
        // TODO: optimize
        for (int j = sum; j >= 1; j--) {
            for (int k = 1; k < j; k++) {
                dp[root][j] = min(dp[root][j], dp[root][j - k] + dp[son][k] - 1);
            }
        }
    }
    return sum;
}

int dset[201];

struct Edge {
    int u, v;
};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, p;
    cin >> n >> p;
    int u, v;
    dset[1] = 1;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = n;
        }
    }
    // 构建单向边
    map<int, Edge> edge;
    int cnt = 0;
    for (int i = 1; i < n; i++) {
        cin >> u >> v;
        // 干就完事了
        if (u == 1) {
            nodes[u].insert(v);
            dset[v] = u;
        } else if (v == 1) {
            nodes[v].insert(u);
            dset[u] = v;
        } else if (dset[u]) {
            nodes[u].insert(v);
            dset[v] = 1;
        } else if (dset[v]) {
            nodes[v].insert(u);
            dset[u] = 1;
        } else {
            // 姑且记录一下
            edge[cnt].u = u;
            edge[cnt].v = v;
            cnt += 1;
        }
    }
    set<int> af;
    while (!edge.empty()) {
        for (auto &it:edge) {
            if (dset[it.second.u]) {
                nodes[it.second.u].insert(it.second.v);
                dset[it.second.v] = 1;
                af.insert(it.first);
            } else if (dset[it.second.v]) {
                nodes[it.second.v].insert(it.second.u);
                dset[it.second.u] = 1;
                af.insert(it.first);
            }
        }
        for (auto &it:af) {
            edge.erase(it);
        }
        af.clear();
    }
    // init
    for (int i = 1; i <= n; i++) {
        dp[i][1] = nodes[i].size();
    }
    // search
    dfs(1);
    int ans = dp[1][p];
#if DISPLAY_A
    int idx = 1;
#endif
    for (int i = 2; i <= n; i++) {
#if DISPLAY_A
        if (ans > dp[i][p] + 1) {
            ans = dp[i][p] + 1;
            idx = i;
        }
#else
        ans = min(ans, dp[i][p] + 1);
#endif
    }
#if DISPLAY_A
    cout << ans << " " << idx << endl;
#else
    cout << ans << endl;
#endif
    return 0;
}
```

### 测试例生成（Python）

```Python
def N(n, p, filename='in'):
    n = int(n)
    p = int(p)
    f = open(filename, 'w')
    f.write("{} {}\r\n{}\r\n".format(
        n, p,
        "\r\n".join([
            "{} {}".format(random.randint(1, i - 1), i) for i in range(2, n + 1)
        ])
    ))
```
