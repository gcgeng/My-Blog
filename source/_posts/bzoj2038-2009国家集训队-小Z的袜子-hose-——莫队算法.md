---
title: '[bzoj2038][2009国家集训队]小Z的袜子(hose)——莫队算法'
tags: [数据结构, 莫队算法]
date: 2017-03-10 21:21:00
---

#Brief Description
给定一个序列，您需要处理m个询问，每个询问形如[l,r]，您需要回答在区间[l,r]中任意选取两个数相同的概率。

#Algorithm Design
莫队算法入门题目。
[这篇博客讲的不错](http://blog.csdn.net/bossup/article/details/39236275)
对于L,R的询问。设袜子的个数为$cnt_i$
那么答案即为$$\frac{\sum_i C_{cnt}^2}{\frac{(R-L+1)*(R-L)}{2}}$$
其中$C_n^m$为组合数。
化简得:$$\frac{\sum_i cnt^2 -(R-L+1)}{(R-L+1)*(R-L)}$$
所以这道题目的关键是求一个区间内每种颜色数目的平方和。
如果你知道了[L,R]的答案。你可以在O(1)的时间下得到[L,R-1]和[L,R+1]和[L-1,R]和[L+1,R]的答案的话。就可以使用莫队算法。
对于莫队算法我感觉就是暴力。只是预先知道了所有的询问。可以合理的组织计算每个询问的顺序以此来降低复杂度。要知道我们算完[L,R]的答案后现在要算[L',R']的答案。由于可以在O(1)的时间下得到[L,R-1]和[L,R+1]和[L-1,R]和[L+1,R]的答案.所以计算[L',R']的答案花的时间为|L-L'|+|R-R'|。如果把询问[L,R]看做平面上的点a(L,R).询问[L',R']看做点b(L',R')的话。那么时间开销就为两点的曼哈顿距离。所以对于每个询问看做一个点。我们要按一定顺序计算每个值。那开销就为曼哈顿距离的和。要计算到每个点。那么路径至少是一棵树。所以问题就变成了求二维平面的最小曼哈顿距离生成树。
关于二维平面最小曼哈顿距离生成树。感兴趣的可以参考[胡泽聪大佬的这篇文章](http://blog.csdn.net/huzecong/article/details/8576908)

这样只要顺着树边计算一次就ok了。可以证明时间复杂度为$n* \sqrt n$。

但是这种方法编程复杂度稍微高了一点。所以有一个比较优雅的替代品。那就是先对序列分块。然后对于所有询问按照L所在块的大小排序。如果一样再按照R排序。然后按照排序后的顺序计算。为什么这样计算就可以降低复杂度呢。

一、i与i+1在同一块内，r单调递增，所以r是O(n)的。由于有n^0.5块,所以这一部分时间复杂度是n^1.5。
二、i与i+1跨越一块，r最多变化n，由于有n^0.5块，所以这一部分时间复杂度是n^1.5
三、i与i+1在同一块内时变化不超过n^0.5，跨越一块也不会超过2*n^0.5，不妨看作是n^0.5。由于有n个数，所以时间复杂度是n^1.5
于是就变成了$\Theta(n^{1.5})$了。

#Code
```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <iostream>
const int maxn = 50010;
#define ll long long
ll num[maxn], up[maxn], dw[maxn], ans, aa, bb, cc;
int col[maxn], pos[maxn];
struct qnode {
  int l, r, id;
} qu[maxn];
bool cmp(qnode a, qnode b) {
  if (pos[a.l] == pos[b.l])
    return a.r < b.r;
  else
    return pos[a.l] < pos[b.l];
}
ll gcd(ll x, ll y) {
  ll tp;
  while ((tp = x % y)) {
    x = y;
    y = tp;
  }
  return y;
}
void update(int x, int d) {
  ans -= num[col[x]] * num[col[x]];
  num[col[x]] += d;
  ans += num[col[x]] * num[col[x]];
}
int main() {
  int n, m, bk, pl, pr, id;
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  memset(num, 0, sizeof(num));
  bk = ceil(sqrt(1.0 * n));
  for (int i = 1; i <= n; i++) {
    scanf("%d", &col[i]);
    pos[i] = (i - 1) / bk;
  }
  for (int i = 0; i < m; i++) {
    scanf("%d %d", &qu[i].l, &qu[i].r);
    qu[i].id = i;
  }
  std::sort(qu, qu + m, cmp);
  pl = 1, pr = 0;
  ans = 0;
  for (int i = 0; i < m; i++) {
    id = qu[i].id;
    if (qu[i].l == qu[i].r) {
      up[id] = 0, dw[id] = 1;
      continue;
    }
    if (pr < qu[i].r) {
      for (int j = pr + 1; j <= qu[i].r; j++)
        update(j, 1);
    } else {
      for (int j = pr; j > qu[i].r; j--)
        update(j, -1);
    }
    pr = qu[i].r;
    if (pl < qu[i].l) {
      for (int j = pl; j < qu[i].l; j++)
        update(j, -1);
    } else {
      for (int j = pl - 1; j >= qu[i].l; j--)
        update(j, 1);
    }
    pl = qu[i].l;
    aa = ans - qu[i].r + qu[i].l - 1;
    bb = (ll)(qu[i].r - qu[i].l + 1) * (qu[i].r - qu[i].l);
    cc = gcd(aa, bb);
    aa /= cc, bb /= cc;
    up[id] = aa, dw[id] = bb;
  }
  for (int i = 0; i < m; i++)
    printf("%lld/%lld\n", up[i], dw[i]);
}
```