---
title: UESTC 2360
tags:
  - Stack
  - 数据结构
  - 刷题
  - 括号匹配
categories:
  - ICPC训练
  - 数据结构
  - 栈
abbrlink: 6443
date: 2020-04-25 18:35:44
---


> 关键字：括号匹配；栈；

## 题解

括号匹配，直接上栈就完事了。首先判断输入的符号是`(`还是`)`，如果是前者则入栈，后者先判断栈是否为空，空则`ans+=1`，不空的话出栈，最后栈内剩余的是没有被匹配的，所以答案是`ans`加上栈内元素数量。时间复杂度$o(n)$。

## 代码

### AC代码

```C++
// @author: hzy
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

int main() {
    std::ios::sync_with_stdio(false);
    int t;
    cin >> t;
    stack<char> st;
    while (t--) {
        int n;
        int ans = 0;
        cin >> n;
        char c;
        while (n--) {
            cin >> c;
            if (c == '(') {
                st.push(c);
            } else {
                // ) matches a (
                if (st.empty()) {
                    ans += 1;
                } else {
                    st.pop();
                }
            }
        }
        cout << ans + st.size() << endl;
        while (!st.empty()){
            st.pop();
        }
    }
    return 0;
}

```

## 后记

其实不用STL的`stack`也可以，只需要记录有多少个`(`，应该会更快，毕竟栈内只会有着一种符号，我以为这样写会TLE，没想到居然过了，可能是训练题比较水吧。如果是多种括号的匹配，比如`()[]{}`的组合，那应该只能用栈。
