---
title: '[bzoj1486][HNOI2009]最小圈——分数规划+spfa+负环'
tags: [分数规划, spfa, 负环]
date: 2017-02-22 09:32:00
---

#题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1486)
#题解
这个题是一个经典的分数规划问题。
把题目形式化地表示，就是
$$Minimize\ \lambda = \frac{\sum W_{i, i+1}}{k}$$
整理一下，就是
$$\lambda * k = \sum W_{i, i+1}$$
定义新的函数
$$g(\lambda) = Min(\lambda * k - \sum W_{i, i+1})$$
显然这个函数单调，我们二分$\lambda$，等价于求一个负环。
如果用spfa求负环会Tle，所以学习了用dfs求负环，如代码所示。
#代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 3005 * 2;
#define exp 1e-10
const double inf = 1000000000;
int n, m;
struct edge {
  int to;
  double value;
};
vector<edge> G[maxn];
vector<edge> rg[maxn];
int vis[maxn];
int flag;
double dist[maxn];
inline void spfa(int x) {
  int i;
  vis[x] = false;
  for (i = 0; i < rg[x].size(); i++) {
    edge &e = rg[x][i];
    if (dist[e.to] > dist[x] + e.value)
      if (!vis[e.to]) {
        flag = true;
        break;
      } else {
        dist[e.to] = dist[x] + e.value;
        spfa(e.to);
      }
  }
  vis[x] = true;
}
bool check(double lambda) {
  for (int i = 1; i <= n; i++) {
    rg[i].clear();
    for (int j = 0; j < G[i].size(); j++) {
      rg[i].push_back((edge){G[i][j].to, (double)G[i][j].value - lambda});
    }
  }
  memset(vis, 1, sizeof(vis));
  memset(dist, 0, sizeof(dist));
  flag = false;
  for (int i = 1; i <= n; i++) {
    spfa(i);
    if (flag)
      return true;
  }
  return false;
}
int main() {
  // freopen("input", "r", stdin);
  scanf("%d %d", &n, &m);
  int tot = 0;
  for (int i = 1; i <= m; i++) {
    int x, y;
    double z;
    scanf("%d %d %lf", &x, &y, &z);
    G[x].push_back((edge){y, z});
    tot += z;
  }
  double l = -inf, r = inf;
  while (l + exp < r) {
    double mid = (l + r) / 2;
    if (check(mid))
      r = mid;
    else
      l = mid;
  }
  printf("%.8f", l);
}
```