---
title: '[bzoj3524==bzoj2223][Poi2014]Couriers/[Coci 2009]PATULJCI——主席树+权值线段树'
tags: [数据结构, 权值线段树, 主席树]
date: 2017-03-03 14:22:00
---

# 题目大意
给定一个大小为n，每个数的大小均在[1,c]之间的数列，你需要回答m个询问，其中第i个询问形如$(l_i, r_i)$，你需要回答是否存在一个数使得它在区间$[l_i,r_i]$中出现至少$\frac{r-l+1}{2}$次。

# 题解
第一次写主席树。
不难发现，对于一个询问，只有可能要么有解，要么有一个解。
考虑到每个数均在一个确定的区间内，我们考虑开一棵权值线段树（以前一直用这种方法，但不知到这就是权值线段树）来记录每一个数字的出现次数。
考虑到他要求询问一个区间，我们只要知道在后一个时间点树的情况和前一个时间点的树的情况就好了。但是，我们需要修改线段树，所以我们需要使用一个可持久化数据结构来实现这种修改与查询。
具体地，我们考虑对于每次修改建一棵新树来存储这个时间点的情况。显然这个时间点可以从上一个时间点推出来。
如果我们真的建一棵新树，显然会爆空间。我们考虑仅仅在修改过的节点新建节点，同时把边<del>乱连</del>合理地连接即可。
对于每一个询问，我们二分答案即可。
至于复杂度，时间复杂度的话就是$\Theta(跑得过)$，但空间复杂度就是$\Theta(不一定能跑过)$辣（逃
不过在bzoj改数据以后还是能过的。
# 注意
二分答案的时候，注意返回0的情况。也就是说，充分考虑到每一种情况并为其设立出口。
# 代码（bzoj3524）
```c++
#include <cstdio>
int n, m, sz;
const int maxn = 500010;
const int maxm = 10000010;
int rt[maxn], lc[maxm], rc[maxm], sum[maxm];
void update(int l, int r, int x, int &y, int v) {
  y = ++sz;
  sum[y] = sum[x] + 1;
  if (l == r)
    return;
  lc[y] = lc[x];
  rc[y] = rc[x];
  int mid = (l + r) >> 1;
  if (v <= mid)
    update(l, mid, lc[x], lc[y], v);
  else
    update(mid + 1, r, rc[x], rc[y], v);
}
int que(int L, int R) {
  int l = 1, r = n, mid, x, y, tmp = (R - L + 1) >> 1;
  x = rt[L - 1];
  y = rt[R];
  while (l != r) {
    if (sum[y] - sum[x] <= tmp)
      return 0;
    mid = (l + r) >> 1;
    if (sum[lc[y]] - sum[lc[x]] > tmp) {
      r = mid;
      x = lc[x];
      y = lc[y];
    } else if (sum[rc[y]] - sum[rc[x]] > tmp) {
      l = mid + 1;
      x = rc[x];
      y = rc[y];
    } else
      return 0;
  }
  return l;
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++) {
    int x;
    scanf("%d", &x);
    update(1, n, rt[i - 1], rt[i], x);
  }
  for (int i = 1; i <= m; i++) {
    int l, r;
    scanf("%d %d", &l, &r);
    printf("%d\n", que(l, r));
  }
}
```

#代码（bzoj2223）
```
#include <cstdio>
int n, m, a, sz;
const int maxn = 500010;
const int maxm = 10000010;
int rt[maxn], lc[maxm], rc[maxm], sum[maxm];
void update(int l, int r, int x, int &y, int v) {
  y = ++sz;
  sum[y] = sum[x] + 1;
  if (l == r)
    return;
  lc[y] = lc[x];
  rc[y] = rc[x];
  int mid = (l + r) >> 1;
  if (v <= mid)
    update(l, mid, lc[x], lc[y], v);
  else
    update(mid + 1, r, rc[x], rc[y], v);
}
int que(int L, int R) {
  int l = 1, r = a, mid, x, y, tmp = (R - L + 1) >> 1;
  x = rt[L - 1];
  y = rt[R];
  while (l != r) {
    if (sum[y] - sum[x] <= tmp)
      return 0;
    mid = (l + r) >> 1;
    if (sum[lc[y]] - sum[lc[x]] > tmp) {
      r = mid;
      x = lc[x];
      y = lc[y];
    } else if (sum[rc[y]] - sum[rc[x]] > tmp) {
      l = mid + 1;
      x = rc[x];
      y = rc[y];
    } else
      return 0;
  }
  return l;
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &a);
  for (int i = 1; i <= n; i++) {
    int x;
    scanf("%d", &x);
    update(1, a, rt[i - 1], rt[i], x);
  }
  scanf("%d", &m);
  for (int i = 1; i <= m; i++) {
    int l, r;
    scanf("%d %d", &l, &r);
    int ans = que(l, r);
    if (ans > 0) {
      printf("yes %d\n", ans);
    } else
      printf("no\n");
  }
}
```