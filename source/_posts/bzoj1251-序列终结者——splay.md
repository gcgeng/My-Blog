---
title: '[bzoj1251]序列终结者——splay'
tags: [数据结构, splay]
date: 2017-03-01 15:18:00
---

#题目大意
网上有许多题，就是给定一个序列，要你支持几种操作：A、B、C、D。一看另一道题，又是一个序列 要支持几种操作：D、C、B、A。尤其是我们这里的某人，出模拟试题，居然还出了一道这样的，真是没技术含量……这样 我也出一道题，我出这一道的目的是为了让大家以后做这种题目有一个“库”可以依靠，没有什么其他的意思。这道题目 就叫序列终结者吧。 【问题描述】 给定一个长度为N的序列，每个序列的元素是一个整数（废话）。要支持以下三种操作： 1\. 将[L,R]这个区间内的所有数加上V。 2\. 将[L,R]这个区间翻转，比如1 2 3 4变成4 3 2 1。 3\. 求[L,R]这个区间中的最大值。 最开始所有元素都是0。
Input

第一行两个整数N，M。M为操作个数。 以下M行，每行最多四个整数，依次为K，L，R，V。K表示是第几种操作，如果不是第1种操作则K后面只有两个数。
Output

对于每个第3种操作，给出正确的回答。
#题解
首先，终于做完splay的5道模板题了。
说一下这个题怎么做。
我们沿用线段树中的lazy标记，给每个节点打两个标记，一旦在find中访问到这个节点，立即pushdown，同时及时update。
然后就是splay的经典操作了。
这个题调了很长时间，所以，以后一定要先静态调试，避免SB错误和理清逻辑后，再用gdb动态调试。
#代码
```c++
#include <algorithm>
#include <cstdio>
#include <cstring>
#ifdef D
const int maxn = 100;
#else
const int maxn = 100000;
#endif
int data[maxn], fa[maxn], ch[maxn][2], rev[maxn], mx[maxn], pls[maxn],
    size[maxn];
int n, m, sz = 0, rt = 0;
void update(int x) {
  mx[x] = data[x];
  if (ch[x][0] != -1)
    mx[x] = std::max(mx[x], mx[ch[x][0]]);
  if (ch[x][1] != -1)
    mx[x] = std::max(mx[x], mx[ch[x][1]]);
  size[x] = 1;
  if (ch[x][0] != -1)
    size[x] += size[ch[x][0]];
  if (ch[x][1] != -1)
    size[x] += size[ch[x][1]];
}
void pushdown(int k) {
  int &l = ch[k][0], &r = ch[k][1], t = pls[k];
  if (t) {
    pls[k] = 0;
    if (l != -1) {
      pls[l] += t;
      mx[l] += t;
      data[l] += t;
    }
    if (r != -1) {
      pls[r] += t;
      mx[r] += t;
      data[r] += t;
    }
  }
  if (rev[k]) {
    rev[k] = 0;
    rev[l] ^= 1;
    rev[r] ^= 1;
    std::swap(ch[k][0], ch[k][1]);
  }
}
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  if (z != -1) {
    ch[z][ch[z][1] == y] = x;
  }
  update(y);
  update(x);
}
void splay(int x, int aim = -1) {
  for (int y; (y = fa[x]) != aim; zig(x))
    if (fa[y] != aim)
      zig((ch[fa[y]][0] == y) == (ch[y][0] == x) ? y : x);
  if (aim == -1)
    rt = x;
}
int find(int k, int rank) {
  if (pls[k] || rev[k])
    pushdown(k);
  int l = ch[k][0], r = ch[k][1];
  if (size[l] + 1 == rank)
    return k;
  else if (size[l] >= rank)
    return find(l, rank);
  else
    return find(r, rank - size[l] - 1);
}
void build(int l, int r, int last) {
  if (l > r)
    return;
  int now = l;
  if (l == r) {
    size[now] = 1;
    fa[now] = last;
    data[now] = rev[now] = mx[now] = 0;
    if (l < last)
      ch[last][0] = now;
    else
      ch[last][1] = now;
    return;
  }
  int mid = (l + r) >> 1;
  now = mid;
  build(l, mid - 1, mid);
  build(mid + 1, r, mid);
  fa[now] = last;
  update(now);
  if (mid < last) {
    ch[last][0] = now;
  } else
    ch[last][1] = now;
  return;
}
void rv(int l, int r) {
  int x = find(rt, l), y = find(rt, r + 2);
  splay(x);
  splay(y, rt);
  int z = ch[y][0];
  rev[z] ^= 1;
}
void ps(int l, int r, int v) {
  int x = find(rt, l), y = find(rt, r + 2);
  splay(x);
  splay(y, rt);
  int z = ch[y][0];
  pls[z] += v;
  data[z] += v;
  mx[z] += v;
}
int Max(int l, int r) {
  int x = find(rt, l), y = find(rt, r + 2);
  splay(x);
  splay(y, rt);
  int z = ch[y][0];
  return mx[z];
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  memset(ch, -1, sizeof(ch));
  memset(rev, 0, sizeof(rev));
  memset(pls, 0, sizeof(pls));
  memset(fa, 0, sizeof(fa));
  build(0, n + 1, -1);
  rt = (n + 1) >> 1;
  for (int i = 1; i <= m; i++) {
    int k;
    scanf("%d", &k);
    if (k == 1) {
      int l, r, v;
      scanf("%d %d %d", &l, &r, &v);
      ps(l, r, v);
    }
    if (k == 2) {
      int l, r;
      scanf("%d %d", &l, &r);
      rv(l, r);
    }
    if (k == 3) {
      int l, r;
      scanf("%d %d", &l, &r);
      int ans = Max(l, r);
      printf("%d\n", ans);
    }
  }
}
```