---
title: '[bzoj3306]树——树上倍增+dfs序+线段树'
tags: [树上倍增, dfs序列, 线段树, 数据结构]
date: 2017-03-07 11:55:00
---

# Brief Description
您需要写一种数据结构，支持：
1\. 更改一个点的点权
2\. 求一个子树的最小点权
3\. 换根

# Algorithm Design
我们先忽略第三个要求。
看到要求子树的最小点权，我们想到使用dfs序。容易看到，一个节点的子树在dfs序中的范围就是$[l(x),r(x)]$，所以我们把树结构变成了线性结构，从而变成了一个RMQ问题，我们使用线段树即可求解。
对于换根，我们不必重新求出拓扑结构。我们考察换根会影响到的节点。对于新根的子树中的节点，一定没有影响，对于不是新根祖先的节点，一定也没有影响。所以我们只考虑新根的祖先。
我们可以看出，换成新根以后，祖先的覆盖范围就变成了全树抛去新根到祖先路径上距离祖先距离最近的节点的子树大小，设这个点为y，那么dfs序中的范围就是$[1,l[y]-1] \cup [r[y]+1, n]$。
所以问题就变成了如何求树上离某个点距离最近的儿子。显然可以使用树上倍增来求。

# Code
```cpp
#include <algorithm>
#include <cstdio>
using std::max;
using std::min;
const int maxn = 100010;
int q[maxn], ind = 0, l[maxn], r[maxn], n, m, root, val[maxn], deep[maxn],
             fa[maxn][20];
int cnt = 0;
struct edge {
  int to, next;
} e[maxn];
int last[maxn];
struct seg {
  int l, r, mn;
} t[maxn << 2];
void insert(int x, int y) {
  e[++cnt].to = y;
  e[cnt].next = last[x];
  last[x] = cnt;
}
void update(int k) { t[k].mn = min(t[k << 1].mn, t[k << 1 | 1].mn); }
void dfs(int x) {
  l[x] = ++ind;
  q[ind] = x;
  for (int i = 1; i <= 16; i++) {
    if (deep[x] < (1 << i))
      break;
    fa[x][i] = fa[fa[x][i - 1]][i - 1];
  }
  for (int i = last[x]; i; i = e[i].next) {
    fa[e[i].to][0] = x;
    deep[e[i].to] = deep[x] + 1;
    dfs(e[i].to);
  }
  r[x] = ind;
}
void build(int k, int l, int r) {
  t[k].l = l, t[k].r = r;
  int mid = (l + r) >> 1;
  if (l == r) {
    t[k].mn = val[q[l]];
    return;
  }
  build(k << 1, l, mid);
  build(k << 1 | 1, mid + 1, r);
  t[k].mn = min(t[k << 1].mn, t[k << 1 | 1].mn);
}
void modify(int k, int pos, int val) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (l == r) {
    t[k].mn = val;
    return;
  }
  if (pos <= mid)
    modify(k << 1, pos, val);
  else
    modify(k << 1 | 1, pos, val);
  update(k);
}
int query(int k, int x, int y) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (x <= l && r <= y)
    return t[k].mn;
  int ans = 0x3f3f3f;
  if (x <= mid)
    ans = min(ans, query(k << 1, x, y));
  if (y > mid)
    ans = min(ans, query(k << 1 | 1, x, y));
  return ans;
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++) {
    int f;
    scanf("%d %d", &f, &val[i]);
    if (f)
      insert(f, i);
  }
  dfs(root = 1);
#ifndef ONLINE_JUDGE
  for (int i = 1; i <= ind; i++)
    printf("%d ", q[i]);
  printf("\n");
#endif
  build(1, 1, n);
  while (m--) {
    char ch[5];
    int x;
    scanf("%s %d", ch, &x);
    if (ch[0] == 'V') {
      int val;
      scanf("%d", &val);
      modify(1, l[x], val);
    } else if (ch[0] == 'E')
      root = x;
    else {
      if (root == x)
        printf("%d\n", t[1].mn);
      else if (l[x] <= l[root] && r[x] >= r[root]) { // x is the father of root
        int y = root, d = deep[y] - deep[x] - 1;
        for (int i = 0; i <= 16; i++)
          if (d & (1 << i))
            y = fa[y][i];
        printf("%d\n", min(query(1, 1, l[y] - 1), query(1, r[y] + 1, n)));
      } else
        printf("%d\n", query(1, l[x], r[x]));
    }
  }
  return 0;
}
```