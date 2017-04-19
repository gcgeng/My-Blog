---
title: '[bzoj3277==bzoj3473]出现k次子串计数——广义后缀自动机+STL'
tags: [字符串, 后缀自动机, 广义后缀自动机, STL]
date: 2017-03-15 09:30:00
---

# Brief Description
给定n个字符串，对于每个字符串，您需要求出在所有字符串中出现次数大于等于k次的子串个数。

# Algorithm Design 
先建立一个广义后缀自动机，什么是广义后缀自动机？就是所有主串一起建立的一个后缀自动机。
广义后缀自动机的建立很简单，对于每个串，该怎么增量建立自动机就怎么建立，只不过为每个节点维护一个set保存这个节点的状态在那些字符串中出现过。当一个串增量构建完毕后，将后缀自动机的last指针指向后缀自动机的根即可进行下一发字符串的增量构建，这样就建出来了一发广义后缀自动机。
考虑一个节点，如果他在x个字符串中出现过，那么他的fa指针所指向的节点所代表的状态出现过的次数一定不小于他。
并且我们已经为每个节点维护了一个set来记录在那些字符串中出现过，那么我们只需要自下向上合并set集合即可，在这之前需要整理出parent树的具体形态，然后一遍dfs，逆序处理set的启发式合并即可。
统计答案只需把每个字符串在自动机上跑，跑到一个节点发现出现次数<K就往fa指针那里跳，直到符合条件。这时候贡献的答案就是当前节点的len属性的值了.

# Code
```cpp
#include <cstdio>
#include <cstdlib>
#include <iostream>
#include <set>
#include <string>
const int maxn = 200010;
using std::set;
using std::string;
#define ll long long
set<int> d[maxn];
set<int>::iterator it;
int n, K, tot = 1, head[maxn], sum[maxn];
struct edge {
  int to, next;
} e[maxn * 6];
string str[maxn];
struct Suffix_Automaton {
  int trans[maxn][26], len[maxn], sz;
  int fa[maxn], last, root;
  void init() {
    tot = 0;
    last = root = ++sz;
  }
  void add(int c, int id) {
    int p = last, np = last = ++sz;
    len[np] = len[p] + 1;
    d[np].insert(id);
    while (p && !trans[p][c])
      trans[p][c] = np, p = fa[p];
    if (!p)
      fa[np] = root;
    else {
      int q = trans[p][c];
      if (len[q] == len[p] + 1)
        fa[np] = q;
      else {
        int nq = ++sz;
        len[nq] = len[p] + 1;
        fa[nq] = fa[q];
        for (int i = 0; i < 26; i++)
          trans[nq][i] = trans[q][i];
        fa[q] = fa[np] = nq;
        while (trans[p][c] == q)
          trans[p][c] = nq, p = fa[p];
      }
    }
  }
  void print() {
    for (int i = 1; i <= sz; i++) {
      std::cout << fa[i] << ' ';
    }
    std::cout << std::endl;
    for (int i = 1; i <= sz; i++)
      printf("%d ", sum[i]);
    printf("\n");
  }
} sam;
void dfs(int x) {
  for (int i = head[x]; i; i = e[i].next) {
    int v = e[i].to;
    dfs(v);
    for (it = d[v].begin(); it != d[v].end(); it++)
      d[x].insert(*it);
  }
  sum[x] = d[x].size();
}
void add_edge(int from, int to) {
  e[++tot].to = to;
  e[tot].next = head[from];
  head[from] = tot;
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &K);
  sam.init();
  for (int i = 1; i <= n; i++) {
    std::cin >> str[i];
    int len = str[i].length();
    for (int j = 0; j < len; j++)
      sam.add(str[i][j] - 'a', i);
    sam.last = sam.root;
  }
  for (int i = 1; i <= sam.sz; i++)
    if (sam.fa[i])
      add_edge(sam.fa[i], i);
  dfs(sam.root);
  // sam.print();
  if (K > n) {
    for (int i = 1; i <= n; i++)
      printf("0 ");
    return 0;
  }
  for (int i = 1; i <= n; i++) {
    ll ans = 0;
    int now = sam.root, len = str[i].length();
    for (int j = 0; j < len; j++) {
      now = sam.trans[now][str[i][j] - 'a'];
      while (sum[now] < K)
        now = sam.fa[now];
      ans += sam.len[now];
    }
    printf("%lld ", ans);
  }
  return 0;
}
```