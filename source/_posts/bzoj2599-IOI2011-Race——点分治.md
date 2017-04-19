---
title: '[bzoj2599][IOI2011]Race——点分治'
tags: [点分治]
date: 2017-03-05 16:44:00
---

# Brief Description
给定一棵带权树，你需要找到一个点对，他们之间的距离为k，且路径中间的边的个数最少。
# Algorithm Analyse
我们考虑点分治。
对于子树，我们递归处理，所以我们只考察经过重心的情况。
我们很容易把所有点的dist和deep预处理出来，所以，问题就转化成了求一个点对，使得
$dist[i]+dist[j] = K\ and\ Belong[i] \neq Belong[j]$
开始的时候，我想：这题我做过的！不就是在数列里面设两个指针吗？
然后，我就发现这个方法行不通。因为有重复元素，而这些元素全部需要保留。最坏情况下，仍然是$\Theta(n^2)$的复杂度来枚举。
回去看题，发现K的范围还是比较小的。我们开设一个桶记录每个值的最优方案，这样就可以了。具体方法参见[hzwer](http://hzwer.com/4286.html)。
# Code
```c++
//[bzoj2599][IOI2011]Race
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <set>
#include <vector>
#ifndef ONLINE_JUDGE
const int maxn = 200;
#else
const int maxn = 200050;
#endif
using std::max;
using std::vector;
using std::min;
int n, m, k, rt, ans, sum;
int tot = 0, deep[maxn], dep[maxn], vis[maxn], size[maxn], f[maxn], t[1000005];
struct edge {
  int to, v;
};
vector<edge> G[maxn];
void add_edge(int u, int v, int w) {
  G[u].push_back((edge){v, w});
  G[v].push_back((edge){u, w});
}
void getroot(int x, int fa) {
  size[x] = 1;
  f[x] = 0;
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to] && e.to != fa) {
      getroot(e.to, x);
      size[x] += size[e.to];
      f[x] = max(f[x], size[e.to]);
    }
  }
  f[x] = max(f[x], sum - size[x]);
  if (f[x] < f[rt])
    rt = x;
}
void getdeep(int x, int fa) {
  if (deep[x] <= k)
    ans = min(ans, dep[x] + t[k - deep[x]]);
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to] && e.to != fa) {
      dep[e.to] = dep[x] + 1;
      deep[e.to] = deep[x] + e.v;
      getdeep(e.to, x);
    }
  }
}
void update(int x, int fa, bool flag) {
  if (deep[x] <= k) {
    if (flag)
      t[deep[x]] = min(t[deep[x]], dep[x]);
    else
      t[deep[x]] = 0x3f3f3f;
  }
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to] && e.to != fa)
      update(e.to, x, flag);
  }
}
void work(int x) {
  vis[x] = 1;
  t[0] = 0;
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to]) {
      dep[e.to] = 1;
      deep[e.to] = e.v;
      getdeep(e.to, 0);
      update(e.to, 0, 1);
    }
  }
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to])
      update(e.to, 0, 0);
  }
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (!vis[e.to]) {
      rt = 0;
      sum = size[e.to];
      getroot(e.to, 0);
      work(rt);
    }
  }
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &k);
  for (int i = 1; i < n; i++) {
    int x, y, z;
    scanf("%d %d %d", &x, &y, &z);
    x++, y++;
    add_edge(x, y, z);
  }
  f[0] = sum = ans = n;
  memset(t, 0x3f, sizeof(t));
  getroot(1, 0);
  work(rt);
  printf("%d\n", ans == n ? -1 : ans);
}
```