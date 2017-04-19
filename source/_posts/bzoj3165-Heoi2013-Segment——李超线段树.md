---
title: '[bzoj3165][Heoi2013]Segment——李超线段树'
tags: [数据结构, 李超线段树]
date: 2017-03-02 08:14:00
---

# 题目大意
要求在平面直角坐标系下维护两个操作：
1.在平面上加入一条线段。记第i条被插入的线段的标号为i。
2.给定一个数k,询问与直线 x = k相交的线段中，交点最靠上的线段的编号。
# 题解
和上一个题几乎一样。
# 代码
```c++
#include <algorithm>
#include <cctype>
#include <cstdio>
int read() {
  int x = 0, f = 1;
  char ch = getchar();
  while (!isdigit(ch)) {
    if (ch == '-')
      f = -1;
    ch = getchar();
  }
  while (isdigit(ch)) {
    x = x * 10 + ch - '0';
    ch = getchar();
  }
  return x * f;
}
int N, M;
struct line {
  double k, b;
  int id;
  line(int x0 = 0, int y0 = 0, int x1 = 0, int y1 = 0, int ID = 0) {
    id = ID;
    if (x0 == x1)
      k = 0, b = std::max(y0, y1);
    else
      k = (double)(y0 - y1) / (x0 - x1), b = (double)y0 - k * x0;
  }
  double getf(int x) { return k * x + b; };
};
bool cmp(line a, line b, int x) {
  if (!a.id)
    return 1;
  return a.getf(x) != b.getf(x) ? a.getf(x) < b.getf(x) : a.id < b.id;
}
const int maxn = 50010;
line t[maxn << 2];
line query(int k, int l, int r, int x) {
  if (l == r)
    return t[k];
  int mid = (l + r) >> 1;
  line tmp;
  if (x <= mid)
    tmp = query(k << 1, l, mid, x);
  else
    tmp = query(k << 1 | 1, mid + 1, r, x);
  return cmp(t[k], tmp, x) ? tmp : t[k];
}
void insert(int k, int l, int r, line x) {
  if (!t[k].id)
    t[k] = x;
  if (cmp(t[k], x, l))
    std::swap(t[k], x);
  if (l == r || t[k].k == x.k)
    return;
  int mid = (l + r) >> 1;
  double X = (t[k].b - x.b) / (x.k - t[k].k);
  if (X < l || X > r)
    return;
  if (X <= mid)
    insert(k << 1, l, mid, t[k]), t[k] = x;
  else
    insert(k << 1 | 1, mid + 1, r, x);
}
void Insert(int k, int l, int r, int x, int y, line v) {
  if (x <= l && r <= y) {
    insert(k, l, r, v);
    return;
  }
  int mid = (l + r) >> 1;
  if (x <= mid)
    Insert(k << 1, l, mid, x, y, v);
  if (y > mid)
    Insert(k << 1 | 1, mid + 1, r, x, y, v);
}
#define p1 39989
#define p2 1000000000
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  M = read();
  N = 50000;
  int las, cnt = 0;
  while (M--) {
    int opt = read();
    if (opt == 0) {
      int x = read();
      x = (x + las - 1) % p1 + 1;
      las = query(1, 1, N, x).id;
      printf("%d\n", las);
    }
    if (opt == 1) {
      int x0 = read(), y0 = read(), x1 = read(), y1 = read();
      x0 = (x0 + las - 1) % p1 + 1;
      y0 = (y0 + las - 1) % p2 + 1;
      x1 = (x1 + las - 1) % p1 + 1;
      y1 = (y1 + las - 1) % p2 + 1;
      if (x0 > x1)
        std::swap(x0, x1), std::swap(y0, y1);
      Insert(1, 1, N, x0, x1, line(x0, y0, x1, y1, ++cnt));
    }
  }
  return 0;
}
```