---
title: '[bzoj1588][HNOI2002]营业额统计——splay'
tags: [数据结构, splay]
date: 2017-02-28 14:52:00
---

# 题目大意
你被要求编写一个数据结构，支援以下操作，操作在线。
1\. 插入一个元素
2\. 查询一个元素与之前插入元素的最小差值。

# 题解
一道模板题。我是写了一个pre和succ函数水过的。1A，比较高兴。
# 代码
```cpp
#include <algorithm>
#include <cstdio>
const int maxn = 40000;
int ch[maxn][2], fa[maxn], data[maxn], cnt[maxn], rt, sz, size[maxn];
void update(int k) { size[k] = size[ch[k][0]] + size[ch[k][1]] + cnt[k]; }
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  if (z)
    ch[z][ch[z][1] == y] = x;
  update(y);
}
void splay(int x, int aim = 0) {
  for (int y; (y = fa[x]) != aim; zig(x))
    if (fa[y] != aim)
      zig((ch[y][0] == x) == (ch[fa[y]][0] == x) ? y : x);
  if (aim == 0)
    rt = x;
  update(x);
}
void insert(int v) {
  int x = rt;
  if (rt == 0) {
    rt = x = ++sz;
    data[x] = v;
    size[x] = cnt[x] = 1;
    fa[x] = ch[x][0] = ch[x][1] = 0;
    return;
  }
  while (x) {
    size[x]++;
    if (v == data[x]) {
      cnt[x]++;
      return;
    }
    int &y = ch[x][v >= data[x]];
    if (y == 0) {
      y = ++sz;
      data[y] = v;
      size[y] = cnt[y] = 1;
      fa[y] = x;
      ch[y][0] = ch[y][1] = 0;
      x = y;
      break;
    }
    x = y;
  }
  splay(x);
}
int pre(int v) {
  int ans = -1;
  for (int y = rt; y;) {
    if (data[y] <= v)
      ans = data[y], y = ch[y][1];
    else
      y = ch[y][0];
  }
  return ans;
}
int succ(int v) {
  int ans = -1;
  for (int y = rt; y;) {
    if (data[y] >= v)
      ans = data[y], y = ch[y][0];
    else
      y = ch[y][1];
  }
  return ans;
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  int ans = 0;
  int n, x;
  rt = 0;
  scanf("%d %d", &n, &x);
  insert(x);
  ans += x;
  for (int i = 2; i <= n; i++) {
    scanf("%d", &x);
    int ret = 0x3f3f3f;
    int y = pre(x);
    if (y != -1)
      ret = std::min(ret, abs(x - y));
    y = succ(x);
    if (y != -1)
      ret = std::min(ret, abs(x - y));
    if (ret != 0x3f3f3f)
      ans += ret;
    insert(x);
  }
  printf("%d\n", ans);
  return 0;
}
```