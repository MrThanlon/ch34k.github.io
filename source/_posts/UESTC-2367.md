---
title: UESTC 2367
tags:
  - 分块
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 分块
abbrlink: 56170
date: 2020-05-07 20:46:21
---

## 题目

在与拉菲、标枪等人相遇几次之后，綾波心里有些烦恼。一天，她独自在港区玩一个既有趣又能提升自己的能力的游戏，在这个游戏中，她面对一面长墙，墙上排列着 n个点（编号 1∼n），每个点都带有一个数值（可能为负数），同时，还有一个一定要记住的数值 k。

在每一轮游戏中，綾波需要使用鱼雷击中在一个区间（[l,r]）内的所有对 k取模（取**非负最小**剩余）后数值与 c对 k取模（同样取**非负最小**剩余）相等的点，每次击中每个点需要一枚鱼雷。

作为港区鱼雷能力最强的驱逐舰，綾波当然不可能因为没有命中而浪费鱼雷。

每轮游戏结束后，该轮目标区间内的所有点的值将增加 d（d 可能为负数）。

## 题解

参照出题人的PPT题解。

先做分块，把数组分成$\sqrt{n}$块，最后一块适当加长。

因为取模对加减符合交换律结合律分配律，所以为了不溢出，每次都取模就好了。

因为取模之后最大为$k$，直接用桶就行，对每个区块放一个长度$k$对桶，记录取模后各个数字的数量。

对于每次查询，如果$l$和$r$在一个区块内就直接暴力，否则对整块的部分直接加上，剩下的暴力。

修改的时候区块可以用一个偏移量来记录，暴力的部分直接加上就行。

每次查询和修改的时间复杂度都是$O(\sqrt n)$，共有$m$次查询，所以总的时间复杂度是$O(m\log_2n)$。

## 代码

放了个`DISPLAY_A`宏定义用来在调试的时候输出数组。

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
const long long MAX = 1e5 + 7;
int k = 1;
int blockSize, numBlock;

int mod(int x) {
    int ans = x < 0 ? x % k + k : x % k;
    return ans >= k ? ans % k : ans;
}

int getBlock(int x) {
    int ans = (x - 1) / blockSize;
    if (ans == numBlock) {
        return ans - 1;
    }
    return ans;
}

struct Block {
    int left, right;
    int offset;
    int buck[MAX];
};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    cin >> n >> k;
    int a[n + 1];
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i] = mod(a[i]);
    }
    // 分块
    numBlock = sqrt(n);
    blockSize = n / numBlock;
    vector<Block> blocks(316);
    blocks[0].left = 1;
    blocks[0].right = blockSize;
    for (int i = 1; i < numBlock; i++) {
        blocks[i].left = blocks[i - 1].right + 1;
        if (i == numBlock - 1) {
            blocks[i].right = n;
        } else {
            blocks[i].right = blocks[i].left + blockSize - 1;
        }
    }
    // 预处理
    for (int i = 0; i < numBlock; i++) {
        for (int i2 = blocks[i].left; i2 <= blocks[i].right; i2++) {
            blocks[i].buck[a[i2]] += 1;
        }
    }
    // 查询
    int m, l, r, c, d;
    cin >> m;
    while (m--) {
        cin >> l >> r >> c >> d;
        int ans = 0;
        // 分块起点
        // s为l所在区块
        int s = getBlock(l);
        // 分块终点
        // e为r所在区块
        int e = getBlock(r);
        if (s == e) {
            // 在同一个区块，直接遍历吧
            int t = mod(mod(c) - blocks[s].offset);
            for (int i = l; i <= r; i++) {
                if (a[i] == t) {
                    ans += 1;
                }
                blocks[s].buck[a[i]] -= 1;
                a[i] = mod(a[i] + mod(d));
                blocks[s].buck[a[i]] += 1;
            }
#if DISPLAY_A
            for (int i = 1; i <= n; i++) {
                cout << mod(a[i] + blocks[getBlock(i)].offset) << " ";
            }
#endif
            cout << ans << endl;
            continue;
        }
        int t = 0;
        if (l != blocks[s].left) {
            // c % k == (a[i] + offset) % k
            t = mod(mod(c) - blocks[s].offset);
            for (int i = l; i <= blocks[s].right; i++) {
                // a[i] + offset == t
                if (a[i] == t) {
                    ans += 1;
                }
                blocks[s].buck[a[i]] -= 1;
                a[i] = mod(a[i] + mod(d));
                blocks[s].buck[a[i]] += 1;
            }
        } else {
            s -= 1;
        }
        if (r != blocks[e].right) {
            t = mod(mod(c) - blocks[e].offset);
            for (int i = blocks[e].left; i <= r; i++) {
                if (a[i] == t) {
                    ans += 1;
                }
                blocks[e].buck[a[i]] -= 1;
                a[i] = mod(a[i] + mod(d));
                blocks[e].buck[a[i]] += 1;
            }
        } else {
            e += 1;
        }
        // 分块查询
        for (int i = s + 1; i < e; i++) {
            ans += blocks[i].buck[mod(mod(c) - blocks[i].offset)];
            blocks[i].offset = mod(blocks[i].offset + mod(d));
        }
#if DISPLAY_A
        for (int i = 1; i <= n; i++) {
            cout << mod(a[i] + blocks[getBlock(i)].offset) << " ";
        }
#endif
        cout << ans << endl;
    }
    return 0;
}
```

### 测试例生成（Python）

```Python
#!/bin/env python
import random


def h(n, m, k=random.randint(1, 1e5), max=2 ** 31 - 1, filename='in'):
    n = int(n)
    m = int(m)
    k = int(k)
    fi = open(filename, 'w')
    fi.write("{} {}\r\n{}\r\n{}\r\n{}\r\n".format(
        n, k, " ".join([str(random.randint(-max, max)) for i in range(n)]),
        m,
        "\r\n".join([
            "{} {} {} {}".format(
                random.randint(1, n // 2),
                random.randint(n // 2, n),
                random.randint(-max, max),
                random.randint(-max, max)
            ) for i in range(m)
        ])
    ))
    fi.close()

```

### 暴力验证

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

#define DISPLAY_A 0

using namespace std;

int k;

int mod(int x) {
    int ans = x < 0 ? x % k + k : x % k;
    return ans >= k ? ans % k : ans;
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    cin >> n >> k;
    int a[n + 1];
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
        a[i] = mod(a[i]);
    }
    cin >> m;
    int l, r, c, d;
    while (m--) {
        cin >> l >> r >> c >> d;
        c = mod(c);
        int ans = 0;
        for (int i = l; i <= r; i++) {
            if (a[i] == c) {
                ans += 1;
            }
            a[i] = mod(a[i] + mod(d));
        }
#if DISPLAY_A
        for (int i = 1; i <= n; i++) {
            cout << a[i] << " ";
        }
#endif
        cout << ans << endl;
    }
    return 0;
}
```
