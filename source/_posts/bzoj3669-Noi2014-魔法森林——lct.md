---
title: '[bzoj3669][Noi2014]魔法森林——lct'
tags: [lct, 数据结构]
date: 2017-03-10 19:32:00
---

# Brief description
给定一个无向图，求从1到n的一条路径使得这条路径上最大的a和b最小。

# Algorithm Design 
以下内容选自[某HN神犇的blog](http://www.cnblogs.com/Robert-Yuan/p/5134896.html)
>双瓶颈的最小生成树的感觉，可以首先按a值排序，然后一条边一条边的加入．
>　　如果之前连接的两点还未连通，那么连上先满足最后连通性的必要．
>　　如果之前连接的两点已经连通，那么就在原来的路径上找到一条b值最大的，然后删掉原来的，加上现在的边，保证最优性的需要．
>　　这样会导致最大的b值的减小，但是如果之前1,n已经连通，也会造成最大的a值的增大
>　　[因为是按a排序，在连通前的操作都是不管a值的，只以最后一次加的边为最大[所以之前的替换操作只会让这个路径更优]，但是连通后，添加的边就会让a值增大[不一定会更优]]，这就需要在多种方案间选出最优．
>　　上面说得很轻巧，现在我们想想怎么完成上述操作．
>　　总共要对每条边处理一次，每次需要连边或者在两点之前的链上找最大值．找到之后有删边的操作．
>　　支持这么多操作的数据结构有什么?[注意我们连接的一定是一棵树[或是一片森林]...不然就浪费了...]
>　　lca似乎不兹瓷啊，因为是动态的，哦，那就是动态树了．
>　　动态树中带边权的怎么处理呢?可以将所有实点的值定为0，连(u,v)边改为连(u,x)和(x,v)，x的值代表这条边的边权．
>　　[p.s]有的同学会觉得我连(u,v)把值记在u上或者v上就可以了...每次splay的时候，只有根节点保留的是在原树中连接上个部分的边权，其它的在splay的时候交换．[<-这一步是可以实现的]
>　　　　有的同学觉得我这样不就可以了么?然而...你还有个东西叫Access()，你每次会将原来本来是链的顶部才能连的边，给了当前splay的根，然后连通之后再splay，鬼才找的到原来的边是什么?..当然上面的＂有的同学＂都是说的笔者..有的大神说不定还是可以不加虚拟边点过的...

# Code
```cpp
#include <algorithm>
#include <cstdio>
int n, m;
const int maxn = 50010;
const int maxm = 100010;
const int maxv = maxn + maxm;
struct edge {
  int u, v, a, b;
} e[maxm << 1];
bool cmp1(edge a, edge b) { return a.a < b.a; }
int f[maxn], val[maxv], fa[maxv], ch[maxv][2], rev[maxv], max[maxv];
int findf(int x) { return f[x] == x ? x : f[x] = findf(f[x]); }
void pushdown(int x) {
  if (rev[x]) {
    rev[x] ^= 1;
    rev[ch[x][0]] ^= 1;
    rev[ch[x][1]] ^= 1;
    std::swap(ch[x][0], ch[x][1]);
  }
}
void update(int x) {
  max[x] = x;
  if (val[max[ch[x][0]]] > val[max[x]])
    max[x] = max[ch[x][0]];
  if (val[max[ch[x][1]]] > val[max[x]])
    max[x] = max[ch[x][1]];
}
bool isroot(int x) { return (ch[fa[x]][0] != x) && (ch[fa[x]][1] != x); }
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  if (!isroot(y))
    ch[z][ch[z][1] == y] = x;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  update(y);
  update(x);
}
void splay(int x) {
  int s[maxv], top = 0;
  s[++top] = x;
  for (int i = x; !isroot(i); i = fa[i])
    s[++top] = fa[i];
  while (top)
    pushdown(s[top--]);
  for (int y; !isroot(x); zig(x))
    if (!isroot(y = fa[x]))
      zig((ch[y][0] == x) == (ch[fa[y]][0] == y) ? y : x);
  update(x);
}
void access(int x) {
  for (int t = 0; x; t = x, x = fa[x]) {
    splay(x);
    ch[x][1] = t;
    update(x);
  }
}
void makeroot(int x) {
  access(x);
  splay(x);
  rev[x] ^= 1;
}
void link(int x, int y) {
  makeroot(x);
  fa[x] = y;
}
void cut(int x, int y) {
  makeroot(x);
  access(y);
  splay(y);
  ch[y][0] = fa[x] = 0;
  update(y);
}
void split(int x, int y) {
  makeroot(y);
  access(x);
  splay(x);
}
int ans = 0x3f3f3f;
int main() {
  freopen("input", "r", stdin);
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++)
    f[i] = i;
  for (int i = 1; i <= m; i++) {
    scanf("%d %d %d %d", &e[i].u, &e[i].v, &e[i].a, &e[i].b);
  }
  std::sort(e + 1, e + 1 + m, cmp1);
  for (int i = 1; i <= m; i++) {
    int u = e[i].u, v = e[i].v, x = findf(u), y = findf(v);
    if (x == y) {
      split(u, v);
      int t = max[u];
      if (val[t] > e[i].b) {
        cut(e[t - n].u, t);
        cut(e[t - n].v, t);
      } else {
        if (findf(1) == findf(n)) {
          split(1, n);
          int t = max[1];
          ans = std::min(ans, e[i].a + val[t]);
        }
        continue;
      }
    }
    if (x != y) {
      f[x] = y;
    }
    val[n + i] = e[i].b;
    max[n + i] = n + i;
    link(u, i + n);
    link(v, i + n);
    if (findf(1) == findf(n)) {
      split(1, n);
      int t = max[1];
      ans = std::min(ans, e[i].a + val[t]);
    }
  }
  printf("%d\n", ans == 0x3f3f3f ? -1 : ans);
}
```