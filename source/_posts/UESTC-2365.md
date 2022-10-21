---
title: UESTC 2365
tags:
  - set
  - 数据结构
  - 刷题
categories:
  - ICPC训练
  - 数据结构
  - set
abbrlink: 6891
date: 2020-05-05 18:13:16
---


## 题目

你带着大量佣兵回到城市，看到了大量的新赏金任务。你无视其他等级的任务，拿过最高等级的任务列表，发现其任务数量刚好和你的佣兵数量相同，便把它们都接到了自己手上。

每个佣兵都有自己的能力值，每个任务都有能力值要求，能力值大于要求即可完成任务。

你让佣兵们自行选择任务（每人一个），然而，一旦脱离你的命令，他们就无法做出恰当的决策：你看完他们的选择后，发现他们选择的任务既有重复，也有缺漏，还有不少佣兵选择了自己能力不够、无法完成的任务。但你既然已经下令，朝令夕改会影响你的威信。

因此，你决定让他们按照自己的选择从你的手中接任务，如果选择的任务已经被接，则按照任务列表接下一个任务，如果还是被接则继续往下直至接到任务，如果到了最后一个还需继续往下，则从列表的第一个任务开始。（可以把列表理解为环状。列表顺序是既定的，你不可以修改）

不过，还有一件你可以控制的事：你可以决定他们谁先接任务。你需要找到一个最优的顺序，让佣兵完成最多的任务。

请计算最优顺序下可完成的任务数量。

（不知道怎么概括，以上为直接复制粘贴）

## 题解

按照出题人的题解。

首先要保证所有任务都不会被放弃，选择合适的起点即可。首先将每个任务被选中的人数减1求前缀和，找到最小的位置，从该位置的下一个位置作为起点即可保证所有任务都会有人接。

然后遍历每一个任务，查找可以选的人中有没有刚好大于任务需求的，有就让他接，没有就让能力最低的接，二分查找即可。维护最小值可以用`std::set`，这个容器的第一个元素就是最小的。

## 代码

因为并没有发生WA，所以就懒得写暴力验证程序了。

顺便`std::set`有一个问题，就是容器为空的时候如果调用`erase`删除元素会发生死循环，而不是RE，到OJ上的表现就是TLE，搞得我老在查是不是常数太大了。

### AC代码如下

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
const long long MAX = 5e5 + 7;

int choice[MAX];
int abilities[MAX];
int missionSelectedSum[MAX];
int missions[MAX];

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n;
    cin >> n;
    set<int> ability;
    unordered_set<int> missionSelected[n + 1];
    int tmp;
    // 找起点
    for (int i = 1; i <= n; i++) {
        cin >> choice[i];
    }
    for (int i = 1; i <= n; i++) {
        cin >> missions[i];
    }
    for (int i = 1; i <= n; i++) {
        cin >> tmp;
        missionSelected[choice[i]].insert(tmp);
    }
    int minSum = n + 1;
    missionSelectedSum[0] = 0;
    for (int i = 1; i <= n; i++) {
        missionSelectedSum[i] = missionSelectedSum[i - 1] + missionSelected[i].size() - 1;
    }
    // find the min sum
    int start = 1;
    for (int i = 1; i <= n; i++) {
        if (minSum > missionSelectedSum[i]) {
            start = i;
            minSum = missionSelectedSum[i];
        }
    }
    start += 1;
    // 开始了
    int ans = 0;
    for (int i = start; i <= n; i++) {
        for (auto it:missionSelected[i]) {
            ability.insert(it);
        }
        if (ability.empty()) {
            continue;
        }
        auto it = ability.lower_bound(missions[i]);
        if (it != ability.end()) {
            ans += 1;
            ability.erase(it);
        } else {
            ability.erase(ability.begin());
        }
    }
    for (int i = 1; i < start; i++) {
        for (auto it:missionSelected[i]) {
            ability.insert(it);
        }
        if (ability.empty()) {
            continue;
        }
        auto it = ability.lower_bound(missions[i]);
        if (it != ability.end()) {
            ans += 1;
            ability.erase(it);
        } else {
            ability.erase(ability.begin());
        }
    }
    cout << ans << endl;
    return 0;
}
```

### 测试例生成（Python）

这个程序其实不是很优，当`max`比较大的时候生成会很慢，主要是因为会生成长度为`max`的数组。如果不用这个方法的话得手写一个能产生唯一数字的序列，懒得再改了。

```Python
#!/bin/env python
import random


def f(n, max=1e9, filename='in'):
    n = int(n)
    max = int(max)
    choice = [str(random.randint(1, n)) for i in range(n)]
    mission = list(range(1, max))
    random.shuffle(mission)
    mission = list(map(lambda x: str(x), mission[: n]))
    ability = list(range(1, max))
    random.shuffle(ability)
    ability = list(map(lambda x: str(x), ability[: n]))
    fi = open(filename, 'w')
    fi.write("{}\r\n{}\r\n{}\r\n{}\r\n".format(
        n, " ".join(choice), " ".join(mission), " ".join(ability)
    ))
    fi.close()

```
