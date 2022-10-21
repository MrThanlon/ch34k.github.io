---
title: UESTC 2459
tags:
  - KMP
  - 字符串
  - 刷题
categories:
  - ICPC训练
  - 字符串与搜索
  - KMP
abbrlink: 11866
date: 2020-06-20 11:00:59
---


## 题目

给两段字符串$A$和$B$，输出$A$中所有出现$B$的位置。

## 题解

这题是老KMP了，不过还是说下怎么做吧<del>不然太水了</del>。

<!--more-->

其实KMP可以看成是一个状态机，按照模式串（也就是要匹配的字符串，这里是$B$）建立状态转移，字符串作为输入。

理论上来说一个状态应该有多个转移方式，由当前状态（也就是匹配到的位置）和下一个字符来决定转移到哪个状态，但是为了方便，我们只取两种，即匹配成功和匹配失败，这样的简化与前面的那个方式基本上是等价的，这也就是`next`数组的由来，`next[i]`表示在第`i`位失配时模式串指针应该回退到的位置，而`next`数组本身的含义是“字符串的前缀与后缀相同的最大长度”，其中“前缀”指除了最后一个字符以外，一个字符串的全部头部组合，“后缀”指除了第一个字符以外，一个字符串的全部尾部组合。

另外一点，与普通的查找字符串不同的是，这题要求的是所有出现的位置，所以对于一个长度为$n$的模式串，在计算`next`数组的时候需要计算到第$n$位，也就是说在完全匹配（也就是模式串指针到达尾部）的时候模式串指针应该进行同样的跳转而不是回到0。

说实话对于KMP还是有一点迷糊，虽然知道它的原理，但总是觉得自己并没有真正理解。

## 代码

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

const int MAX = 1e6 + 7;

int nextt[MAX];

void getNext(string &p) {
    int len = p.length();
    nextt[0] = -1;
    int k = -1;
    int j = 0;
    while (j < len) {
        //p[k]表示前缀，p[j]表示后缀（并没有“真”！）
        if (k == -1 || p[j] == p[k])//j在这
        {
            j++;//j+1在这
            k++;//k=k+1
            //---->若p[k]=p[j]，则next[j+1]=next[j]+1;
            //下一个位置的next
            if (p[j] != p[k])//当p[k]！=p[j]时，与主串不匹配时可以返回到这
            {
                nextt[j] = k;
            } else//当p[j]=p[k]时与主串匹配时你在j位置不匹配匹配串返回这里当前
                //还是 不能与主串匹配，然后还是要返回next[]即next[next[k]]
                nextt[j] = nextt[k];//优化了
        } else {
            k = nextt[k];
        }
    }
}

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    string a, b;
    getline(cin, a);
    getline(cin, b);
    getNext(b);
    int j = 0;
    for (int i = 0; i < a.length();) {
        if (j == -1 || a[i] == b[j]) {
            j++;
            i++;
        } else {
            j = nextt[j];
        }
        if (j == b.length()) {
            // 匹配成功
            cout << i + 1 - j << " ";
            j = nextt[j];
        }
    }
    return 0;
}
```

### 测试例生成（Python）

```Python
def A(lenA, lenB, r=50, filename='in'):
    asciiSet = range(32, 126 + 1)
    a = "".join([chr(asciiSet[random.randint(32, 32 + r) - 32]) for x in range(lenA)])
    b = "".join([chr(asciiSet[random.randint(32, 32 + r) - 32]) for x in range(lenB)])
    f = open(filename, 'w')
    f.write(a)
    f.write("\n")
    f.write(b)
    f.close()
```

### 暴力验证

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

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    string a, b;
    getline(cin, a);
    getline(cin, b);
    for (int i = 0; i <= (int)(a.length() - b.length()); i++) {
        int c = 0, j = 0;
        while (j < b.length() && a[c + i] == b[j]) {
            j += 1;
            c += 1;
        }
        if (j == b.length()) {
            cout << i + 1 << " ";
        }
    }
    return 0;
}
```
