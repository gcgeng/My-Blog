---
title: '[bzoj4567][Scoi2016]背单词-Trie+贪心+模型转化'
tags: [模型转化, Trie, 贪心]
date: 2017-03-23 22:01:00
---

# Brief Description

* 给你N个互不相同的字符串，记$S_i$为第i个字符串，现在要求你指定N个串的出现顺序，我们用$V_i$表示第i个字符串是第几个出现的，则V为1到N的一个排列。我们希望你指定的出现顺序可以使总代价最小
* 一个出现顺序的代价的计算方法如下依次考虑第i个串$S_i$对代价的贡献：
  * 假如存在一个串$S_j$, $S_j$为$S_i$的后缀：
    * 假如存在一个串$S_j$, $S_j$为$S_i$的后缀，且$V_j>V_i$则第i个串对代价的贡献为$N\times N$
    * 否则，记$P_i$为所有满足$S_j$为$S_i$的后缀的j中$V_j$的最大值，第i个串对代价的贡献为$V_i-P_i$
  * 否则，如果不存在一个串$S_j$，使得$S_j$为$S_i$的后缀，则第i个串对代价的贡献为$V_i$
  * 你需要输出这个最小的总代价.

# Algorithm Design

首先可以证明第一个条件是没有用处的.

然后我们视后缀为一种偏序关系, 那么字符串就构成了一颗树, 那么问题就转化成了给树上每个节点标号使得他的标号减去他的父亲的标号的和最小.

这是一个经典的贪心问题. 我们选择每次都选择最小的size子树进行标号, 不难证明这样做的correctness.

# Code

```c++
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <vector>
#define ll long long
#define pa std::pair<int, int>
const int maxn = 100000 + 100;
const int maxlen = 520000 + 100;
int n;
struct edge {
  int to, next;
} e[maxn << 1];
char str[maxlen];
int rt = 1, sz = 1, ch[maxlen][26], len, id[maxlen], tot, head[maxn],
    size[maxn], f[maxn], cnt, fa[maxn];
std::vector<pa> v[maxn << 1];
inline void ins(int id) {
  int p = rt;
  for (int i = len; i >= 1; i--) {
    int x = str[i] - 'a';
    if (!ch[p][x]) {
      ch[p][x] = ++sz;
    }
    p = ch[p][x];
  }
  ::id[p] = id;
}
inline void add_edge(int u, int v) {
  e[++tot].to = v;
  e[tot].next = head[u];
  fa[v] = u;
  head[u] = tot;
}
void dfs(int x, int fa) {
  if (id[x]) {
    add_edge(fa, id[x]);
    fa = id[x];
  }
  for (int i = 0; i < 26; i++) {
    if (ch[x][i])
      dfs(ch[x][i], fa);
  }
}
void dfs2(int x) {
  size[x] = 1;
  for (int i = head[x]; i; i = e[i].next) {
    int y = e[i].to;
    dfs2(y);
    v[x].push_back(std::make_pair(size[y], y));
    size[x] += size[y];
  }
  std::sort(v[x].begin(), v[x].end());
}
void getf(int x) {
  if (x)
    f[x] = ++cnt;
  for (int i = 0; i < v[x].size(); i++)
    getf(v[x][i].second);
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d", &n);
  for (int i = 1; i <= n; i++) {
    scanf("%s", str + 1);
    len = strlen(str + 1);
    ins(i);
  }
  dfs(1, 0);
  dfs2(0);
  getf(0);
  ll ans = 0;
  for (int i = 1; i <= n; i++)
    ans += f[i] - f[fa[i]];
  printf("%lld", ans);
}

```