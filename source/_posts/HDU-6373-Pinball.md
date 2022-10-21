---
title: HDU 6373 Pinball
tags: 
  - 刷题
  - ICPC
  - 物理
categories:
  - ICPC训练
  - 数学与几何
  - 物理题
abbrlink: 20160
date: 2018-09-28 11:03:58
---

嘛，做科协的新生赛题目碰到了，这里记录一下思路。题目如下

```
Pinball

Time Limit: 2000/1000 MS (Java/Others)    Memory Limit: 262144/262144 K (Java/Others)
Total Submission(s): 574    Accepted Submission(s): 249

 

Problem Description

There is a slope on the 2D plane. The lowest point of the slope is at the origin. There is a small ball falling down above the slope. Your task is to find how many times the ball has been bounced on the slope.

It's guarantee that the ball will not reach the slope or ground or Y-axis with a distance of less than 1 from the origin. And the ball is elastic collision without energy loss. Gravity acceleration g=9.8m/s2.






 

Input

There are multiple test cases. The first line of input contains an integer T (1 ≤ T ≤ 100), indicating the number of test cases.

The first line of each test case contains four integers a, b, x, y (1 ≤ a, b, -x, y ≤ 100), indicate that the slope will pass through the point(-a, b), the initial position of the ball is (x, y).




 

Output

Output the answer.

It's guarantee that the answer will not exceed 50.




 

Sample Input

1

5 1 -5 3




Sample Output

2
```

 这题基本上是物理题，我看了大多数人的写法，大都是计算每次跳跃的距离然后判断。这题数据算是比较小的，这样当然要过也很容易，这里给出一个另一个思路。按照沿斜面方向和垂直斜面方向分解之后，沿斜面方向是典型的匀加速直线运动，直接求出距离然后计算总时间（二次方程求根），而每次跳跃的时间都是固定的，所以总时间再除以每次跳跃的时间，结果就出来了。这样非常快，时间空间复杂度都是常数，在vj平台上显示用时为0。

AC代码如下

```
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
#include<map>
#include<cmath>
#include<iomanip>
#define ll long long
#define ld long double

using namespace std;

const ll tests[][4] = {
	{5,1,-5,3},
	{99,17,-97,25}
};

int main() {
	std::ios::sync_with_stdio(false);
	cin.tie(NULL), cout.tie(NULL);

	ll a, b, x, y;
	const ld pi = acos(-1);
	const ld pi2 = pi / 2;
	const ld g = 9.8;

	ld v0, ax, ay, vx, vy, tan0, sin0, cos0, h0, l0, tt, t0;

	ll T;
	cin >> T;
	while (T--) {
		cin >> a >> b >> x >> y;
		
		tan0 = (ld)b / a;
		h0 = y - (-x)*tan0;
		v0 = sqrt(2 * g*h0);
		sin0 = sqrt(1 - 1 / (tan0*tan0 + 1));
		cos0 = sqrt(1 / (tan0*tan0 + 1));
		l0 = (-x) / cos0;//总长度
		vx = v0 * sin0;
		vy = v0 * cos0;
		ax = g * sin0;
		ay = g * cos0;
		t0 = 2 * vy / ay;//单次跳跃时间
		
		//解方程，1/2*ax*t*t+vx*t=l0
		tt = (sqrt(vx*vx + 2 * ax*l0) - vx) / ax;
		cout << (ll)(tt / t0) + 1 << endl;
	}
	return 0;
}
```
