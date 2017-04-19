---
title: '[bzoj1977][BeiJing2010组队]次小生成树 Tree——树上倍增+lca'
tags: [数据结构, 图论, 生成树, 树上倍增, lca]
date: 2017-03-07 10:32:00
---

#Brief Description
求一个无向图的*严格*次小生成树。

#Algorithm Design
考察最小生成树的生成过程。对于一个非树边而言，如果我们使用这一条非树边去替换原MST的路径上的最大边，可以证明仍然满足生成树性质，而且这个生成树的大小一定不小于原生成树，那么枚举所有这样的非树边，尝试去替换，找到最小值就可以了。
那么问题就转化成了求树上两个点的最大/最小距离，这是树上倍增的经典应用，可以知道：
$$Max(x,i) = max(Max(x,i-1), Max(fa(x,i-1),i-1))$$
我们可以通过递推用$\Theta(nlogn)$时间预处理出来这些信息。
问题解决了吗？还没有。原题要求严格小于原树。所以我们顺便预处理一下树上两个点的次大值就可以了。

#Code
```cpp
#include <algorithm>
#include <cstdio>
using std::max;
using std::min;
const int maxn = 100001;
const int maxm = 300001;
const int inf = 0x3f3f3f;
#define ll long long
int n, m, tot, cnt, mn = inf;
ll ans;
int f[maxn], head[maxn], deep[maxn], fa[maxn][17], d1[maxn][17], d2[maxn][17];
struct data {
  int x, y, v;
  bool sel;
  bool operator<(const data b) const { return this->v < b.v; }
} a[maxm];
struct edge {
  int to, next, v;
} e[maxn << 1];
void ins(int u, int v, int w) {
  e[++cnt].to = v;
  e[cnt].next = head[u];
  e[cnt].v = w;
  head[u] = cnt;
}
int find(int x) { return f[x] == x ? x : f[x] = find(f[x]); }
void dfs(int x, int f) {
  for (int i = 1; i <= 16; i++) {
    if (deep[x] < (1 << i))
      break;
    fa[x][i] = fa[fa[x][i - 1]][i - 1];
    d1[x][i] = max(d1[x][i - 1], d1[fa[x][i - 1]][i - 1]);
    if (d1[x][i - 1] == d1[fa[x][i - 1]][i - 1])
      d2[x][i] = max(d2[x][i - 1], d2[fa[x][i - 1]][i - 1]);
    else {
      d2[x][i] = min(d1[x][i - 1], d1[fa[x][i - 1]][i - 1]);
      d2[x][i] = max(d2[x][i], d2[x][i - 1]);
      d2[x][i] = max(d2[x][i], d2[fa[x][i - 1]][i - 1]);
    }
  }
  for (int i = head[x]; i; i = e[i].next) {
    if (e[i].to != f) {
      fa[e[i].to][0] = x;
      d1[e[i].to][0] = e[i].v;
      deep[e[i].to] = deep[x] + 1;
      dfs(e[i].to, x);
    }
  }
}
int lca(int x, int y) {
  if (deep[x] < deep[y])
    std::swap(x, y);
  int t = deep[x] - deep[y];
  for (int i = 0; i <= 16; i++)
    if (t & (1 << i))
      x = fa[x][i];
  for (int i = 16; i >= 0; i--)
    if (fa[x][i] != fa[y][i])
      x = fa[x][i], y = fa[y][i];
  if (x == y)
    return x;
  return fa[x][0];
}
void cal(int x, int f, int v) {
  int mx1 = 0, mx2 = 0;
  int t = deep[x] - deep[f];
  for (int i = 0; i <= 16; i++) {
    if (t & (1 << i)) {
      if (d1[x][i] > mx1) {
        mx2 = mx1;
        mx1 = d1[x][i];
      }
      mx2 = max(mx2, d2[x][i]);
      x = fa[x][i];
    }
  }
  if (mx1 != v)
    mn = min(mn, v - mx1);
  else
    mn = min(mn, v - mx2);
}
void solve(int t, int v) {
  int x = a[t].x, y = a[t].y, f = lca(x, y);
  cal(x, f, v);
  cal(y, f, v);
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++)
    f[i] = i;
  for (int i = 1; i <= m; i++)
    scanf("%d %d %d", &a[i].x, &a[i].y, &a[i].v);
  std::sort(a + 1, a + m + 1);
  for (int i = 1; i <= m; i++) {
    int p = find(a[i].x), q = find(a[i].y);
    if (p != q) {
      f[p] = q;
      ans += a[i].v;
      a[i].sel = 1;
      ins(a[i].x, a[i].y, a[i].v);
      ins(a[i].y, a[i].x, a[i].v);
      tot++;
      if (tot == n - 1)
        break;
    }
  }
  dfs(1, 0);
  for (int i = 1; i <= m; i++)
    if (!a[i].sel)
      solve(i, a[i].v);
  printf("%lld\n", ans + mn);
}
```