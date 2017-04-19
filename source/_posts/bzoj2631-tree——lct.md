---
title: '[bzoj2631]tree——lct'
tags: [lct, 数据结构]
date: 2017-03-10 10:43:00
---

# Brief Description
　一棵n个点的树，每个点的初始权值为1。对于这棵树有q个操作，每个操作为以下四种操作之一：
\+ u v c：将u到v的路径上的点的权值都加上自然数c；
\- u1 v1 u2 v2：将树中原有的边(u1,v1)删除，加入一条新边(u2,v2)，保证操作完之后仍然是一棵树；
\* u v c：将u到v的路径上的点的权值都乘上自然数c；
  / u v：询问u到v的路径上的点的权值和，求出答案对于51061的余数。

# Algorithm Design
lct裸题。考察标记下传，先下传乘法，再下传加法。

# Code
```cpp
#include <algorithm>
#include <cctype>
#include <cstdio>
#define ll unsigned int
const int maxn = 100005;
#define mod 51061
inline int read() {
  int x = 0, f = 1;
  char ch = getchar();
  while (!isdigit(ch)) {
    if (ch == '-') {
      f = -1;
    }
    ch = getchar();
  }
  while (isdigit(ch)) {
    x = x * 10 + ch - '0';
    ch = getchar();
  }
  return x * f;
}
int n, m, top, cnt;
int ch[maxn][2], fa[maxn], size[maxn];
bool rev[maxn];
ll sum[maxn], val[maxn], at[maxn], mt[maxn];
inline bool isroot(int x) { return ch[fa[x]][0] != x && ch[fa[x]][1] != x; }
inline void cal(int k, int m, int a) {
  if (k) {
    val[k] = (val[k] * m + a) % mod;
    sum[k] = (sum[k] * m + a * size[k]) % mod;
    at[k] = (at[k] * m + a) % mod;
    mt[k] = (mt[k] * m) % mod;
  }
}
inline void update(int k) {
  int l = ch[k][0], r = ch[k][1];
  sum[k] = (sum[l] + sum[r] + val[k]) % mod;
  size[k] = (size[l] + size[r] + 1) % mod;
}
inline void pushdown(int k) {
  int &l = ch[k][0], &r = ch[k][1];
  if (rev[k]) {
    rev[k] ^= 1, rev[l] ^= 1, rev[r] ^= 1;
    std::swap(l, r);
  }
  int m = mt[k], a = at[k];
  mt[k] = 1, at[k] = 0;
  if (m != 1 || a != 0) {
    cal(l, m, a);
    cal(r, m, a);
  }
}
inline void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  if (!isroot(y))
    ch[z][ch[z][1] == y] = x;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  update(y);
  update(x);
}
inline void splay(int x) {
  int s[maxn], top = 0;
  s[++top] = x;
  for (int i = x; !isroot(i); i = fa[i])
    s[++top] = fa[i];
  while (top)
    pushdown(s[top--]);
  for (int y; !isroot(x); zig(x))
    if (!isroot(y = fa[x]))
      zig((ch[fa[y]][0] == y) == (ch[y][0] == x) ? y : x);
}
inline void access(int x) {
  for (int t = 0; x; t = x, x = fa[x]) {
    splay(x);
    ch[x][1] = t;
    update(x);
  }
}
inline void makeroot(int x) {
  access(x);
  splay(x);
  rev[x] ^= 1;
}
inline void split(int x, int y) {
  makeroot(y);
  access(x);
  splay(x);
}
inline void link(int x, int y) {
  makeroot(x);
  fa[x] = y;
}
inline void cut(int x, int y) {
  makeroot(x);
  access(y);
  splay(y);
  ch[y][0] = fa[x] = 0;
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  n = read(), m = read();
  int u, v, c;
  for (int i = 1; i <= n; i++)
    val[i] = sum[i] = size[i] = mt[i] = 1;
  for (int i = 1; i < n; i++) {
    u = read(), v = read();
    link(u, v);
  }
  char st[5];
  while (m--) {
    scanf("%s", st);
    u = read(), v = read();
    if (st[0] == '+') {
      c = read();
      split(u, v);
      cal(u, 1, c);
    }
    if (st[0] == '-') {
      cut(u, v);
      u = read(), v = read(), link(u, v);
    }
    if (st[0] == '*') {
      c = read();
      split(u, v);
      cal(u, c, 0);
    }
    if (st[0] == '/') {
      split(u, v);
      printf("%d\n", sum[u]);
    }
  }
}
```