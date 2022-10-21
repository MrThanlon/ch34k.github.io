---
title: 树状数组的一个优雅实现（C++）
tags:
  - 树状数组
  - 数据结构
  - ICPC
  - 刷题
categories:
  - 数据结构
  - 树状数组
abbrlink: 13506
date: 2020-04-29 20:35:43
---

废话不多说，直接上代码：

```C++
class BIT {
private:
    vector<int> tree;
    int n;
public:
    BIT(int _n) : n(_n), tree(_n + 1) {}

    static int lowbit(int x) {
        return x & (-x);
    }

    // 查询前缀和
    int query(int x) {
        int ret = 0;
        while (x) {
            ret += tree[x];
            x -= lowbit(x);
        }
        return ret;
    }

    // 更新
    void update(int x, int v) {
        while (x <= n) {
            tree[x] = v;
            x += lowbit(x);
        }
    }
};
```

从leetcode的一个题解里面抄的，构造函数接受一个参数`_n`表示数组长度，并初始化（为0），做了一些改动，感觉非常舒适。

## TODO

- [ ] 从数组或者迭代器中初始化
- [ ] 使用模板，不局限于int类型
