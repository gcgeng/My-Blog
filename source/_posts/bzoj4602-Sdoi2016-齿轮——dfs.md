---
title: '[bzoj4602][Sdoi2016]齿轮——dfs'
tags: [搜索]
date: 2017-02-24 07:03:00
---

# 题目
现有一个传动系统，包含了N个组合齿轮和M个链条。每一个链条连接了两个组合齿轮u和v，并提供了一个传动比x 
: y。即如果只考虑这两个组合齿轮，编号为u的齿轮转动x圈，编号为v的齿轮会转动y圈。传动比为正表示若编号
为u的齿轮顺时针转动，则编号为v的齿轮也顺时针转动。传动比为负表示若编号为u的齿轮顺时针转动，则编号为v
的齿轮会逆时针转动。若不同链条的传动比不相容，则有些齿轮无法转动。我们希望知道，系统中的这Ｎ个组合齿
轮能否同时转动。
# 题解：
没想到直接dfs居然能过。
doc出的题真的是。。。水题水的不敢想象，难题难得不敢想象。。。
# 代码
```
#include <bits/stdc++.h>
using namespace std;
#define eps 1e-8
struct edge {
  int to;
  double w;
};
const int maxn = 2000;
double f[maxn];
int vis[maxn];
vector<edge> G[maxn];
void add_edge(int u, int v, double w) { G[u].push_back((edge){v, w}); }
int n, m, t, kase = 0;
bool dfs(int x) {
  vis[x] = 1;
  for (int i = 0; i < G[x].size(); i++) {
    edge &e = G[x][i];
    if (vis[e.to]) {
      if (fabs(f[e.to] - e.w * f[x]) > eps)
        return false;
    } else {
      f[e.to] = f[x] * e.w;
      if (!dfs(e.to))
        return false;
    }
  }
  return true;
}
int main() {
  // freopen("input", "r", stdin);
  scanf("%d", &t);
  while (t--) {
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= n; i++)
      G[i].clear();
    for (int i = 1; i <= n; i++)
      f[i] = 0;
    for (int i = 1; i <= m; i++) {
      int u, v;
      long long x, y;
      scanf("%d %d %lld %lld", &u, &v, &x, &y);
      double d = (double)y / (double)x;
      add_edge(u, v, d);
      add_edge(v, u, 1 / d); //注意要连一条反向边，这样才能做到一整个连通块在一次赋值内全部dfs到。
    }
    memset(vis, 0, sizeof(vis));
    int flag = 1;
    for (int i = 1; i <= n; i++) {
      if (!vis[i]) {
        f[i] = 1;
        if (!dfs(i)) {
          flag = 0;
          break;
        }
      }
    }
    if (flag) {
      printf("Case #%d: Yes\n", ++kase);
    } else {
      printf("Case #%d: No\n", ++kase);
    }
  }
  return 0;
}
```