---
title: UESTC 2374
tags:
  - 线段树
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 线段树
abbrlink: 18987
date: 2020-05-05 18:13:08
---


## 题目

一个环，环上有点，查询从$s$点到最近的大于等于$t$的点的正向距离，没有则输出$-1$。每个点在被查询过后会消失，没消失的点的值每次查询都增加1。

## 题解

基本上还是用线段树。

首先用一个`grow`变量标记过了多少个晚上，长了多少，然后就是用线段树查找出距离$s$最近的大于等于$t-\mathrm{grow}$的点，输出距离完事。

因为有环，所以在查找的时候需要做一些处理来拆环。我的做法比较省事，数组长度为$n$，先查找从$s$到$n$有没有，如果没有就查找从$0$到$s-1$，如果也没有就输出$-1$，有就计算距离输出。

因为是线段树，所以查询复杂度$O(\log_2 n)$，因为有$m$次查询，所以总的时间复杂度是$O(m\log_2 n)$。

## 代码

因为基本上是一发入魂（手点快了交了十多发完全一样的代码，有些AC有些TLE），所以就没写样例生成程序和暴力验证。

### AC代码

```C++
// @author: hzy
#pragma G++ optimize("O3")

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

using namespace std;
const long long MAX = 2e6 + 7;

int h[MAX];

struct Node {
    int left, right;
    int max;
    bool empty;
    Node *leftSon;
    Node *rightSon;
    Node *father;
};

Node &buildTree(int left, int right, Node *father = nullptr) {
    auto &node = *(new Node[1]);
    node.left = left;
    node.right = right;
    node.father = father;
    node.empty = false;
    if (left == right) {
        // leaf
        node.max = h[left];
        return node;
    }
    // build son
    int mid = (left + right) / 2;
    node.leftSon = &buildTree(left, mid, &node);
    node.rightSon = &buildTree(mid + 1, right, &node);
    node.max = max(node.leftSon->max, node.rightSon->max);
    return node;
}

void updateFatherMax(Node &node) {
    int newMax = max(node.leftSon->max, node.rightSon->max);
    if (newMax != node.max) {
        node.max = newMax;
        if (node.father != nullptr) {
            updateFatherMax(*node.father);
        }
    }
}

int queryMax(int left, int right, int t, Node &node) {
    if (node.empty || node.max < t) {
        return -1;
    }
    if (node.left == node.right) {
        // get caught
        node.empty = true;
        // update to father
        updateFatherMax(*node.father);
        return node.left;
    }
    int leftAns = -1;
    if (left <= node.left && right >= node.right) {
        // sub set
        leftAns = queryMax(left, right, t, *node.leftSon);
        return leftAns != -1 ? leftAns : queryMax(left, right, t, *node.rightSon);
    }
    int mid = (node.left + node.right) / 2;
    if (left <= mid) {
        leftAns = queryMax(left, right, t, *node.leftSon);
        if (leftAns != -1) {
            return leftAns;
        }
    }
    if (right > mid) {
        return queryMax(left, right, t, *node.rightSon);
    }
    return -1;
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> h[i];
    }
    // build tree
    auto root = buildTree(1, n);
    cin >> m;
    int s, t;
    int grow = 0;
    while (m--) {
        cin >> s >> t;
        int ans = -1;
        ans = queryMax(s + 1, n, t - grow, root);
        if (ans != -1) {
            ans = ans - s - 1;
        } else if (s != 0) {
            ans = queryMax(1, s, t - grow, root);
            if (ans != -1) {
                ans = (n - s - 1) + ans;
            }
        }
        cout << ans << " ";
        grow += 1;
    }
    return 0;
}
```
