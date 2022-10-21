---
title: UESTC 2361
tags:
  - 并查集
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 并查集
abbrlink: 55786
date: 2020-05-05 18:11:04
---


> 关键字：并查集；

## 快慢指针法

这个方法来自于判断链表中是否存在环，原理很直观，就是加入一个移动较慢的指针（一般是快指针速度的一半），如果链表中有环的话，就会导致两个指针出发了一段时间后“撞”在一起。所以只要在出发后判断两个指针是否相等即可确定链表中是否有环。

## 题解

并查集模板题，快慢指针法判断是否成环，再加个路径压缩就完事。复杂度$o(n)$，这题限时是200ms，太极限了。

## 代码

### AC代码

```C++
// @author: hzy
#pragma G++ optimize("O3")

#include <algorithm>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <map>
#include <set>
#include <string>
#include <stack>
#include <unordered_set>
#include <vector>

using namespace std;

int a[1000000];
int isLoop = 0;

int findSet(int fast, int slow, bool moveSlow = false, int step = 0) {
    if (fast == a[fast]) {
        isLoop = 0;
        return fast;
    } else {
        fast = a[fast];
        if (moveSlow) {
            slow = a[slow];
        } else {
            if (step && fast == slow) {
                // loop
                isLoop = 1;
                return fast;
            }
        }
        return findSet(fast, slow, !moveSlow, step + 1);
    }
}

void mergeSet(int x) {
    if (x == a[x]) {
        return;
    }
    int t = a[x];
    a[x] = x;
    mergeSet(t);
}

int main() {
    // std::ios::sync_with_stdio(false);
    // std::cin.tie(0);
    int n;
    cin >> n;
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        //cin >> a[i];
        scanf("%d", a + i);
        if (a[i] == i) {
            ans += 1;
        }
    }
    for (int i = 1; i <= n; i++) {
        if (a[i] == i) {
            continue;
        }
        findSet(i, i);
        if (isLoop) {
            // new loop
            ans += 1;
        }
        // compress
        mergeSet(i);
    }
    cout << ans;
    return 0;
}
```

其中的`findSet`函数用于查找根结点，成环的话会返回环上某一点，具体是哪个这里不展开说，同时会标记`isLoop`变量。

`mergeSet`用于路径压缩，如果我们只关心一个结点的根结点，在`findSet`完成之后直接把根结点设为它的父结点，这样如果以后还有结点在`findSet`过程中路过这个结点就可以快速找到根结点了，再引申一下，如果将路径上所有的结点都重新设置父亲结点，就可以加快经过那些结点的`findSet`。再优化一下，`ans+=1`之后，这个结点的根结点其实也不重要了，它的子结点必然不需要`ans+=1`，所以直接把路径上每个结点都设为根节点，速度又能提升一些。
