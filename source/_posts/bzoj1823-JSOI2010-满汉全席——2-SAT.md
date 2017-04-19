---
title: '[bzoj1823][JSOI2010]满汉全席——2-SAT'
tags: [图论, 2-SAT]
date: 2017-02-22 11:53:00
---

# 题目大意
题目又丑又长我就不贴了，说一下大意，有n种菜，m个评委，每一个评委又有两种喜好，每种菜有满汉两种做法，只能选一种。判断是否存在一种方案使得所有评委至少喜欢一种菜品。输入包含多组数据。
<!--more-->
# 题解
显然是2-SAT，注意两种不同做法的菜也要连边。
# 代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1000;
vector<int> G[maxn];
vector<int> rG[maxn];
vector<int> sc[maxn];
int vis[maxn], v, cnt[maxn];
vector<int> vs;
void add_edge(int from, int to) {
  G[from].push_back(to);
  rG[to].push_back(from);
}
void dfs(int u) {
  vis[u] = true;
  for (int i = 0; i < G[u].size(); i++) {
    if (!vis[G[u][i]])
      dfs(G[u][i]);
  }
  vs.push_back(u);
}
void rdfs(int v, int k) {
  vis[v] = true;
  cnt[v] = k;
  for (int i = 0; i < rG[v].size(); i++) {
    if (!vis[rG[v][i]])
      rdfs(rG[v][i], k);
  }
  vs.push_back(v);
  sc[k].push_back(v);
}
void scc() {
  memset(vis, 0, sizeof(vis));
  vs.clear();
  for (int i = 1; i < v; i++) {
    if (!vis[i])
      dfs(i);
  }
  memset(vis, 0, sizeof(vis));
  int k = 0;
  for (int i = vs.size() - 1; i >= 0; i--) {
    if (!vis[vs[i]]) {
      rdfs(vs[i], k++);
    }
  }
}
int n, m, T;
int main() {
  // freopen("input", "r", stdin);
  scanf("%d", &T);
  while (T--) {
    memset(cnt, 0, sizeof(cnt));
    vs.clear();
    for (int i = 0; i < maxn - 2; i++)
      sc[i].clear();
    scanf("%d %d", &n, &m);
    v = 4 * n + 1;
    for (int i = 1; i < v; i++) {
      G[i].clear(), rG[i].clear();
    }
    char str[2][maxn];
    for (int i = 1; i <= m; i++) {
      scanf("%s %s", str[0], str[1]);
      int x = 0, y = 0;
      for (int i = 1; i < strlen(str[0]); i++)
        x = x * 10 + str[0][i] - '0';
      for (int i = 1; i < strlen(str[1]); i++)
        y = y * 10 + str[1][i] - '0';
      if (str[0][0] == 'm')
        x = n + x;
      if (str[1][0] == 'm')
        y = n + y;
      add_edge(2 * n + y, x), add_edge(2 * n + x, y);
    }
    for (int i = 1; i <= n; i++) {
      // p:i q:i+n not p:2*n+i not q:3*n+i
      add_edge(i, 3 * n + i), add_edge(i + n, 2 * n + i);
    }
    scc();
    int flag = 0;
    for (int i = 1; i <= n; i++) {
      if (cnt[i] == cnt[i + n]) {
        printf("BAD\n");
        flag = 1;
        break;
      }
    }
    if (!flag)
      printf("GOOD\n");
  }
}
```