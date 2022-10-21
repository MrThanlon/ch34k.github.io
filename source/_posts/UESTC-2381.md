---
title: UESTC 2381
tags:
  - 线段树
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 线段树
abbrlink: 47598
date: 2020-05-05 18:13:00
---


## 题目

查找长度为$n$的序列中第一个大于等于$k$的元素的位置，并将这个位置的值减去$k$，重复$m$次。

## 题解

直接线段树。

首先建立线段树，每个节点保存区间内的最大值，从根节点递归下去即可，复杂度$O(n\log_2n)$。因为只需要单点更新，其实zkw线段树可能更适合，不过这里普通线段树就能过，就没必要了。

查找的过程也很简单，从根节点开始，如果当前节点的`max`小于$k$直接返回$-1$，如果当前节点是叶子节点就返回自己的索引，否则先找左儿子，左儿子如果是$-1$就返回右儿子，每次查询的复杂度是$O(\log_2n)$。我在自己笔记本上测试，$T=1,n=10^6,m=10^6$情况下的随机数据，基本压着1秒的线，太极限了。

## 代码

### AC代码

有学长说这是主席树（持久化线段树），我也不知道是不是，照着[OI-WiKi](https://oi-wiki.org)写的。

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

#define TREE_SUM 0
#define TREE_MAX 1
#define TREE_MIN 0

//节点，包含区间和、最大值、最小值、区间左、区间右
struct Node {
#if TREE_SUM
    int sum;
#endif
#if TREE_MAX
    long long max;
#endif
#if TREE_MIN
    int min;
#endif
    int left, right;
};
Node tree[MAX * 4];

long long a[MAX];

class SegmentTree {
private:
    int n;
public:
    SegmentTree(int _n) : n(_n * 4)/*, tree(_n * 4 + 1)*/ {}

    SegmentTree(int _n, long long *a) : n(_n + 1)/*, tree(_n * 4 + 3)*/ {
        build(1, _n, 1, a);
    }

    void build(int left, int right, int index, const long long *a) {
        auto &node = tree[index];
        if (left == right) {
            // 当前值
#if TREE_SUM
            tree[index].sum = a[left];
#endif
#if TREE_MAX
            node.max = a[left];
#endif
#if TREE_MIN
            tree[index].min = a[left];
#endif
            node.left = left;
            node.right = left;
            return;
        }
        int mid = (left + right) / 2;
        build(left, mid, index * 2, a);
        build(mid + 1, right, index * 2 + 1, a);
#if TREE_SUM
        tree[index].sum = tree[index * 2].sum + tree[index * 2 + 1].sum;
#endif
#if TREE_MAX
        node.max = max(tree[index * 2].max, tree[index * 2 + 1].max);
#endif
#if TREE_MIN
        tree[index].min = min(tree[index * 2].min, tree[index * 2 + 1].min);
#endif
        node.left = left;
        node.right = right;
    }

#if TREE_SUM
    // 求区间和
    int querySum(int left, int right, int index = 1) {
        if (left <= tree[index].left && right >= tree[index].right) {
            // 子集
            return tree[index].sum;
        }
        int ans = 0;
        int mid = (tree[index].left + tree[index].right) / 2;
        if (left <= mid) {
            // [left,mid]与询问区间有交集，递归查询左儿子
            ans += querySum(left, right, index * 2);
        }
        if (right > mid) {
            // [mid+1,right]与询问区间有交集，递归查询左儿子
            ans += querySum(left, right, index * 2 + 1);
        }
        return ans;
    }
#endif
#if TREE_MAX

    // 求区间最大值
    int queryMax(int left, int right, int index = 1) {
        if (left <= tree[index].left && right >= tree[index].right) {
            // 子集
            return tree[index].max;
        }
        int leftAns = INT_MIN;
        int rightAns = INT_MIN;
        int mid = (tree[index].left + tree[index].right) / 2;
        if (left <= mid) {
            leftAns = queryMax(left, right, index * 2);
        }
        if (right > mid) {
            rightAns += queryMax(left, right, index * 2 + 1);
        }
        return max(leftAns, rightAns);
    }

#endif
#if TREE_MIN
    // 求区间最小值
    int queryMin(int left, int right, int index = 1) {
        if (left <= tree[index].left && right >= tree[index].right) {
            // 子集
            return tree[index].min;
        }
        int leftAns = INT_MAX;
        int rightAns = INT_MAX;
        int mid = (tree[index].left + tree[index].right) / 2;
        if (left <= mid) {
            leftAns = queryMin(left, right, index * 2);
        }
        if (right > mid) {
            rightAns = queryMin(left, right, index * 2 + 1);
        }
        return min(leftAns, rightAns);
    }
#endif
#if TREE_SUM
    // 更新父节点的区间和
    void updateFatherSum(int treeIndex) {
        tree[treeIndex].sum = tree[treeIndex * 2].sum + tree[treeIndex * 2 + 1].sum;
        if (treeIndex != 1) {
            updateFatherSum(treeIndex / 2);
        }
    }
#endif

#if TREE_MAX

    void updateFatherMax(int treeIndex) {
        auto &node = tree[treeIndex];
        auto newMax = max(tree[treeIndex * 2].max, tree[treeIndex * 2 + 1].max);
        if (newMax != node.max && treeIndex != 1) {
            node.max = newMax;
            updateFatherMax(treeIndex / 2);
        }
    }

#endif

    // 单点更新
    void update(int index, long long value, int treeIndex = 1) {
        auto node = &tree[treeIndex];
        if (node->left == node->right) {
            //当前节点，更新
#if TREE_SUM
            tree[treeIndex].sum = value;
            updateFatherSum(treeIndex);
#endif
#if TREE_MAX
            node->max = value;
            updateFatherMax(treeIndex / 2);
#endif
#if TREE_MIN
            tree[treeIndex].min = value;
#endif
            return;
        }
#if TREE_MAX
        // 逐渐变小，应该不需要
        // node->max = max(node->max, value);
#endif
#if TREE_MIN
        tree[treeIndex].min = min(tree[treeIndex].min, value);
#endif
        int mid = (node->left + node->right) / 2;
        if (index <= mid) {
            update(index, value, treeIndex * 2);
        } else {
            update(index, value, treeIndex * 2 + 1);
        }
    }

    // 查询第一个大于等于k的元素，通过max>=k确定
    int getAns(int left, int right, long long k, int treeIndex = 1) {
        auto &node = tree[treeIndex];
        if (node.max < k) {
            // 别找了，就没有
            return -1;
        }
        if (node.left == node.right) {
            // 叶子，返回索引
            a[node.left] -= k;
            node.max -= k;
            updateFatherMax(treeIndex / 2);
            return node.left;
        }
        int leftAns = -1, rightAns = -1;
        if (node.left >= left && node.right <= right) {
            // 此节点被包含在区间中
            // 先找左边，是否有大于等于k的
            leftAns = getAns(left, right, k, treeIndex * 2);
            if (leftAns != -1) {
                return leftAns;
            }
            return getAns(left, right, k, treeIndex * 2 + 1);
        }
        // 有交集
        int mid = (node.left + node.right) / 2;
        if (left <= mid) {
            leftAns = getAns(left, right, k, treeIndex * 2);
            if (leftAns != -1) {
                return leftAns;
            }
        }
        if (right > mid) {
            return getAns(left, right, k, treeIndex * 2 + 1);
        }
        return -1;
    }
};

auto st = SegmentTree(MAX);

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int t, n, m;
    int l, r, k;
    cin >> t;
    while (t--) {
        cin >> n >> m;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
        }
        // build tree
        st.build(1, n, 1, a);
        for (int i = 1; i <= m; i++) {
            int idx = -1;
            cin >> l >> r >> k;
            idx = st.getAns(l, r, k);
            cout << idx << " ";
        }
    }
    return 0;
}

```

### 测试例生成（Python）

```Python
#!/bin/env python
import random


def v(t, n, m, max=1e9, filename='in'):
    t = int(t)
    n = int(n)
    m = int(m)
    max = int(max)
    step = t // 10
    f = open(filename, 'w')
    f.write("{}\r\n".format(t))
    for i in range(t):
        a = [str(random.randint(1, max)) for i in range(n)]
        q = [" ".join([
            str(random.randint(1, n // 2)),
            str(random.randint(n // 2, n)),
            str(random.randint(1, max))]) for i in range(m)]
        f.write("{} {}\r\n{}\r\n{}\r\n".format(
            n, m, " ".join(a), "\r\n".join(q)
        ))
        if i % step == 0:
            print(i)
    f.close()

```

### 暴力验证

```C++
// @author: hzy
#pragma G++ optimize("O3")

#include <algorithm>
#include <cmath>
#include <cstdio>
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

using namespace std;
const long long MAX = 1e6 + 7;
long long a[MAX];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int t, n, m;
    int l, r;
    long long k;
    cin >> t;
    while (t--) {
        cin >> n >> m;
        for (int i = 1; i <= n; i++) {
            cin >> a[i];
        }
        while (m--) {
            cin >> l >> r >> k;
            // 暴力
            for (; l <= r; l++) {
                if (a[l] >= k) {
                    break;
                }
            }
            if (l <= r) {
                cout << l << " ";
                a[l] -= k;
            } else {
                cout << -1 << " ";
            }
        }
    }
    return 0;
}

```

