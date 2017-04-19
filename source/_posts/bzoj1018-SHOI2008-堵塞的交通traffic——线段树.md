---
title: '[bzoj1018][SHOI2008]堵塞的交通traffic——线段树'
tags: [数据结构, 线段树]
date: 2017-03-02 11:22:00
---

# 题目大意
给定一个2×n的矩形网格，你需要设计一种数据结构支持以下操作。
1\. 连接两个相邻的网格。
2\. 断开两个相邻网格的连接。
3\. 查询两个网格的连通性。

最开始，所有网格之间均不连通。
# 题解
首先看到是图的连通性，我想到了并查集，但是由于题目需要删除操作，只好暂时作罢。
一般化地考虑原题。原题就是给定一个图，查连通性。看似和星球大战非常像。但是，这个题有一个特殊的特点：
它是一个2×n的矩形网格。
显然行的个数远远小于列的个数。我们考虑使用线段树，维护一个2×n的网格的四个顶点的连通性，使用一个结构体定义这种状态，
a1[i][j]表示table[i][l]是否与table[i][r]连通，a2[i][j]表示左边或右边的两个节点的上下连通性。
我们考虑两个节点的合并。
显然，合并需要中间的状态。具体状态转移见代码。
# 代码
```
#include <algorithm>
#include <cstdio>
const int maxn = 100010;
struct status {
  int a1[2][2];
  int a2[2];
} s[maxn << 2];
bool b[maxn << 2][2];
int l[maxn << 2], r[maxn << 2], c;
status update(status s1, status s2, bool b[]) {
  status res;
  for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 2; j++) {
      res.a1[i][j] = (s1.a1[i][0] && b[0] && s2.a1[0][j]) ||
                     (s1.a1[i][1] && b[1] && s2.a1[1][j]);
    }
  }
  res.a2[0] =
      (s1.a2[0]) || (s1.a1[0][0] && b[0] && s2.a2[0] && b[1] && s1.a1[1][1]);
  res.a2[1] =
      (s2.a2[1]) || (s2.a1[0][0] && b[0] && s1.a2[1] && b[1] && s2.a1[1][1]);
  return res;
}
status access(int k, int y1, int y2) {
  int mid = (l[k] + r[k]) >> 1;
  if (y1 <= l[k] && r[k] <= y2)
    return s[k];
  else if (y2 <= mid)
    return access(k << 1, y1, y2);
  else if (y1 > mid)
    return access(k << 1 | 1, y1, y2);
  else
    return update(access(k << 1, y1, y2), access(k << 1 | 1, y1, y2), b[k]);
}
void change(bool v, int k, int x1, int y1, int x2, int y2) {
  int mid = (l[k] + r[k]) >> 1;
  if (x1 == x2 && y1 == mid) {
    b[k][x1] = v;
    s[k] = update(s[k << 1], s[k << 1 | 1], b[k]);
  } else if (l[k] == r[k])
    s[k].a1[0][1] = s[k].a1[1][0] = s[k].a2[0] = s[k].a2[1] = v;
  else {
    change(v, y2 <= mid ? k << 1 : k << 1 | 1, x1, y1, x2, y2);
    s[k] = update(s[k << 1], s[k << 1 | 1], b[k]);
  }
}
void ask(int x1, int y1, int x2, int y2) {
  status left = access(1, 1, y1), right = access(1, y2, c),
         mid = access(1, y1, y2);
  bool res = false;
  for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 2; j++) {
      if (mid.a1[i][j] && (i == x1 || left.a2[1]) && (j == x2 || right.a2[0])) {
        res = true;
        break;
      }
    }
  }
  printf(res ? "Y\n" : "N\n");
}

void build(int k, int x, int y) {
  l[k] = x;
  r[k] = y;
  if (x == y) {
    s[k].a1[0][0] = s[k].a1[1][1] = 1;
    return;
  }
  int mid = (x + y) >> 1;
  build(k << 1, x, mid);
  build(k << 1 | 1, mid + 1, y);
}
int main() {
  scanf("%d", &c);
  build(1, 1, c);
  while (1) {
    char opt[10];
    scanf("%s", opt);
    if (opt[0] == 'E')
      break;
    int x0, y0, x1, y1;
    scanf("%d %d %d %d", &x0, &y0, &x1, &y1);
    --x0, --x1;
    if (y0 > y1) {
      std::swap(x0, x1);
      std::swap(y0, y1);
    }
    if (opt[0] == 'O') {
      change(1, 1, x0, y0, x1, y1);
    } else if (opt[0] == 'C') {
      change(0, 1, x0, y0, x1, y1);
    } else
      ask(x0, y0, x1, y1);
  }
  return 0;
}
```