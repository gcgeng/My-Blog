---
title: '[bzoj2662][BeiJing wc2012]冻结——分层图最短路'
tags: [图论, 分层图, spfa]
date: 2017-02-23 12:08:00
---

# 题目大意
我们考虑最简单的旅行问题吧：  现在这个大陆上有 N 个城市，M 条双向的
道路。城市编号为 1~N，我们在 1 号城市，需要到 N 号城市，怎样才能最快地
到达呢？
  这不就是最短路问题吗？我们都知道可以用 Dijkstra、Bellman-Ford、
Floyd-Warshall等算法来解决。
  现在，我们一共有 K 张可以使时间变慢 50%的 SpellCard，也就是说，在通
过某条路径时，我们可以选择使用一张卡片，这样，我们通过这一条道路的时间
就可以减少到原先的一半。需要注意的是：
  1\. 在一条道路上最多只能使用一张 SpellCard。
  2\. 使用一张SpellCard 只在一条道路上起作用。
  3\. 你不必使用完所有的 SpellCard。

  给定以上的信息，你的任务是：求出在可以使用这不超过 K 张时间减速的
SpellCard 之情形下，从城市1 到城市N最少需要多长时间。 
# 题解
记录dist[i][j]为到了i点用了j张票。
用spfa转移状态。
# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 55;
struct edge {
  int to;
  int value;
};
struct state {
  int pos;
  int tic;
};
vector<edge> G[maxn];
void add_edge(int from, int to, int value) {
  G[from].push_back((edge){to, value});
  G[to].push_back((edge){from, value});
}

int dist[maxn][maxn], inq[maxn][maxn];
int n, m, k;
void spfa() {
  memset(dist, 0x3f, sizeof(dist));
  memset(inq, 0, sizeof(inq));
  for (int i = 0; i <= k; i++)
    dist[1][i] = 0;
  queue<state> q;
  q.push((state){1, 0});
  inq[1][0] = 1;
  while (!q.empty()) {
    state u = q.front();
    q.pop();
    inq[u.pos][u.tic] = 0;
    for (int i = 0; i < G[u.pos].size(); i++) {
      edge &e = G[u.pos][i];
      if (dist[e.to][u.tic] > dist[u.pos][u.tic] + e.value) {
        dist[e.to][u.tic] = dist[u.pos][u.tic] + e.value;
        if (!inq[e.to][u.tic]) {
          q.push((state){e.to, u.tic});
          inq[e.to][u.tic] = 0;
        }
      }
    }
    if (u.tic < k)
      for (int i = 0; i < G[u.pos].size(); i++) {
        edge &e = G[u.pos][i];
        if (dist[e.to][u.tic + 1] > dist[u.pos][u.tic] + (e.value >> 1)) {
          dist[e.to][u.tic + 1] = dist[u.pos][u.tic] + (e.value >> 1);
          if (!inq[e.to][u.tic + 1]) {
            q.push((state){e.to, u.tic + 1});
            inq[e.to][u.tic + 1] = 1;
          }
        }
      }
  }
}
int main() {
  // freopen("input", "r", stdin);
  scanf("%d %d %d", &n, &m, &k);
  for (int i = 1; i <= m; i++) {
    int x, y, z;
    scanf("%d %d %d", &x, &y, &z);
    add_edge(x, y, z);
  }
  spfa();
  int ans = 0x3f3f3f;
  for (int i = 0; i <= k; i++) {
    ans = min(ans, dist[n][i]);
  };
  printf("%d\n", ans);
  return 0;
}
```