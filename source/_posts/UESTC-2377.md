---
title: UESTC 2377
tags:
  - 线段树
  - 主席树
  - 持久化线段树
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 线段树
abbrlink: 19307
date: 2020-05-05 18:13:05
---


## 题目

可持久化线段树模板题。

给一段序列，完成两个操作

- 从一个版本新建一个版本，然后单点修改
- 查询某个版本中某个点的值

## 题解

其实模板题没什么好说的，看下可持久化线段树的工作方式就知道了。[OI-WiKi](https://oi-wiki.org/ds/persistent-seg/)真是个好东西。这里还是描述一下怎么做吧。

假如我们要更新原数组中的1号元素，可以从上往下也可以从下往上，我选择的是从上往下。如下图

![img](https://oi-wiki.org/ds/images/persistent-seg.png)

可以发现，每次更新一个点，只需要将查询路径中所有的节点都拷贝一份就好了。假如要更新的位置`x`在当前节点区间的左边，那么新拷贝的节点右儿子不变，左儿子建立一个拷贝，然后递归下去。复杂度和一般线段树一样，查询和修改都是$O(\log_2n)$。

## 代码

### AC代码

居然一发入魂，懒得写测试例生成程序和暴力验证了。

个人觉得不是很优雅，在类中要使用自身的引用会有的麻烦，直接指针了。

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
const long long MAX = 1e6 + 7;

//节点，包含区间左、区间右、当前值
struct Node {
    int left = 1, right = 1;
    bool init = false;
    int value = 0;
    Node *leftSon;
    Node *rightSon;
};

int a[MAX];

Node &buildTree(int left, int right) {
    auto &node = *(new Node[1]);
    node.left = left;
    node.right = right;
    node.init = true;
    if (left == right) {
        node.value = a[left];
        return node;
    }
    int mid = (left + right) / 2;
    node.leftSon = &buildTree(left, mid);
    node.rightSon = &buildTree(mid + 1, right);
    return node;
}

Node &update(int x, int y, Node &node) {
    auto &newNode = *(new Node[1]);
    memcpy(&newNode, &node, sizeof(Node));
    if (node.left == node.right) {
        newNode.value = y;
        return newNode;
    }
    // 更新并生成新版本
    int mid = (node.left + node.right) / 2;
    if (x <= mid) {
        // left
        newNode.leftSon = &update(x, y, *node.leftSon);
    } else {
        newNode.rightSon = &update(x, y, *node.rightSon);
    }
    return newNode;
}

int query(int x, Node &node) {
    if (node.left == node.right) {
        return node.value;
    }
    int mid = (node.left + node.right) / 2;
    if (x <= mid) {
        return query(x, *node.leftSon);
    } else {
        return query(x, *node.rightSon);
    }
}

Node root[100000];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    char c;
    int v, x, y;
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    // build tree
    int version = 0;
    root[version] = buildTree(1, n);
    for (int i = 1; i <= m; i++) {
        cin >> c >> v >> x;
        if (c == '1') {
            // update
            cin >> y;
            version += 1;
            root[version] = update(x, y, root[v]);
        } else {
            // query
            cout << query(x, root[v]) << endl;
        }
    }
    return 0;
}
```
