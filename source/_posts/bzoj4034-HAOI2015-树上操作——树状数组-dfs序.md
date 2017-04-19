---
title: '[bzoj4034][HAOI2015]树上操作——树状数组+dfs序'
tags: [树状数组, 数据结构, dfs序列]
date: 2017-03-10 15:19:00
---

# Brief Description
您需要设计一种数据结构支持以下操作：
1\. 把某个节点 x 的点权增加 a 。
2\. 把某个节点 x 为根的子树中所有点的点权都增加 a 。
3\. 询问某个节点 x 到根的路径中所有点的点权和。

# Algorithm Design
我们考察操作对于查询的贡献。
对于操作1，如果节点y是节点x的后代，那么可以贡献$a$
对于操作2，如果节点y是节点x的后代，那么可以贡献$a*(dep_y-dep_x+1)$
我们可以使用两个树状数组来维护贡献。

# Code
```cpp
#include <cstdio>
#define lowbit(i) (i) & -(i)
const int maxn = 101000;
#define ll long long
ll bit[2][maxn];
ll n, m, cnt = 0;
void change(ll id, ll pos, ll val) {
  for (ll i = pos; i <= n; i += lowbit(i)) {
    bit[id][i] += val;
  }
}
ll query(ll id, ll pos) {
  ll ans = 0;
  for (ll i = pos; i; i -= lowbit(i)) {
    ans += bit[id][i];
  }
  return ans;
}
struct edge {
  ll to, next;
} e[maxn << 1];
ll l[maxn], r[maxn], dfn = 0, val[maxn], deep[maxn], head[maxn], q[maxn];
void add(ll x, ll y) {
  e[++cnt].to = y;
  e[cnt].next = head[x];
  head[x] = cnt;
}
void add_edge(ll x, ll y) {
  add(x, y);
  add(y, x);
}
void dfs(ll x, ll fa) {
  l[x] = ++dfn;
  q[dfn] = x;
  for (ll i = head[x]; i; i = e[i].next) {
    if (e[i].to != fa) {
      deep[e[i].to] = deep[x] + 1;
      dfs(e[i].to, x);
    }
  }
  r[x] = dfn;
}
int main() {
  //  freopen("haoi2015_t2.in", "r", stdin);
  //  freopen("haoi2015_t2.out", "w", stdout);
  scanf("%lld %lld", &n, &m);
  for (ll i = 1; i <= n; i++) {
    scanf("%lld", &val[i]);
  }
  for (ll i = 1; i < n; i++) {
    ll x;
    ll y;
    scanf("%lld %lld", &x, &y);
    add_edge(x, y);
  }
  dfs(1, 0);
  for (ll i = 1; i <= n; i++) {
    change(1, l[i], val[i]);
    change(1, r[i] + 1, -val[i]);
  }
  while (m--) {
    ll opt, x, y;
    scanf("%lld %lld", &opt, &x);
    if (opt == 1) {
      scanf("%lld", &y);
      change(1, l[x], y);
      change(1, r[x] + 1, -y);
    }
    if (opt == 2) {
      scanf("%lld", &y);
      change(0, l[x], y);
      change(1, l[x], -deep[x] * y + y);
      change(0, r[x] + 1, -y);
      change(1, r[x] + 1, deep[x] * y - y);
    }
    if (opt == 3) {
      printf("%lld\n", query(0, l[x]) * deep[x] + query(1, l[x]));
    }
  }
}
```