---
title: UESTC 2402
tags:
  - 珂朵莉树
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 珂朵莉树
abbrlink: 47384
date: 2020-05-08 13:06:22
---

## 题目

这次在每一轮游戏中，綾波需要使用鱼雷把一个区间[l,r]内的所有点的数值变为c，每改变一个点的数值需要消耗一枚特制鱼雷。

尽管如此，在知道了墙面上各点初始数值和每轮的任务后，綾波还是希望指挥官能帮她计算她各轮需要的鱼雷数量的说。呐，快来帮帮綾波吧。

## 题解

参照出题人题解PPT。

出题人告诉我们这题可以分块也可以用珂朵莉树。

分块实在太恶心了，所以我就试了下所谓的珂朵莉树，没有去看OI-WiKi，凭着感觉写的，没有用std::set而是用的std::map，当然他们内部也差不多，只是std::map可以很容易地带上一个载荷。

首先是初始化，把所有连续的部分都装进去。

对于每次查询，用二分查找到对对应的块，整块的就直接加到`ans`，其他的部分单独处理。

因为第一次写这种结构，还出了不少bug。

第一次查询和修改的复杂度都是$O(n)$，之后每次查询和修改的理想复杂度都是$O(\log_2n)$，因为总共$m$次查询，所以总的复杂度大概是$O(m\log_2n)$。

## 代码

有个宏定义`DISPLAY_A`用于在每次查询后打印完整的数组。

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

class Node {
public:
    int left, right;
    int val;

    Node(int left, int val, int right) : left(left), val(val), right(right) {}


    Node(int left, int val) : left(left), val(val), right(left) {}

    Node(int left) : left(left), val(0), right(left) {}

    Node() : left(0), val(0), right(1) {}

    bool operator<(Node node) const {
        return this->left < node.left;
    }
};

int arr[100000 + 7];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    int l, r, c;
    map<int, Node> tree;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    for (int i = 1; i <= n; i++) {
        int right = i;
        int i2 = i + 1;
        for (; i2 <= n; i2++) {
            if (arr[i2] != arr[i2 - 1]) {
                break;
            } else {
                right += 1;
            }
        }
        tree[i] = Node(i, arr[i], right);
        i = i2 - 1;
    }
    cin >> m;
    while (m--) {
        int ans = 0;
        cin >> l >> r >> c;
        auto it = tree.lower_bound(l);
        auto st = it;
        if (it->first != l) {
            st--;
            // 前面的区域
            if (st->second.val != c) {
                if (st->second.right > r) {
                    ans += r - l + 1;
                    tree[r + 1] = Node(r + 1, st->second.val, st->second.right);
                } else {
                    ans += st->second.right - l + 1;
                }
                st->second.right = l - 1;
            } else {
                l = st->first;
                if (r < st->second.right) {
                    r = st->second.right;
                }
            }
        }
        // 覆盖的部分
        for (; it->second.right <= r && it != tree.end();) {
            auto &node = it->second;
            if (node.val != c) {
                ans += node.right - node.left + 1;
            }
            it++;
            tree.erase(prev(it));
        }
        // 后面的区域
        auto et = it;
        if (et->first <= r && et != tree.end()) {
            if (et->second.val != c) {
                ans += r - et->first + 1;
                if (et->second.right - et->first > 0) {
                    // 长度大于1的时候才行，不然会错位
                    tree[r + 1] = Node(r + 1, et->second.val, et->second.right);
                }
                tree.erase(et);
            } else {
                r = et->second.right;
                tree.erase(et);
            }
        }
        tree[l] = Node(l, c, r);
#if DISPLAY_A
        for (auto itt:tree) {
            for (int i = itt.second.left; i <= itt.second.right; i++) {
                cout << itt.second.val << " ";
            }
        }
#endif
        cout << ans << endl;
    }
    return 0;
}
```

### 测试例生成（Python）

```Python
def z(n, m, max=2 ** 31 - 1, filename='in'):
    n = int(n)
    m = int(m)
    fi = open(filename, 'w')
    fi.write("{}\r\n{}\r\n{}\r\n{}\r\n".format(
        n, " ".join([str(random.randint(-max, max)) for i in range(n)]),
        m,
        "\r\n".join([
            "{} {} {}".format(
                random.randint(1, n // 2),
                random.randint(n // 2, n),
                random.randint(-max, max),
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

#define DISPLAY_A 1

using namespace std;
int arr[100000 + 7];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    int l, r, c;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> arr[i];
    }
    cin >> m;
    while (m--) {
        cin >> l >> r >> c;
        int ans = 0;
        for (int i = l; i <= r; i++) {
            if (c != arr[i]) {
                ans += 1;
                arr[i] = c;
            }
        }
#if DISPLAY_A
        for (int i = 1; i <= n; i++) {
            cout << arr[i] << " ";
        }
#endif
        cout << ans << endl;
    }
    return 0;
}
```
