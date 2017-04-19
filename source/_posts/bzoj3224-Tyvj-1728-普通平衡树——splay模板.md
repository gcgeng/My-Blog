---
title: '[bzoj3224]Tyvj 1728 普通平衡树——splay模板'
tags: [数据结构, splay]
date: 2017-02-28 09:38:00
---

# 题目
你需要写一种数据结构支援以下操作。
1\. 插入元素。
2\. 删除元素。
3\. 查询元素的排名。
4\. 查询第k小的元素。
5\. 查询元素前趋。
6\. 查询元素后继。

# 题解
BBST裸题。

# 代码
```cpp
#include <cstdio>
#include <iostream>
#define ll long long
using namespace std;
#ifdef D
const int maxn = 100;
#else
const int maxn = 100005;
#endif
int ch[maxn][2], fa[maxn], size[maxn], data[maxn], cnt[maxn];
int rt, n, m;
ll sz = 0;
int read() {
  int x = 0, f = 1;
  char ch = getchar();
  while (ch < '0' || ch > '9') {
    if (ch == '-')
      f = -1;
    ch = getchar();
  }
  while (ch >= '0' && ch <= '9') {
    x = x * 10 + ch - '0';
    ch = getchar();
  }
  return x * f;
}
inline void update(int x) {
  size[x] = size[ch[x][0]] + size[ch[x][1]] + cnt[x];
}
inline void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  if (z)
    ch[z][ch[z][1] == y] = x;
  update(y);
}
inline void splay(int x, int aim = 0) { //调整x成为aim的儿子
  for (int y; (y = fa[x]) != aim; zig(x)) {
    if (fa[y] != aim)
      zig((ch[y][0] == x) == (ch[fa[y]][0] == y) ? y : x);
  }
  if (aim == 0)
    rt = x;
  update(x);
}
inline void insert(int v) {
  int x = rt;
  if (rt == 0) {
    rt = x = ++sz;
    data[x] = v;
    fa[x] = ch[x][0] = ch[x][1] = 0;
    size[x] = cnt[x] = 1;
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
      fa[y] = x;
      ch[y][0] = ch[y][1] = 0;
      size[y] = cnt[y] = 1;
      x = y;
      break;
    }
    x = y;
  }
  splay(x);
}
int rank(int v) {
  int ret = 0, ans, x = rt;
  while (x) {
    if (v > data[x]) {
      ret += size[ch[x][0]] + cnt[x];
      x = ch[x][1];
    } else {
      if (v == data[x]) {
        return ret + size[ch[x][0]] + 1;
      }
      x = ch[x][0];
    }
  }
}
int kth(int v) {
  int x = rt;
  while (x) {
    if (v >= size[ch[x][0]] + 1 && v <= size[ch[x][0]] + cnt[x])
      return data[x];
    if (v > size[ch[x][0]] + cnt[x]) {
      v -= size[ch[x][0]] + cnt[x];
      x = ch[x][1];
    } else
      x = ch[x][0];
  }
}
int pre(int v) {
  int ans;
  for (int y = rt; y;)
    if (data[y] < v)
      ans = data[y], y = ch[y][1];
    else
      y = ch[y][0];
  return ans;
}
int succ(int v) {
  int ans;
  for (int y = rt; y;)
    if (data[y] <= v)
      y = ch[y][1];
    else
      ans = data[y], y = ch[y][0];
  return ans;
}
inline void del(int v) {
  int x = rt;
  while (x) {
    size[x]--;
    if (data[x] == v) {
      cnt[x]--;
      break;
    }
    int &y = ch[x][v >= data[x]];
    x = y;
  }
  if (cnt[x] != 0)
    return;
  splay(x);
  if (ch[x][0] == 0) {
    rt = ch[x][1];
    fa[rt] = 0;
    return;
  }
  if (ch[x][1] == 0) {
    rt = ch[x][0];
    fa[rt] = 0;
    return;
  }
  x = ch[rt][0];
  while (ch[x][1])
    x = ch[x][1];
  splay(x, rt);
  ch[x][1] = ch[rt][1];
  fa[ch[x][1]] = x;
  size[x] = size[rt] - 1;
  fa[x] = 0;
  rt = x;
}
int main() {
#ifdef DEBUG
  freopen("input", "r", stdin);
#endif
  scanf("%d", &n);
  rt = sz = 0;
  while (n--) {
    int opt = read();
    if (opt == 1) {
      int x = read();
      insert(x); // O(log n)
    } else if (opt == 2) {
      int x = read();
      del(x); // O(log n)
    } else if (opt == 3) {
      int k = read();
      printf("%d\n", rank(k));
    } else if (opt == 4) {
      int k = read();
      printf("%d\n", kth(k));
    } else if (opt == 5) {
      int x = read();
      printf("%d\n", pre(x));
    } else if (opt == 6) {
      int x = read();
      printf("%d\n", succ(x));
    }
  }
}
```