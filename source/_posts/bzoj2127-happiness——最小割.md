---
title: '[bzoj2127]happiness——最小割'
tags: [网络流, 最小割]
date: 2017-02-21 08:13:00
---

#这个题太恶心了。。。并不想继续做了。。。
**本代码在bzoj上TLE！**
大致说一下思路：   建立ST，首先由S连边(S,u,a)a代表学文的分数，连向T(u,T,b)b表示学理的分数，这样构造出了两个人独立的分数。

然后考虑联合分数，对于相邻的两个点xy，看下图（盗个图：
![](http://images2015.cnblogs.com/blog/890886/201702/890886-20170221081306351-455916177.png)

　 设xy都学文的分数为w1,都学理的分数为w2，则a=w1/2,b=w1/2,c=w2/2,d=w2/2,e=(w1+w2)/2，每一种割与其对应的亏损分数如下：

    　　a+b          -w1　　　　　　都学理->w2

    　　c+d          -w2　　　　　　都学文->w1

    　　a+d+e       -w1-w2　　　  不同->   0

    　　c+d+e       -w1-w2　　　　...

　 注意双向边e，我们是变成两条有向边加入网络，而又因为我们求最小割用的是最大流的算法，所以这条边可以看作是一条双向且权值为e的边。

　 然后把权值*2，解决精度问题。

```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 105; /////////////////////////////////////////////
const int maxv = maxn * maxn * 2;
const int inf = INT_MAX;
int n, m, s, t, v;
struct edge {
  int from;
  int to;
  int cap;
};
vector<edge> edges;
vector<int> G[maxv];
int dist[maxv], iter[maxv];
int z[maxv][maxv];
bool zz[maxv][maxv];
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
inline void add_edge(int from, int to, int cap) {
  edges.push_back((edge){from, to, cap});
  edges.push_back((edge){to, from, 0});
  int m = edges.size();
  G[from].push_back(m - 2);
  G[to].push_back(m - 1);
}
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
      int d = dfs(e.to, t, min(flow, e.cap));
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
    int F;
    while (F = dfs(s, t, inf))
      flow += F;
  }
}
int main() {
  // freopen("input", "r", stdin); //////////////////////////
  memset(z, 0, sizeof(z));
  memset(zz, 0, sizeof(zz));
  scanf("%d %d", &n, &m);
  s = 0;         //文科
  t = n * m + 1; //理科
  v = t + 1;
  int ans = 0;
  for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++) {
      int x;
      x = read();
      x <<= 1;
      z[s][(i - 1) * m + j] += x;
      zz[s][(i - 1) * m + j] = 1;
    }
  for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++) {
      int x;
      x = read();
      x <<= 1;
      z[(i - 1) * m + j][t] += x;
      zz[(i - 1) * m + j][t] = 1;
    }
  for (int i = 1; i < n; i++) {
    for (int j = 1; j <= m; j++) {
      int x;
      x = read();
      z[(i - 1) * m + j][i * m + j] += x;
      z[s][(i - 1) * m + j] += x;
      z[s][i * m + j] += x;
      zz[(i - 1) * m + j][i * m + j] = 1;
      zz[s][(i - 1) * m + j] = 1;
      zz[s][i * m + j] = 1;
      add_edge((i - 1) * m + j, i * m + j, x);
      add_edge(i * m + j, (i - 1) * m + j, x);
    }
  }
  for (int i = 1; i < n; i++) {
    for (int j = 1; j <= m; j++) {
      int x;
      x = read();
      z[(i - 1) * m + j][i * m + j] += x;
      z[(i - 1) * m + j][t] += x;
      z[i * m + j][t] += x;
      zz[(i - 1) * m + j][i * m + j] = 1;
      zz[(i - 1) * m + j][t] = 1;
      zz[i * m + j][t] = 1;
      add_edge((i - 1) * m + j, i * m + j, x);
      add_edge(i * m + j, (i - 1) * m + j, x);
    }
  }
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j < m; j++) {
      int x;
      x = read();
      z[(i - 1) * m + j][(i - 1) * m + j + 1] += x;
      z[s][(i - 1) * m + j] += x;
      z[s][(i - 1) * m + j + 1] += x;
      zz[(i - 1) * m + j][(i - 1) * m + j + 1] = 1;
      zz[s][(i - 1) * m + j] = 1;
      zz[s][(i - 1) * m + j + 1] = 1;
      add_edge((i - 1) * m + j, (i - 1) * m + j + 1, x);
      add_edge((i - 1) * m + j + 1, (i - 1) * m + j, x);
    }
  }
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j < m; j++) {
      int x;
      x = read();
      z[(i - 1) * m + j][(i - 1) * m + j + 1] += x;
      z[(i - 1) * m + j][t] += x;
      z[(i - 1) * m + j + 1][t] += x;
      zz[(i - 1) * m + j][(i - 1) * m + j + 1] = 1;
      zz[(i - 1) * m + j][t] = 1;
      zz[(i - 1) * m + j + 1][t] = 1;
      add_edge((i - 1) * m + j, (i - 1) * m + j + 1, x);
      add_edge((i - 1) * m + j + 1, (i - 1) * m + j, x);
    }
  }
  for (int i = 1; i < t; i++) {
    ans += z[s][i];
    ans += z[i][t];
    if (zz[s][i])
      add_edge(s, i, z[s][i]);
    if (zz[i][t])
      add_edge(i, t, z[i][t]);
  }
  for (int i = 1; i < t; i++) {
    for (int j = 1; j < t; j++) {
      if (zz[i][j]) {
        add_edge(i, j, z[i][j]);
        add_edge(j, i, z[i][j]);
      }
    }
  }
  /*  for (int i = 0; i < v; i++) {
      cout << "For " << i << ':' << endl;
      for (int j = 0; j < G[i].size(); j++) {
        edge &e = edges[G[i][j]];
        if (e.cap > 0)
          cout << "to " << e.to << " cap " << e.cap << endl;
      }
    }*/
  ans -= dinic(s, t);
  printf("%d\n", ans >> 1);
}
```