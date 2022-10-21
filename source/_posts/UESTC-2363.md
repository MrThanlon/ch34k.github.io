---
title: UESTC 2363
tags:
  - 线段树
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 线段树
abbrlink: 6251
date: 2020-05-07 14:53:29
---


## 题目

利姆露现在想知道一个奇怪魔法序列的含义，于是她把这个任务交给了大贤者。

这个魔法序列由 n*n* 个字符组成，仅包含 L，R，小写拉丁字母和括号（即 ‘(’ 和 ‘)’），表达对一个空白字符串的操作（因为是空白字符串，所以最初操作位置在哪里都一样），其中

1. L 表示当前操作位置左移一位。
2. R 表示当前操作位置右移一位。
3. 其余任何小写拉丁字母或括号表示将当前操作位置修改为该字符。

大贤者知道这个操作产生的序列对应一个魔法，当序列中括号不合法时，魔法会失败；而括号序列合法时，括号的最大嵌套重数就是魔法的强度。

现在大贤者想知道这个过程中每一步产生的序列对应的魔法是一个失败的魔法还是一个成功的魔法，若是一个成功的魔法，则其威力是多少？你能够帮忙完成这件事吗？

（不知道怎么概括，直接复制粘贴了）

## 题解

参考出题人给的题解PPT。

主要是维护一个编辑器，同时能快速得到字符串中括号是否匹配，以及括号的嵌套层数。

我一开始看到括号匹配就想到栈，但是每次都遍历一次铁定超时。实际上我们只需要对字符串求一个前缀和，`(`加1，`)`减1，如果前缀和数组中有小于0的数或者最后一位不为0（即左右括号数量不相等）则非法，前缀和数组的最大值则是最大嵌套数。所以只需要维护一个前缀和数组，以及最大值和最小值即可，原数组的单点修改会导致前缀和数组的区间修改，用线段树可以实现复杂度$O(\log_2 n)$完成，查询最大值和最小值也是。所以总的复杂度是$O(n\log_2n)$。

因为光标可能会移动到左边，所以可以从数组中间开始。

## 代码

因为没有发生WA，而只有RE（数组越界），所以就没写暴力验证。

RE的原因是数组开小了，$n$最大是1e6，所以数组应该要开到2e6长度。

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

using namespace std;
const long long MAX = 1e6 + 7;

char arr[MAX * 2];

struct Node {
    int left, right;
    int val, min, max, lazy;
};

Node tree[MAX * 8];

void build(int left, int right, int treeIndex = 1) {
    auto &node = tree[treeIndex];
    node.left = left;
    node.right = right;
    if (left != right) {
        int mid = (left + right) / 2;
        build(left, mid, treeIndex * 2);
        build(mid + 1, right, treeIndex * 2 + 1);
    }
}

void update(int left, int right, int val, int treeIndex = 1) {
    auto &node = tree[treeIndex];
    if (node.left == node.right) {
        node.val += val;
        node.max = node.val;
        node.min = node.val;
        return;
    }
    if (node.left >= left && node.right <= right) {
        // 子集
        node.lazy += val;
        node.max += val;
        node.min += val;
        return;
    }
    int mid = (node.left + node.right) / 2;
    if (mid >= left) {
        update(left, right, val, treeIndex * 2);
    }
    if (mid < right) {
        update(left, right, val, treeIndex * 2 + 1);
    }
    node.max = max(tree[treeIndex * 2].max, tree[treeIndex * 2 + 1].max) + node.lazy;
    node.min = min(tree[treeIndex * 2].min, tree[treeIndex * 2 + 1].min) + node.lazy;
}

int query(int pos, int treeIndex = 1) {
    auto &node = tree[treeIndex];
    if (node.left == node.right) {
        return node.val;
    }
    int mid = (node.left + node.right) / 2;
    // FIXME: 优化一波？
    if (pos <= mid) {
        return node.lazy + query(pos, treeIndex * 2);
    } else {
        return node.lazy + query(pos, treeIndex * 2 + 1);
    }
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    int arrCursor = MAX;
    cin >> n;
    char c;
    const int rMax = arrCursor + n + 5;
    const int lMin = arrCursor - n + 5;
    build(lMin, rMax);
    while (n--) {
        cin >> c;
        if (c == 'L') {
            arrCursor -= 1;
        } else if (c == 'R') {
            arrCursor += 1;
        } else {
            char origin = arr[arrCursor];
            arr[arrCursor] = c;
            if (c == '(') {
                if (origin == ')') {
                    // +2
                    update(arrCursor, rMax, 2);
                } else if (origin != '(') {
                    // +1
                    update(arrCursor, rMax, 1);
                }
            } else if (c == ')') {
                if (origin == '(') {
                    // -2
                    update(arrCursor, rMax, -2);
                } else if (origin != ')') {
                    update(arrCursor, rMax, -1);
                }
            } else {
                if (origin == '(') {
                    // -1
                    update(arrCursor, rMax, -1);
                } else if (origin == ')') {
                    // +1
                    update(arrCursor, rMax, 1);
                }
            }
        }
        if (tree[1].min >= 0 && query(rMax) == 0) {
            cout << tree[1].max << " ";
        } else {
            cout << -1 << " ";
        }
    }
    return 0;
}
```

测试例生成（Python）

```Python
#!/bin/env python
import random


def d(n, filename='in'):
    n = int(n)
    arr = ['(', ')', 'L', 'R', 'a']
    random.shuffle(arr)
    a = [arr[random.randint(0, 4)] for i in range(n)]
    fi = open(filename, 'w')
    fi.write("{}\r\n{}\r\n".format(
        n, "".join(a)
    ))
    fi.close()

```

