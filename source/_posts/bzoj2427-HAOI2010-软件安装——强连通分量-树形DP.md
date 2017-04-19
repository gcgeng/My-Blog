---
title: '[bzoj2427][HAOI2010]软件安装——强连通分量+树形DP'
tags: [图论, 强连通分量, 树形dp]
date: 2017-02-22 17:50:00
---

# 题目大意
现在我们的手头有N个软件，对于一个软件i，它要占用Wi的磁盘空间，它的价值为Vi。我们希望从中选择一些软件安装到一台磁盘容量为M计算机上，使得这些软件的价值尽可能大（即Vi的和最大）。

但是现在有个问题：软件之间存在依赖关系，即软件i只有在安装了软件j（包括软件j的直接或间接依赖）的情况下才能正确工作（软件i依赖软件j)。幸运的是，一个软件最多依赖另外一个软件。如果一个软件不能正常工作，那么它能够发挥的作用为0。

我们现在知道了软件之间的依赖关系：软件i依赖软件Di。现在请你设计出一种方案，安装价值尽量大的软件。一个软件只能被安装一次，如果一个软件没有依赖则Di=0，这时只要这个软件安装了，它就能正常工作。

# 题解
根据题目，我们建立图。
显然这个图由一些树和一些scc构成（注意：scc一定不在树上），那么我们可以知道，如果选了scc中的一个点，其他点必须也要选，所以我们把所有的scc缩成一个点，这样就构成了一个森林。
对于一个入度为0的点，我们从一个虚点向其连接一条边，这样图就变成了树。
考虑树形dp，定义f[i][j]为对于i为根的子树总共分配j点权值能拿到的最大value
我们可以有$$f[i][j] = f[k][l] + f[i][j-l]$$
记忆化搜索即可。

# 代码
```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;
const ll maxn = 505;
const ll maxm = 1000;
ll n, m, K, s, ans = 0;
ll w[maxn], v[maxn], W[maxn], V[maxn];
ll cnt[maxn], vis[maxn], in[maxn], f[maxn][maxm];
vector<ll> sc[maxn];
vector<ll> vs;
vector<ll> G[maxn];
vector<ll> rg[maxn];
vector<ll> ng[maxn];
void add(ll from, ll to) {
  G[from].push_back(to);
  rg[to].push_back(from);
}
void add_edge(ll from, ll to) {
  in[to] = 1;
  ng[from].push_back(to);
}
void dfs(ll s) {
  vis[s] = 1;
  for (ll i = 0; i < G[s].size(); i++) {
    if (!vis[G[s][i]])
      dfs(G[s][i]);
  }
  vs.push_back(s);
}
void rdfs(ll s, ll k) {
  vis[s] = 1;
  for (ll i = 0; i < rg[s].size(); i++) {
    if (!vis[rg[s][i]])
      rdfs(rg[s][i], k);
  }
  cnt[s] = k;
  sc[k].push_back(s);
}
void scc() {
  memset(vis, 0, sizeof(vis));
  vs.clear();
  for (ll i = 1; i <= n; i++) {
    if (!vis[i])
      dfs(i);
  }
  ll k = 0;
  memset(vis, 0, sizeof(vis));
  for (ll i = vs.size() - 1; i >= 0; i--) {
    if (!vis[vs[i]])
      rdfs(vs[i], k++);
  }
  K = k;
}
void build_graph() {
  for (ll i = 0; i < K; i++) {
    for (ll j = 0; j < sc[i].size(); j++) {
      W[i] += w[sc[i][j]];
      V[i] += v[sc[i][j]];
    }
  }
  for (ll i = 1; i <= n; i++) {
    for (ll j = 0; j < G[i].size(); j++) {
      if (cnt[i] != cnt[G[i][j]])
        add_edge(cnt[i], cnt[G[i][j]]);
    }
  }
  s = K + 1;
  for (ll i = 0; i < K; i++)
    if (!in[i])
      add_edge(s, i);
}
void dp(ll x) {
  for (ll i = 0; i < ng[x].size(); i++) {
    dp(ng[x][i]);
    for (ll j = m - W[x]; j >= 0; j--) { //鏋氫妇閫夊畬鑷繁鍚庤垂鐢?
      for (ll k = 0; k <= j; k++) {      //鏋氫妇缁欏効瀛愮殑璐圭敤
        f[x][j] = max(f[x][j], f[x][k] + f[ng[x][i]][j - k]);
      }
    }
  }
  for (ll j = m; j >= 0; j--) {
    if (j >= W[x])
      f[x][j] = f[x][j - W[x]] + V[x];
    else
      f[x][j] = 0;
  }
}
int main() {
  // freopen("input", "r", stdin);
  scanf("%lld %lld", &n, &m);
  for (ll i = 1; i <= n; i++)
    scanf("%lld", &w[i]);
  for (ll i = 1; i <= n; i++)
    scanf("%lld", &v[i]);
  for (ll i = 1; i <= n; i++) {
    ll x;
    scanf("%lld", &x);
    if (x)
      add(x, i);
  }
  scc();
  build_graph();
  dp(s);
  printf("%lld\n", f[s][m]);
}
```