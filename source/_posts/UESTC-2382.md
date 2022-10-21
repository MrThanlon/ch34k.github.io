---
title: UESTC 2382
tags:
  - 欧拉图
  - 图论
  - 刷题
categories:
  - ICPC训练
  - 图论
  - 欧拉图
abbrlink: 47278
date: 2020-06-19 16:00:59
---


## 题目

输入$n$条有向边，确定是否能一笔画，如果能则输出最大的“起点*终点“，否则输出$-1$。

## 题解

欧拉图模板题。

<!--more-->

根据维基百科上的说法，一个连通的有向图有从$u$到$v$欧拉路径的充要条件是$u$的出度比入度多1，$v$的入度比出度多1，其他节点出度等于入度，一个连通的有向图有欧拉回路的充要条件是所有节点的出度等于入度。

所以直接遍历边确定出度和入度就好了。

这题有个坑是给的有向图不一定连通，如果不连通的话当然不会有欧拉路径或欧拉回路。考虑到有向图的单连通+强连通等价于对应无向图的连通，无向图判断连通性的方法有很多，DFS、BFS等等，但是我觉得搜索太麻烦，就用的并查集。

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

class Edges {
public:
    int u, v, ans;

    Edges(int u, int v) : u(u), v(v), ans(u * v) {}

    bool operator<(Edges &e) const {
        return ans < e.ans;
    }
};

// 并查集判断连通性
int dset[101];
int iSet[101];

int find(int x) {
    if (dset[x] == x) {
        return x;
    }
    dset[x] = find(dset[x]);
    return dset[x];
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int pot[101];
    memset(pot, 0, sizeof(int) * 101);
    int n;
    cin >> n;
    int u, v;
    vector<Edges> ct;
    for (int i = 0; i < n; i++) {
        cin >> u >> v;
        // u -> v
        pot[u] -= 1;
        pot[v] += 1;
        ct.emplace_back(u, v);
        // 建立并查集，应该能保证不会有环
        if (dset[u]) {
            // u在一个集合了
            if (dset[v]) {
                if (find(u) != find(v)) {
                    // 连接
                    dset[find(v)] = find(u);
                }
                // 已经连通，不管
            } else {
                dset[v] = find(u);
            }
        } else if (dset[v]) {
            // dset[u] is 0
            dset[u] = find(v);
        } else {
            // 都没有，先建立吧
            dset[u] = u;
            dset[v] = u;
        }
    }
    int numSet = 0;
    for (int i = 1; i <= 100; i++) {
        if (!dset[i]) {
            continue;
        }
        int root = find(i);
        if (!iSet[root]) {
            iSet[root] = 1;
            numSet += 1;
            if (numSet > 1) {
                cout << -1 << endl;
                return 0;
            }
        }
    }
    int start = 0, end = 0;
    for (int i = 1; i <= 100; i++) {
        if (pot[i] == 1) {
            if (!end) {
                end = i;
            } else {
                // unavailable
                cout << -1 << endl;
                return 0;
            }
        } else if (pot[i] == -1) {
            if (!start) {
                start = i;
            } else {
                cout << -1 << endl;
                return 0;
            }
        } else if (pot[i] != 0) {
            cout << -1 << endl;
            return 0;
        }
    }
    if (start && end) {
        cout << start * end << endl;
    } else if (start == 0 && end == 0) {
        // start和end都为0，找最大的
        int maxAns = 0;
        for (auto &i:ct) {
            maxAns = max(maxAns, max(i.u, i.v));
        }
        cout << maxAns * maxAns << endl;
    } else {
        // 只有start或者只有end
        cout << -1 << endl;
    }
    return 0;
}

```
