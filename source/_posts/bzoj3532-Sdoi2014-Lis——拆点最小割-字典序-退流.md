---
title: '[bzoj3532][Sdoi2014]Lis——拆点最小割+字典序+退流'
tags: [网络流, 拆点, 最小割, 退流, 字典序]
date: 2017-02-21 18:01:00
---

# 题目大意
 给定序列A，序列中的每一项Ai有删除代价Bi和附加属性Ci。请删除若
干项，使得4的最长上升子序列长度减少至少1，且付出的代价之和最小，并输出方案。
    如果有多种方案，请输出将删去项的附加属性排序之后，字典序最小的一种。
# 题解
首先我们很容易用一个$\Theta (n^2)$的算法求出对于每个元素的lis。
考虑以下的建图方式：
由S向f[i]==1的点连边，容量为$\infty$，
由f[i] = max 向 T 连边， 容量为$\infty$,
对于每个点，拆为两个点，费用就是B[i]。
为什么这样建图是对的？原因很简单，原问题等价于在新图中找一个点集，使得s到t不再连通，这让我们联想到最小割，考虑到s，t到每个点不计入费用，所以我们设为$\infty$。
那么第一个问题解决了，关键是第二个问题：怎样找出字典序最小的割？
我们这样考虑：既然是字典序最小，我们把C[i]从小到大排序，依次检查每一条边，如果在剩余网络中它不连通，那么这条边一定是原图的一个最小割，我们把这条边删除，同时从汇点开始退流。具体的，我们可以从汇点向v跑一次最大流，u向原点跑一次最大流，这样一定会退回一些流，使得这条边的影响消失。
这样我们就解决了这个问题，详细过程见（我写了4个小时的）代码。

# 代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 10000;
int A[maxn], B[maxn], C[maxn], f[maxn];
int n, s, t, v;
const int inf = 1000000010;
struct edge {
  int from;
  int to;
  int cap;
};
struct haha {
  int id;
  int Ci;
} cc[maxn];

inline int read() {
  char c = getchar();
  int f = 1, x = 0;
  while (!isdigit(c)) {
    if (c == '-')
      f = -1;
    c = getchar();
  }
  while (isdigit(c))
    x = x * 10 + c - '0', c = getchar();
  return x * f;
}

bool cmp(haha a, haha b) { return a.Ci < b.Ci; }

vector<edge> edges;
vector<int> G[maxn];
inline void add_edge(int from, int to, int cap) {
  edges.push_back((edge){from, to, cap});
  edges.push_back((edge){to, from, 0});
  int m = edges.size();
  G[from].push_back(m - 2);
  G[to].push_back(m - 1);
}
inline void reaad() {
  scanf("%d", &n);
  s = 0, t = 2 * n + 1, v = t + 1;
  for (int i = 1; i <= n; i++) {
    A[i] = read();
  }
  for (int i = 0; i < v; i++)
    G[i].clear();
  edges.clear();
  for (int i = 1; i <= n; i++)
    B[i] = read();
  for (int i = 1; i <= n; i++) {
    C[i] = read();
    cc[i].id = i;
    cc[i].Ci = C[i];
  }
  for (int i = 1; i <= n; i++)
    add_edge(i, i + n, B[i]);
  for (int i = 1; i <= n; i++)
    f[i] = 1;
  int ans = 0;
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j < i; j++) {
      if (A[j] < A[i])
        f[i] = max(f[i], f[j] + 1);
    }

    ans = max(ans, f[i]);
  }
  for (int i = 1; i <= n; i++) {
    if (ans == f[i]) {
      add_edge(i + n, t, inf);
    }
    if (f[i] == 1)
      add_edge(s, i, inf);
    for (int j = 1; j <= n; j++) {
      if (A[j] < A[i] && j < i && f[i] == f[j] + 1)
        add_edge(j + n, i, inf);
    }
  }
}
int dist[maxn], iter[maxn];
inline void bfs(int s) {
  memset(dist, -1, sizeof(dist));
  dist[s] = 0;
  queue<int> q;
  q.push(s);
  while (!q.empty()) {
    int u = q.front();
    q.pop();
    for (int i = 0; i < G[u].size(); i++) {
      edge &e = edges[G[u][i]];
      if (e.cap > 0 && dist[e.to] == -1) {
        dist[e.to] = dist[u] + 1;
        q.push(e.to);
      }
    }
  }
}
inline int dfs(int s, int t, int flow) {
  if (s == t)
    return flow;
  for (int &i = iter[s]; i < G[s].size(); i++) {
    edge &e = edges[G[s][i]];
    if (e.cap > 0 && dist[e.to] > dist[s]) {
      int d = dfs(e.to, t, min(e.cap, flow));
      if (d > 0) {
        e.cap -= d;
        edges[G[s][i] ^ 1].cap += d;
        return d;
      }
    }
  }
  return 0;
}
inline int dinic(int s, int t) {
  int flow = 0;
  while (1) {
    bfs(s);
    if (dist[t] == -1)
      return flow;
    memset(iter, 0, sizeof(iter));
    int d;
    while (d = dfs(s, t, inf))
      flow += d;
  }
  return flow;
}
int main() {
  // freopen("input", "r", stdin);
  int T;
  scanf("%d", &T);
  while (T--) {
    reaad();
    int ans = dinic(s, t);
    int tot = 0;
    int rec[maxn];
    memset(rec, 0, sizeof(rec));
    sort(cc + 1, cc + 1 + n, cmp);
    for (int i = 1; i <= n; i++) {
      int now = cc[i].id;
      int u = now, v = now + n;
      bfs(u);
      if (dist[v] != -1)
        continue;
      rec[tot++] = now;
      dinic(t, v);
      dinic(u, s);
      edges[(now - 1) * 2].cap = edges[(now - 1) * 2 + 1].cap = 0;
    }
    sort(rec, rec + tot);
    printf("%d %d\n", ans, tot);
    for (int i = 0; i < tot-1; i++) {
      printf("%d ", rec[i]);
    }
    printf("%d\n", rec[tot-1]);
  }
  return 0;
}
```