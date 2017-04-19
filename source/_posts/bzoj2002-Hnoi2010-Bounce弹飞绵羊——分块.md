---
title: '[bzoj2002][Hnoi2010]Bounce弹飞绵羊——分块'
tags: [数据结构, 分块, lct]
date: 2017-03-09 07:55:00
---

#Brief description
某天，Lostmonkey发明了一种超级弹力装置，为了在他的绵羊朋友面前显摆，他邀请小绵羊一起玩个游戏。游戏一开始，Lostmonkey在地上沿着一条直线摆上n个装置，每个装置设定初始弹力系数ki，当绵羊达到第i个装置时，它会往后弹ki步，达到第i+ki个装置，若不存在第i+ki个装置，则绵羊被弹飞。绵羊想知道当它从第i个装置起步时，被弹几次后会被弹飞。为了使得游戏更有趣，Lostmonkey可以修改某个弹力装置的弹力系数，任何时候弹力系数均为正整数。

#Algorithm design 
因为太弱不会用lct，所以我们使用分块做Orz
考察暴力解法：
第一种暴力做法就是修改的时候直接$\Theta(1)$修改，查询的时候要$\Theta(n)$查询。
第二种暴力做法就是修改的时候把所有的答案重新递推出来，复杂度$\Theta(n)$，查询的时候就可以$\Theta(1)$查询了。
我们考虑分块把两种方法结合起来。设分块的大小为$h(n)$。
首先我们预处理出来每个点跳出当前块需要多少步，并且跳出之后会落在哪里。
考虑修改。对于每个点，如果我们修改，会导致所有可以到达它的点的答案变化。我们只修改块内的答案，不难证明这样做是可行的。复杂度$\Theta(h(n))$
考虑查询。根据预处理信息，我们可以方便的查询。

#Code
```cpp
#include <cmath>
#include <cstdio>
#include <cstring>
const int maxn = 200100;
const int inf = 0x3f3f3f;
int n, m, f[maxn], g[maxn], h[maxn], k[maxn], block;
int id(int x) {
  if (block != 0)
    return (x - 1) / block + 1;
  return 1;
}
int main(int argc, char const *argv[]) {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d", &n);
  block = (int)sqrt(n);
  memset(f, -1, sizeof(f));
  for (int i = 1; i <= n; i++) {
    int x;
    scanf("%d", &x);
    f[i] = i + x;
    if (f[i] > n)
      f[i] = -1;
    k[i] = -1;
    h[i] = -1;
    g[i] = inf;
  }
  k[n + 1] = 0;
  for (int i = n; i >= 1; i--) {
    if (f[i] != -1) {
      k[i] = k[f[i]] + 1;
      if (id(i) == id(f[i])) {
        g[i] = g[f[i]] + 1;
        h[i] = h[f[i]];
      } else {
        g[i] = 1;
        h[i] = f[i];
      }
    } else {
      k[i] = 1;
      g[i] = 1;
      h[i] = -1;
    }
  }
  scanf("%d", &m);
  while (m--) {
    int opt;
    scanf("%d", &opt);
    if (opt == 1) {
      if (block == 0) {
        printf("0\n");
        continue;
      }
      int x;
      scanf("%d", &x);
      x++;
      int ans = 1;
      while (f[x] != -1) {
        if (h[x] != -1) {
          ans += g[x];
          x = h[x];
        } else {
          x = f[x];
          ans++;
        }
      }
      printf("%d\n", ans);
    }
    if (opt == 2) {
      if (block == 0)
        continue;
      int x, y;
      scanf("%d %d", &x, &y);
      x++;
      f[x] = x + y;
      k[x] = k[y] + 1;
      if (f[x] > n)
        f[x] = -1;
      if (id(y) == id(x))
        g[x] = g[y] + 1, h[x] = h[y];
      else
        g[x] = y, h[x] = 1;
      for (int i = x; id(i) == id(x) && i; i--) {
        if (f[i] != -1) {
          k[i] = k[f[i]] + 1;
          if (id(i) == id(f[i])) {
            g[i] = g[f[i]] + 1;
            h[i] = h[f[i]];
          } else {
            g[i] = 1;
            h[i] = f[i];
          }
        } else {
          k[i] = 1;
          g[i] = 1;
          h[i] = -1;
        }
      }
    }
  }
  return 0;
}
```