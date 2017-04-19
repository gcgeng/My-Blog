---
title: '[bzoj1798][Ahoi2009]Seq——线段树+多重标记下传'
tags: [线段树, 数据结构]
date: 2017-02-25 10:08:00
---

# 题意
请你写一个数据结构，支持：
1\. 子序列同加
2\. 子序列同乘
3\. 统计子序列和
<!--more-->

# 题目
线段树裸题，但对于我这种初学者还是非常难写。
我们维护两个标记，一个是在这个节点上作过的所有乘法操作，一个是加法操作，始终保持乘法优先级在前，这就说明，如果原来已经有了加法，那么我们需要把加法让位，即把加法×乘法，然后乘法搞一下原来的值。
对于如果一个操作与一个区间不完全覆盖，我们需要把lazy标记下传，对于下传操作，也需要按照上面的方法计算优先级，同时更新节点sum值。
注意在加法操作时，不要使用节点的add值更新sum，而要用val，因为add已经计算在sum里面了。

# 代码
```
#include <bits/stdc++.h>
using namespace std;
#define ll long long
ll n, p;
ll m;
const int maxn = 100005;
struct seg {
  int l, r;
  ll sum, add, tag;
} t[4 * maxn];
void build(int k, int l, int r) {
  t[k].l = l;
  t[k].r = r;
  if (r == l) {
    t[k].tag = 1;
    t[k].add = 0;
    scanf("%lld", &t[k].sum);
    t[k].sum %= p;
    return;
  }
  int mid = (l + r) >> 1;
  build(k << 1, l, mid);
  build(k << 1 | 1, mid + 1, r);
  t[k].sum = (t[k << 1].sum + t[k << 1 | 1].sum) % p;
  t[k].add = 0;
  t[k].tag = 1;
}
void pushdown(int k) {
  if (t[k].add == 0 && t[k].tag == 1)
    return;
  ll ad = t[k].add, tag = t[k].tag;
  ad %= p, tag %= p;
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  t[k << 1].tag = (t[k << 1].tag * tag) % p;
  t[k << 1].add = ((t[k << 1].add * tag) % p + ad) % p;
  t[k << 1].sum = (((t[k << 1].sum * tag) % p + (ad * (mid - l + 1) % p)%p)%p) % p;
  t[k << 1 | 1].tag = (t[k << 1 | 1].tag * tag) % p;
  t[k << 1 | 1].add = ((t[k << 1 | 1].add * tag) % p + ad) % p;
  t[k << 1 | 1].sum = (((t[k << 1|1].sum * tag) % p + (ad * (r-mid) % p)%p)%p) % p;
  t[k].add = 0;
  t[k].tag = 1;
  return;
}
void update(int k) { t[k].sum = (t[k << 1].sum%p + t[k << 1 | 1].sum%p) % p; }
void add(int k, int x, int y, ll val) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (x <= l && r <= y) {
    t[k].add = (t[k].add + val) % p;
    t[k].sum = (t[k].sum + (val * (r - l + 1) % p) % p) % p;
    return;
  }
  pushdown(k);
  if (x <= mid)
    add(k << 1, x, y, val);
  if (y > mid)
    add(k << 1 | 1, x, y, val);
  update(k);
}
void mul(int k, int x, int y, ll val) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (x <= l && r <= y) {
    t[k].add = (t[k].add * val) % p;
    t[k].tag = (t[k].tag * val) % p;
    t[k].sum = (t[k].sum * val) % p;
    return;
  }
  pushdown(k);
  if (x <= mid)
    mul(k << 1, x, y, val);
  if (y > mid)
    mul(k << 1 | 1, x, y, val);
  update(k);
}
ll query(int k, int x, int y) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (x <= l && r <= y) {
    return t[k].sum%p;
  }
  pushdown(k);
  ll ans = 0;
  if (x <= mid)
    ans = (ans + query(k << 1, x, y)) % p;
  if (y > mid)
    ans = (ans + query(k << 1 | 1, x, y)) % p;
  update(k);
  return ans%p;
}
int main() {
  // freopen("seq.in", "r", stdin);
  // freopen("seq.ans", "w", stdout);
  scanf("%lld %lld", &n, &p);
  build(1, 1, n);
  scanf("%d", &m);
  while (m--) {
    int q;
    scanf("%d", &q);
    if (q == 1) {
      int x, y;
      ll z;
      scanf("%d%d%lld", &x, &y, &z);
      z%=p;
      mul(1, x, y, z);
    } else if (q == 2) {
      int x, y;
      ll z;
      scanf("%d%d%lld", &x, &y, &z);
      z%=p;
      add(1, x, y, z);
    } else {
      int x, y;
      scanf("%d %d", &x, &y);
      printf("%lld\n", query(1, x, y));
    }
  }
  return 0;
}
```