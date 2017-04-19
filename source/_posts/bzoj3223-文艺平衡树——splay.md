---
title: '[bzoj3223]文艺平衡树——splay'
tags: [数据结构, splay]
date: 2017-02-27 10:09:00
---

# 题意
你应当编写一个数据结构，支持以下操作：
1\. 反转一个区间

# 题解
我们把在数组中的位置当作权值，这样原序列就在这种权值意义下有序，我们考虑使用splay维护。
对于操作rev[l,r]，我们首先把l-1 splay 到根，再把r+1 splay 到根的右子树的根，那么根的右子树的左子树就是区间[l,r]。
显然，这样会用到l-1和r+1。我们考虑添加两个哨兵节点，分别是1和n+2,[1,n]的值再分别加一就好了。
另外，为了防止过多的swap，我们延续线段树中的做法， 打一个lazy标记。与线段树不同的是，由于splay的拓扑结构经常会变化，所以每访问到一个节点，都要下传lazy标记。
注意build的写法。
# 代码
```cpp
#include <bits/stdc++.h>
using namespace std;
int n, m, sz, rt;
#ifdef DEBUG
const int maxn = 100;
#else
const int maxn = 100005;
#endif
int fa[maxn], c[maxn][2], id[maxn];
int size[maxn];
bool rev[maxn];
void pushup(int k) {
  int l = c[k][0], r = c[k][1];
  size[k] = size[l] + size[r] + 1;
}
void pushdown(int k) {
  int &l = c[k][0], &r = c[k][1];
  if (rev[k]) {
    swap(l, r);
    rev[l] ^= 1;
    rev[r] ^= 1;
    rev[k] = 0;
  }
}
void zig(int x, int &k) {         // do one zig
  int y = fa[x], z = fa[y], l, r; // y:father z:grandfather
  if (c[y][0] == x) {             // zig
    l = 0;
  } else
    l = 1; // zag
  r = l ^ 1;
  if (y == k)
    k = x; // single zig
  else {   // update grandfather
    if (c[z][0] == y)
      c[z][0] = x; // zig-zig
    else
      c[z][1] = x;
  }
  fa[x] = z;
  fa[y] = x;
  fa[c[x][r]] = y;
  c[y][l] = c[x][r];
  c[x][r] = y;
  pushup(y);
  pushup(x);
}
void splay(int x, int &k) {
  while (x != k) {
    int y = fa[x], z = fa[y];
    if (y != k) {
      if (c[y][0] == x ^ c[z][0] == y) // diff:zig-zag or zag-zig
        zig(x, k);
      else // same:zig-zig or zag-zag
        zig(y, k);
    }
    zig(x, k);
  }
}
int find(int k, int rank) {
  pushdown(k);
  int l = c[k][0], r = c[k][1];
  if (size[l] + 1 == rank)
    return k;
  else if (size[l] >= rank)
    return find(l, rank);
  else
    return find(r, rank - size[l] - 1);
}

void rever(int l, int r) {
  int x = find(rt, l), y = find(rt, r + 2);
  splay(x, rt);
  splay(y, c[x][1]);
  int z = c[y][0];
  rev[z] ^= 1;
}
void build(int l, int r, int f) { // f:last node
  if (l > r)
    return;
  int now = id[l], last = id[f];
  if (l == r) {
    fa[now] = last;
    size[now] = 1;
    if (l < f)
      c[last][0] = now;
    else
      c[last][1] = now;
    return;
  }
  int mid = (l + r) >> 1;
  now = id[mid];
  build(l, mid - 1, mid);
  build(mid + 1, r, mid);
  fa[now] = last;
  pushup(mid);
  if (mid < f) {
    c[last][0] = now;
  } else
    c[last][1] = now;
  return;
}

int main() {
#ifdef DEBUG
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n + 2; i++)
    id[i] = ++sz;
  build(1, n + 2, 0);
  rt = (n + 3) >> 1;
  for (int i = 1; i <= m; i++) {
    int l, r;
    scanf("%d %d", &l, &r);
    rever(l, r);
  }
  for (int i = 2; i <= n + 1; i++) {
    printf("%d ", find(rt, i) - 1);
  }
  return 0;
}
```