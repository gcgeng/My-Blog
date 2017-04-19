---
title: '[bzoj1503][NOI2004]郁闷的出纳员——splay'
tags: [数据结构, splay]
date: 2017-02-28 17:31:00
---

# 题意
你被要求编写一个数据结构，支援以下操作：
1\. 插入一个数
2\. 所有数同时加一个数
3\. 所有数同时减一个数，同时如果有数小于一个值，那么这个数应该被删除。
4\. 查找第k大数
5\. 统计被删除的数的总数

# 题解
看到有插入，有删除，加减还都很方便，自然地想到使用平衡树实现。
我选用了splay。
对于第一个操作和第四个操作，非常简单不再多说。
对于第二个操作，我们采用和线段树类似的做法，打一个lazy标记，考虑到加减没有优先级，随便搞一搞就可以。
对于第三个操作，我们仍然先打lazy标记，然后我们在树中查找min的后继节点，把他splay到根，断掉他的左子树就可以了。

# 注意
元素的重复。
pushdown函数虽然简单但是要反复检查。
注意边界情况与特判

#代码
```cpp
#include <cctype>
#include <cstdio>
const int maxn = 100010;
int ch[maxn][2], data[maxn], fa[maxn], cnt[maxn], size[maxn], ad[maxn],
    dc[maxn];
int sz = 0, rt = 0;
int n, m, total = 0;
void update(int x) { size[x] = size[ch[x][0]] + size[ch[x][1]] + cnt[x]; }
void pushdown(int x) {
  if (ad[x] != 0 || dc[x] != 0) {
    int l = ch[x][0], r = ch[x][1];
    if (l == 0 && r == 0) {
      data[x] += ad[x];
      data[x] -= dc[x];
      ad[x] = 0;
      dc[x] = 0;
      return;
    }
    if (l != 0) {
      ad[l] += ad[x];
      dc[l] += dc[x];
    }
    if (r != 0) {
      ad[r] += ad[x];
      dc[r] += dc[x];
    }
    data[x] += ad[x];
    data[x] -= dc[x];
    ad[x] = dc[x] = 0;
    return;
  }
}
void zig(int x) {
  pushdown(x);
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  ch[z][ch[z][1] == y] = x;
  update(y);
}
void splay(int x, int aim = 0) {
  for (int y; (y = fa[x]) != aim; zig(x))
    if (fa[y] != aim)
      zig((((ch[y][0] == x) == (ch[fa[y]][0] == y))) ? y : x);
  if (aim == 0)
    rt = x;
  update(x);
}
void insert(int v) {
  int x = rt;
  if (rt == 0) {
    rt = x = ++sz;
    data[x] = v;
    cnt[x] = size[x] = 1;
    fa[x] = ch[x][0] = ch[x][1] = 0;
    return;
  }
  while (x) {
    size[x]++;
    pushdown(x);
    if (v == data[x]) {
      cnt[x]++;
      return;
    }
    int &y = ch[x][v > data[x]];
    if (y == 0) {
      y = ++sz;
      data[y] = v;
      size[y] = cnt[y] = 1;
      fa[y] = x;
      ch[y][0] = ch[y][1] = 0;
      ch[x][v > data[x]] = y;
      x = y;
      break;
    }
    x = y;
  }
  splay(x);
}
void add(int x) {
  pushdown(rt);
  ad[rt] = x;
}
void dec(int x) {
  pushdown(rt);
  dc[rt] = x;
}
void fire() {
  int x = rt;
  int y = -1;
  while (x) {
    pushdown(x);
    if (data[x] < m) {
      x = ch[x][1];
    } else {
      y = x;
      x = ch[x][0];
    }
  }
  if (y == -1) {
    total += size[rt];
    rt = 0;
    return;
  }
  splay(y);
  total += size[ch[rt][0]];
  ch[rt][0] = 0;
  update(rt);
}
int kth(int v) {
  if (v > size[rt])
    return -1;
  int x = rt;
  while (x) {
    pushdown(x);
    if (v >= size[ch[x][1]] + 1 && v <= size[ch[x][1]] + cnt[x])
      return data[x];
    if (v > size[ch[x][1]] + cnt[x]) {
      v -= size[ch[x][1]] + cnt[x];
      x = ch[x][0];
    } else
      x = ch[x][1];
  }
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++) {
    char opt = getchar();
    while (!isalpha(opt))
      opt = getchar();
    ;
    if (opt == 'I') {
      int x;
      scanf("%d", &x);
      if (x >= m)
        insert(x);
    } else if (opt == 'A') {
      int x;
      scanf("%d", &x);
      add(x);
    } else if (opt == 'S') {
      int x;
      scanf("%d", &x);
      dec(x);
      fire();
    } else if (opt == 'F') {
      int x;
      scanf("%d", &x);
      printf("%d\n", kth(x));
    }
  }
  printf("%d\n", total);
}
```