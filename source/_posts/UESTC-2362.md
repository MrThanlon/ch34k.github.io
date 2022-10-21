---
title: UESTC 2362
tags:
  - 前缀和
  - 逆序对
  - 树状数组
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 树状数组
abbrlink: 55466
date: 2020-05-05 18:12:50
---


> 关键字：前缀和；逆序对；树状数组；

这题实属有丶东西。

首先确定分母，显然$l$和$r$共有$\frac{n(n+1)}{2}$种取值，所以分母就是那么多。

这题需要算法的部分主要在分子，即有多少个$(l,r)$对使得平均值小于等于$k$。

首先为了方便，我们将输入的每个元素全部减去$k$，这样只需要判断$[l,r]$区间上的和是否小于等于0即可即可，于是很自然地，我们可以算一下前缀和，这里用$S_i$表示。

那么对于$[l,r]$区间上的和，我们可以很方便地用$A=S_r-S_{l-1}$求出来，显然我们知道，$r\ge l$，而$l$和$r$都是整数，所以有$r>l-1$，现在联立一下，得到
$$
\cases{
S_r-S_{l-1}\le0 \\
r>l-1
}
$$
这不就是妥妥的逆序对吗？也就是说，只要求出$S_i$的逆序对即可。

一般求逆序对可以暴力（时间复杂度$O(n^2)$）、归并排序（时间复杂度$O(n\log_2n)$）、树状数组（时间复杂度$O(n\log_2n)$），因为是数据结构专题，很自然地就选树状数组了。

求逆序对的思路参照[leetcode 面试题51](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)，首先对输入的序列，可以从后向前遍历，设当前遍历的元素是$a_i$，那么我们只需要找到在$a_i$之后且小于等于$a_i$的元素个数。具体来说，我们可以用一个桶，并求出桶中$1$到$a_i$的区间和，然后另第$a_i$个元素自增即可，因为桶需要更新，所以可以用树状数组或者线段树来完成，查询和更新的复杂度都是$O(\log n)$。

另外要注意，$a_i$的值可能很大，在题目中范围是1e9，然而我们并没有那么大的内存去开1e9长度的数组作为桶。实际上我们只要求逆序对的个数，对逆序对的具体值并不关心，只要他们的大小关系没问题就行，所以我们可以使用离散化算法，使得$a_i$的值缩小到$n$，具体做法是先拷贝到另一个数组，然后排序，再二分查找回来，就可以将原数组映射到大小为$n$的值域内。离散化算法的复杂度是$O(n\log n)$。

最后的输出就没啥了，gcd求最大公约数，分子分母都除以最法公约数，加个斜杠输出。

## 细节

- int会溢出。
- 答案为0单独判断，输出`0/1`。
- 在求逆序对的时候如果$l-1$从1开始，那么$l$就得从0开始，所以得注意前缀和数组第一个元素，要么在前缀和数组头部添一个0，要么单独遍历$l=0$的情况。

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
const long long MAX = 2e5 + 7;

long long gcd(long long a, long long b) {
    return a == 0 ? b : gcd(b % a, a);
}

class BIT {
private:
    vector<long long> tree;
    long long n;

public:
    BIT(long long _n) : n(_n), tree(_n + 1) {}

    static long long lowbit(long long x) {
        return x & (-x);
    }

    // 查询前缀和
    long long query(long long x) {
        long long ret = 0;
        while (x) {
            ret += tree[x];
            x -= lowbit(x);
        }
        return ret;
    }

    // 自增1
    void update(long long x) {
        while (x <= n) {
            tree[x]++;
            x += lowbit(x);
        }
    }
};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    long long n, k;
    cin >> n >> k;
    vector<long long> sum(n + 1), arr(n + 1);
    long long ans = 0;
    long long tmp;
    for (long long i = 1; i <= n; i++) {
        cin >> tmp;
        arr[i] = tmp - k;
    }
    // prefix sum
    for (long long i = 1; i <= n; i++) {
        sum[i] = arr[i] + sum[i - 1];
    }
    // 求逆序对
    // 离散化
    vector<long long> tmpArr = sum;
    // for l=1
    sort(tmpArr.begin(), tmpArr.end());
    for (long long i = 0; i <= n; i++) {
        sum[i] = lower_bound(tmpArr.begin(), tmpArr.end(), sum[i]) - tmpArr.begin() + 1;
    }
    // 树状数组统计逆序对
    BIT bit(n + 1);
    for (long long i = n; i >= 0; i--) {
        ans += bit.query(sum[i]);
        bit.update(sum[i]);
    }
    // cout << ans << endl;
    // return 0;
    if (ans == 0) {
        cout << "0/1" << endl;
    } else {
        long long g = gcd(ans, n * (n + 1) / 2);
        cout << ans / g << "/" << n * (n + 1) / 2 / g;
    }
    return 0;
}

```

### 测试例生成（Python）

```Python
def c(n, k):
     a = [str(random.randint(1,2 * k)) for i in range(n)]
     f = open('in', 'w')
     f.write("{} {}\r\n{}\r\n".format(n, k, " ".join(a)))
     f.close()
```

### 暴力法（用于验证）

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
const long long MAX = 2e5 + 7;

long long gcd(long long a, long long b) {
    return a == 0 ? b : gcd(b % a, a);
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    long long n, k;
    cin >> n >> k;
    vector<long long> sum(n + 1), arr(n + 1);
    long long ans = 0;
    long long tmp;
    for (long long i = 1; i <= n; i++) {
        cin >> tmp;
        arr[i] = tmp - k;
        if (arr[i] <= 0) {
            ans += 1;
        }
    }
    // prefix sum
    for (long long i = 1; i <= n; i++) {
        sum[i] = arr[i] + sum[i - 1];
    }
    // 暴力
    for (long long l = 1; l < n; l++) {
        for (long long r = l + 1; r <= n; r++) {
            if (sum[r] - sum[l - 1] <= 0) {
                ans += 1;
            }
        }
    }
    // cout << ans << endl;
    // return 0;
    if (ans == 0) {
        cout << "0/1" << endl;
    } else {
        long long g = gcd(ans, n * (n + 1) / 2);
        cout << ans / g << "/" << n * (n + 1) / 2 / g;
    }
    return 0;
}

```
