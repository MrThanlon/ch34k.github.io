---
title: UESTC 2364
tags:
  - ST表
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - ST表
abbrlink: 55850
date: 2020-05-05 18:13:13
---


## 题目

给一个$n$行$m$列的数据，找出其中

- 尽可能大
- 边长为奇数
- 中心元素能被其他元素整除

的正方形。

## 题解

出题人的PPT题解上写的挺清楚了，上述条件成立的情况下，则中心元素一定是正方形内所有元素中最小的，而且正方形内所有元素的最大公约数也是中心元素。

暴力枚举每个点就行。我们可以用二维ST表来实现复杂度$O(1)$求最小值，$O(\log_2 n)$求区域内的最大公约数（GCD）。下标从$0$开始的话，只需要从$(1,1)$枚举到$(n-1,m-1)$即可，因为边缘上的数不会有正方形。

对于每个枚举，首先确定忽略条件3时最大能取的正方形的边长是多少，比如$(i,j)$的话最大边长是

```C++
min(min(i, n - i - 1), min(j, m - j - 1))
```

然后通过二分查找确定条件3下的最大边长。

## 代码

### <del>WA</del> AC代码

<del>我真希望在这里写的是"AC代码"，可惜现在还是卡在第8个测试点。</del>

5.4 20:35 update: 因为把第102行的`i`写成了`j`导致一直卡在第8个test case而自闭。关于错误的位置可以看暴力验证代码的第100行（也就是说那个暴力程序是错误的）。

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

long long a[500][500];
long long st[500][500][10];
long long stGCD[500][500][10];

#define qmin(a, b, c, d) min(min(a,b),min(c,d))

long long gcd(long long x, long long y) {
    if (x == 0) {
        return y;
    }
    return gcd(y % x, x);
}

#define qgcd(a, b, c, d) gcd(gcd(a, b), gcd(c, d))

int pow2[10];

long long queryMin(long long midX, long long midY, long long len) {
    int k = log2(len * 2 + 1);
    return qmin(st[midX - len][midY - len][k],
                st[midX - len][midY + len - pow2[k] + 1][k],
                st[midX + len - pow2[k] + 1][midY - len][k],
                st[midX + len - pow2[k] + 1][midY + len - pow2[k] + 1][k]);
}

long long queryGCD(long long midX, long long midY, long long len) {
    int k = log2(len * 2 + 1);
    return qmin(stGCD[midX - len][midY - len][k],
                stGCD[midX - len][midY + len - pow2[k] + 1][k],
                stGCD[midX + len - pow2[k] + 1][midY - len][k],
                stGCD[midX + len - pow2[k] + 1][midY + len - pow2[k] + 1][k]);
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    cin >> n >> m;
    // n行m列
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> a[i][j];
            st[i][j][0] = a[i][j];
            stGCD[i][j][0] = a[i][j];
        }
    }
    // 预处理
    int len = min(n, m);
    pow2[0] = 1;
    for (int k = 1; k < 10; k++) {
        pow2[k] = pow2[k - 1] * 2;
    }
    int log2len = log2(len) + 1;
    for (int k = 1; k <= log2len; k++) {
        for (int i = 0; i + pow2[k] <= n; i++) {
            for (int j = 0; j + pow2[k] <= m; j++) {
                st[i][j][k] = qmin(st[i][j][k - 1],
                                   st[i][j + pow2[k - 1]][k - 1],
                                   st[i + pow2[k - 1]][j][k - 1],
                                   st[i + pow2[k - 1]][j + pow2[k - 1]][k - 1]);
                stGCD[i][j][k] = qgcd(stGCD[i][j][k - 1],
                                      stGCD[i][j + pow2[k - 1]][k - 1],
                                      stGCD[i + pow2[k - 1]][j][k - 1],
                                      stGCD[i + pow2[k - 1]][j + pow2[k - 1]][k - 1]);
            }
        }
    }
    // 查询
    int ans = 0;
    int count = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 1; j < m; j++) {
            // 确定可以取的范围
            int maxLen = qmin(i, n - i - 1, j, m - j - 1);
            // 二分查找
            int l = 1, r = maxLen + 1;
            while (l < r) {
                int mid = (l + r) / 2;
                if (a[i][j] == queryMin(i, j, mid) && a[i][j] == queryGCD(i, j, mid)) {
                    l = mid + 1;
                } else {
                    r = mid;
                }
            }
            if (l == 1) {
                continue;
            }
            if (ans == l - 1) {
                // cout << i << "\t" << j << "\t" << l - 1 << endl;
                count += 1;
            } else if (l - 1 > ans) {
                // cout << i << "\t" << j << "\t" << l - 1 << endl;
                ans = l - 1;
                count = 1;
            }
        }
    }
    if (ans == 0) {
        cout << 1 << endl << n * m << endl;
        return 0;
    }
    cout << (ans * 2 + 1) * (ans * 2 + 1) << endl;
    cout << count << endl;
    return 0;
}
```

### 测试例生成（Python）

```Python
#!/bin/env python
import random


def e(n, m, max=1e9, filename='in'):
    a = []
    for i in range(n):
        a.append("\t".join([str(random.randint(1, max)) for j in range(m)]))
    f = open(filename, 'w')
    f.write("{} {}\r\n{}\r\n".format(n, m, "\r\n".join(a)))
    f.close()

```

### 暴力验证

这个其实不能算完全暴力，因为还是用了ST表，只是没有用二分查找。

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

int a[500][500];
int st[500][500][10];
int stGCD[500][500][10];

#define qmin(a, b, c, d) min(min(a,b),min(c,d))

int gcd(int x, int y) {
    if (x == 0) {
        return y;
    }
    return gcd(y % x, x);
}

#define qgcd(a, b, c, d) gcd(gcd(a, b), gcd(c, d))

int pow2[10];

int queryMin(int midX, int midY, int len) {
    int k = log2(len * 2 + 1);
    return qmin(st[midX - len][midY - len][k],
                st[midX - len][midY + len - pow2[k] + 1][k],
                st[midX + len - pow2[k] + 1][midY - len][k],
                st[midX + len - pow2[k] + 1][midY + len - pow2[k] + 1][k]);
}

int queryGCD(int midX, int midY, int len) {
    int k = log2(len * 2 + 1);
    return qmin(stGCD[midX - len][midY - len][k],
                stGCD[midX - len][midY + len - pow2[k] + 1][k],
                stGCD[midX + len - pow2[k] + 1][midY - len][k],
                stGCD[midX + len - pow2[k] + 1][midY + len - pow2[k] + 1][k]);
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n, m;
    cin >> n >> m;
    // n行m列
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> a[i][j];
            st[i][j][0] = a[i][j];
            stGCD[i][j][0] = a[i][j];
        }
    }
    // 预处理
    int len = min(n, m);
    pow2[0] = 1;
    for (int k = 1; k < 10; k++) {
        pow2[k] = pow2[k - 1] * 2;
    }
    int log2len = log2(len) + 1;
    for (int k = 1; k <= log2len; k++) {
        for (int i = 0; i + pow2[k] <= n; i++) {
            for (int j = 0; j + pow2[k] <= m; j++) {
                st[i][j][k] = qmin(st[i][j][k - 1],
                                   st[i][j + pow2[k - 1]][k - 1],
                                   st[i + pow2[k - 1]][j][k - 1],
                                   st[i + pow2[k - 1]][j + pow2[k - 1]][k - 1]);
                stGCD[i][j][k] = qgcd(stGCD[i][j][k - 1],
                                      stGCD[i][j + pow2[k - 1]][k - 1],
                                      stGCD[i + pow2[k - 1]][j][k - 1],
                                      stGCD[i + pow2[k - 1]][j + pow2[k - 1]][k - 1]);
            }
        }
    }
    // 查询
    int ans = 0;
    int count = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 1; j < m; j++) {
            // 确定可以取的范围
            int maxLen = qmin(i, n - i - 1, j, m - j - 1);
            // 暴力查找
            for (int l = maxLen; l >= 1; l--) {
                if (a[i][j] == queryMin(i, j, l) && a[i][j] == queryGCD(j, j, l)) {
                    if (ans == l) {
                        // cout << i << "\t" << j << "\t" << l << endl;
                        count += 1;
                    } else if (l > ans) {
                        // cout << i << "\t" << j << "\t" << l << endl;
                        ans = l;
                        count = 1;
                    }
                    break;
                }
            }
        }
    }
    if (ans == 0) {
        cout << 1 << endl << n * m << endl;
        return 0;
    }
    cout << (ans * 2 + 1) * (ans * 2 + 1) << endl;
    cout << count << endl;
    return 0;
}
```

