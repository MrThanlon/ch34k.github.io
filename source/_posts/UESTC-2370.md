---
layout: uestc 2362
title: UESTC 2370
tags:
  - Stack
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - 栈
abbrlink: 35114
date: 2020-05-05 18:12:06
---


> 关键字：栈；贪心；

[leetcode 1081](https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters/)差不多，但是有一点区别。这题直接给出了元素的种类，也就是$k$，而leetcode上那题并不直接给出元素种类，但最多是26（小写字母），而这题的$k$高达1e6...

leetcode社区中的题解主要有两种方法：

- 栈+贪心
- 寻找第一个元素的位置+贪心

我在写本题的时候用的第一个方法，第二个应该也可以，但是有点麻烦，它需要$o(1)$的复杂度确定从$i$到最后共有多少种元素，leetcode那题因为最多26种，所以直接位运算就完事，而本题的$k$高达1e6，位运算显然不现实，也可以用前缀和去判断，但是我懒得写，直接第一种完事。

## 题解

参照[这个题解](https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters/solution/tan-xin-suan-fa-zhan-wei-yan-ma-python-dai-ma-java/)。

上面链接的那个幻灯片就很直观了。首先从输入读取一个数字，如果栈是空的则入栈，如果非空则判断能否进入，首先先确定栈内是否存在这个数字，我的做法是用一个`marked`数组保存已经入栈的数字，如果存在则不处理，不存在则进行下一步，判断栈顶元素是否大于输入的数，如果大于而且栈顶元素还存在于之后的输入中，则出栈并重复这一步骤，直到栈为空或者栈顶小于输入的数，然后入栈，此时就能保证字典序是最小的。

要判断栈顶元素是否还存在于之后的输入中，我的做法是保存每个数字最后出现的位置，这样直接判断当前位置是否小于最后的位置即可。

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
const int MAX = 1e6 + 5;
int lastPos[MAX];
int seq[MAX];
int origin[MAX];
bool marked[MAX];

int main() {
    //std::ios::sync_with_stdio(false);
    //std::cin.tie(0);
    int n;
    int k;
    int tmp;
    cin >> n >> k;
    for (int i = 1; i <= n; i++) {
        // cin >> tmp;
        scanf("%d", &tmp);
        origin[i] = tmp;
        lastPos[tmp] = i;
    }
    // stack
    int stackIter = 1;
    for (int i = 1; i <= n; i++) {
        if (stackIter == 1) {
            // insert anyway
            seq[stackIter] = origin[i];
            marked[origin[i]] = true;
            stackIter += 1;
        } else {
            if (seq[stackIter - 1] == origin[i] || marked[origin[i]]) {
                continue;
            }
            while (true) {
                if (stackIter != 1 && seq[stackIter - 1] >= origin[i] && i < lastPos[seq[stackIter - 1]]) {
                    // pop
                    marked[seq[stackIter - 1]] = false;
                    stackIter -= 1;
                } else {
                    // insert
                    seq[stackIter] = origin[i];
                    marked[origin[i]] = true;
                    stackIter += 1;
                    break;
                }
            }
        }
    }
    if (stackIter <= k) {
        cout << "Kanade";
        return 0;
    }
    for (int i = 1; i < stackIter; i++) {
        //cout << seq[i] << " ";
        printf("%d ", seq[i]);
    }
    return 0;
}

```

### 测试例生成（Python3）

```Python
import random
def k(n,k):
     a = [str(random.randint(1, k)) for i in range(n)]
     f = open('in', 'w')
     f.write("{} {}\r\n{}\r\n".format(n, k, " ".join(a)))
     f.close()
```

## 彩蛋

吐个槽，我在本机上测试1e6的数据量，默认编译器是clang++ 11.0.3 with LLVM，跑了两秒多，换成g++9瞬间提升到0.08秒，绝了。

破案了，macOS上的clang++默认使用libc++，这个库无法通过一般方法关闭流同步，而g++使用的是libstdc++，就没有这个问题。
