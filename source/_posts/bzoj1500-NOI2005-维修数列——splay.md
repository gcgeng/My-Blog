---
title: '[bzoj1500][NOI2005]维修数列——splay'
tags: [数据结构, splay]
date: 2017-03-03 11:05:00
---

#题目
![](http://www.lydsy.com/JudgeOnline/images/1500_1.jpg)
#题解
这道题可以说是数列问题的大BOSS，也算是这一周来学习splay等数据结构的一个总结。
我们一个一个地看这些操作。
对于操作1，我们首先建一棵子树，直接接上原树即可。
对于操作2，我们找到区间，不能直接取消连接关系，而是要一个一个的删除以回收空间。我们把已经删除的节点用一个栈保存起来。
对于操作3，我们找到区间，打标记即可。
对于操作4，同操作3。
对于操作5，我们找到区间，直接调用sum值即可。
对于操作6，我们对于每个节点对应的子树区间维护最大子段和，最大从左边开始的子段和，最大从右边开始的字段和，根据《算法竞赛入门经典——训练指南》P201。
我们设最大字段和为ma, 最大左字段和为la，右为ra，那么，我们可以YY一下合并方式。
$$x_{ma} = Max(Max(x_{left_{ma}}, x_{right_{ma}}), x_{left_{ra}}+x_data+Max(x_{right_{la}}, 0))$$
$$x_{la} = Max(x_{left_{la}}, x_{left_{sum}}+x_{data}+x_{right_{la}})$$
$$x_{ra} = Max(x_{right_{ra}}, x_{right_{sum}}+x_{data}+x_{left_{ra}})$$
这样，我们就完成了对于操作6的维护。
实现时要注意几个问题：
1\. 一旦是从上往下走，就要pushdown。考虑到splay都会在find之后调用，那么我们就直接在find的过程中顺便pushdown即可。
2\. 一旦是从下往上回溯，修改了子节点的值，就要update。

顺便一提的是，这道题没有卡int，非常的良心（看来2005年的出题人就是良心啊（逃
#代码
```c++
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <stack>
#define l(x) ch[(x)][0]
#define r(x) ch[(x)][1]
#ifdef D
const int maxn = 50;
#else
const int maxn = 500000 << 1;
#endif
const int inf = 0x3f3f3f;
int ch[maxn][2], fa[maxn];
int size[maxn], data[maxn], sum[maxn], la[maxn], ra[maxn], ma[maxn], cov[maxn],
    a[maxn];
bool rev[maxn];
int n, m, sz, rt;
std::stack<int> st;
void update(int x) {
  if (!x)
    return;
  la[x] = std::max(la[l(x)], sum[l(x)] + data[x] + std::max(0, la[r(x)]));
  ra[x] = std::max(ra[r(x)], sum[r(x)] + data[x] + std::max(0, ra[l(x)]));
  ma[x] = std::max(std::max(ma[l(x)], ma[r(x)]),
                   data[x] + std::max(0, ra[l(x)]) + std::max(0, la[r(x)]));
  sum[x] = sum[l(x)] + sum[r(x)] + data[x];
  size[x] = size[l(x)] + size[r(x)] + 1;
}
void reverse(int x) {
  if (!x)
    return;
  std::swap(ch[x][0], ch[x][1]);
  std::swap(la[x], ra[x]);
  rev[x] ^= 1;
}
void recover(int x, int v) {
  if (!x)
    return;
  data[x] = cov[x] = v;
  sum[x] = size[x] * v;
  la[x] = ra[x] = ma[x] = std::max(v, sum[x]);
}
void pushdown(int x) {
  if (!x)
    return;
  if (rev[x]) {
    reverse(ch[x][0]);
    reverse(ch[x][1]);
    rev[x] = 0;
  }
  if (cov[x] != -inf) {
    recover(ch[x][0], cov[x]);
    recover(ch[x][1], cov[x]);
    cov[x] = -inf;
  }
}
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  if (z)
    ch[z][ch[z][1] == y] = x;
  update(y);
  update(x);
}
void splay(int x, int aim = 0) {
  for (int y; (y = fa[x]) != aim; zig(x))
    if (fa[y] != aim)
      zig((ch[fa[y]][0] == y) == (ch[y][0] == x) ? y : x);
  if (aim == 0)
    rt = x;
  update(x);
}
int pick() {
  if (!st.empty()) {
    int x = st.top();
    st.pop();
    return x;
  } else
    return ++sz;
}
int setup(int x) {
  int t = pick();
  data[t] = a[x];
  cov[t] = -inf;
  rev[t] = false;
  sum[t] = 0;
  la[t] = ra[t] = ma[t] = -inf;
  size[t] = 1;
  return t;
}
int build(int l, int r) {
  int mid = (l + r) >> 1, left = 0, right = 0;
  if (l < mid)
    left = build(l, mid - 1);
  int t = setup(mid);
  if (r > mid)
    right = build(mid + 1, r);
  if (left) {
    ch[t][0] = left, fa[left] = t;
  } else
    size[ch[t][0]] = 0;
  if (right) {
    ch[t][1] = right, fa[right] = t;
  } else
    size[ch[t][1]] = 0;
  update(t);
  return t;
}
int find(int k) {
  int x = rt, ans;
  while (x) {
    pushdown(x);
    if (k == size[ch[x][0]] + 1)
      return ans = x;
    else if (k > size[ch[x][0]] + 1) {
      k -= size[ch[x][0]] + 1;
      x = ch[x][1];
    } else
      x = ch[x][0];
  }
  return -1;
}
void del(int &x) {
  if (!x)
    return;
  st.push(x);
  fa[x] = 0;
  del(ch[x][0]);
  del(ch[x][1]);
  la[x] = ma[x] = ra[x] = -inf;
  x = 0;
}
void print(int x) {
  if (!x)
    return;
  if (ch[x][0])
    print(ch[x][0]);
  std::cout << data[x] << ' ';
  if (ch[x][1])
    print(ch[x][1]);
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 2; i <= n + 1; i++)
    scanf("%d", &a[i]);
  a[1] = a[n + 2] = 0;
  ra[0] = la[0] = ma[0] = -inf;
  rt = build(1, n + 2);
  char opt[20];
#ifdef D
//  print(rt);
#endif
  // return 0;
  while (m--) {
    scanf("%s", opt);
    if (opt[0] == 'I') {
      int pos, cnt;
      scanf("%d %d", &pos, &cnt);
      pos++;
      int l = find(pos);
      int r = find(pos + 1);
      splay(l);
      splay(r, rt);
      for (int i = 1; i <= cnt; i++)
        scanf("%d", &a[i]);
      int t = build(1, cnt);
      fa[t] = ch[rt][1];
      ch[r][0] = t;
      update(l);
      update(r);
    }
    if (opt[0] == 'D') {
      int pos, cnt;
      scanf("%d %d", &pos, &cnt);
      pos++;
      int l = find(pos - 1);
      int r = find(pos + cnt);
      splay(l);
      splay(r, rt);
      del(ch[r][0]);
      update(l);
      update(r);
    }
    if (opt[0] == 'M' && opt[2] == 'K') {
      int x, y, z;
      scanf("%d %d %d", &x, &y, &z);
      x++;
      int l = find(x - 1);
      int r = find(x + y);
      splay(l);
      splay(r, rt);
      recover(ch[r][0], z);
    }
    if (opt[0] == 'R') {
      int x, y;
      scanf("%d %d", &x, &y);
      x++;
      int l = find(x - 1);
      int r = find(x + y);
      splay(l);
      splay(r, rt);
      reverse(ch[r][0]);
    }
    if (opt[0] == 'G') {
      int x, y;
      scanf("%d %d", &x, &y);
      x++;
      int l = find(x - 1);
      int r = find(x + y);
      splay(l);
      splay(r, rt);
      int ans = sum[ch[r][0]];
      printf("%d\n", ans);
    }
    if (opt[0] == 'M' && opt[2] == 'X') {
      int l, r, x = rt;
      while (ch[x][0])
        x = ch[x][0];
      l = x;
      x = rt;
      while (ch[x][1])
        x = ch[x][1];
      r = x;
      splay(l);
      splay(r, rt);
      int ans = ma[ch[r][0]];
      printf("%d\n", ans);
    }
#ifdef D
// print(rt);
#endif
  }
  return 0;
}
```