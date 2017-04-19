---
title: '[bzoj2049][Sdoi2008]Cave 洞穴勘测——lct'
tags: [数据结构, lct]
date: 2017-03-08 18:08:00
---

#Brief Description
给定一个森林，您需要支持两种操作：
1\. 链接两个节点。
2\. 断开两个节点之间的链接。

#Algorithm Design 
对于树上的操作，我们现在已经有了树链剖分可以处理这些问题。然而树链剖分不支持动态维护树上的拓扑结构。所以我们需要Link-Cut Tree(lct)来解决这种动态树问题。顺带一提的是，动态树也是Tarjan发明的。
首先我们介绍一个概念：Preferred path（实边），其他的边都是虚边。我们使用splay来实时地维护这条路径。
lct的核心操作是access。access操作可以把虚边变为实边，通过改变splay的拓扑结构来维护实边。
有了这个数据结构，我们依次来考虑两个操作。
对于链接两个节点，我们需要首先把x节点变为他所在树的根节点，然后直接令fa[x] = y即可。
怎样换根呢？稍微思考一下可以发现，我们直接把从根到他的路径反转即可。
对于第二种操作，我们直接断开拓扑关系即可。
另外实现的时候要注意，splay的根节点的父亲是他的上一个节点。所以zig和splay的写法应该格外注意。

#Code
```cpp
#include <algorithm>
#include <cctype>
#include <cstdio>
#include <stack>
using std::stack;
const int maxn = 10005;
inline int read() {
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
int n, m;
int fa[maxn], ch[maxn][2];
bool rev[maxn];
inline bool isroot(int x) { return ch[fa[x]][0] != x && ch[fa[x]][1] != x; }
void pushdown(int k) {
  if (rev[k]) {
    rev[k] = 0;
    rev[ch[k][0]] ^= 1;
    rev[ch[k][1]] ^= 1;
    std::swap(ch[k][0], ch[k][1]);
  }
}
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  if (!isroot(y))
    ch[z][ch[z][1] == y] = x;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
}
void splay(int x) {
  stack<int> st;
  st.push(x);
  for (int i = x; !isroot(i); i = fa[i])
    st.push(fa[i]);
  while (!st.empty()) {
    pushdown(st.top());
    st.pop();
  }
  for (int y = fa[x]; !isroot(x); zig(x), y = fa[x])
    if (!isroot(y))
      zig((ch[fa[y]][0] == y) == (ch[y][0] == x) ? y : x);
}
void access(int x) {
  int t = 0;
  while (x) {
    splay(x);
    ch[x][1] = t;
    t = x;
    x = fa[x];
  }
}
void rever(int x) {
  access(x);
  splay(x);
  rev[x] ^= 1;
}
void link(int x, int y) {
  rever(x);
  fa[x] = y;
  splay(x);
}
void cut(int x, int y) {
  rever(x);
  access(y);
  splay(y);
  ch[y][0] = fa[x] = 0;
}
int find(int x) {
  access(x);
  splay(x);
  int y = x;
  while (ch[y][0])
    y = ch[y][0];
  return y;
}
int main() {
  #ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
  #endif
  char ch[10];
  int x, y;
  n = read(), m = read();
  for (int i = 1; i <= m; i++) {
    scanf("%s", ch);
    x = read();
    y = read();
    if (ch[0] == 'C')
      link(x, y);
    else if (ch[0] == 'D')
      cut(x, y);
    else {
      if (find(x) == find(y))
        printf("Yes\n");
      else
        printf("No\n");
    }
  }
}
```