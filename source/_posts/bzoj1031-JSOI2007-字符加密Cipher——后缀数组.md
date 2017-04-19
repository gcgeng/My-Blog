---
title: '[bzoj1031][JSOI2007]字符加密Cipher——后缀数组'
tags: [字符串, 后缀数组]
date: 2017-03-13 14:50:00
---

# Brief Description
给定一个长度为n的字符串，你需要对其进行加密。
1\. 把字符串围成一个环
2\. 显然从任意一个位置开始都可以有一个长度为n的串
3\. 把产生的n个串按字典序排序，把这n个串的最后一个字符顺接起来就得到了加密后的串。

<!--more-->

# Algorithm Design
看到环的题目，可以想到破环为链，我们把原串复制一份贴在后面，求这个串的SA，可以知道这个新串的SA扫一遍就得到了解。
然后这个题就做完了。

# Code
```cpp
#include <cstdio>
#include <cstring>
const int maxn = 200010;
char ch[maxn];
int a[maxn], n, k;
int v[maxn], sa[2][maxn], rank[2][maxn];
void init() {
  scanf("%s", ch + 1);
  n = strlen(ch + 1);
  for (int i = 1; i <= n; i++) {
    a[i] = (int)ch[i];
    a[i + n] = a[i];
    ch[i + n] = ch[i];
  }
  n <<= 1;
}
void calcsa(int sa[maxn], int rank[maxn], int SA[maxn], int RANK[maxn]) {
  for (int i = 1; i <= n; i++)
    v[rank[sa[i]]] = i;
  for (int i = n; i >= 1; i--)
    if (sa[i] > k)
      SA[v[rank[sa[i] - k]]--] = sa[i] - k;
  for (int i = n - k + 1; i <= n; i++)
    SA[v[rank[i]]--] = i;
  for (int i = 1; i <= n; i++) {
    RANK[SA[i]] = RANK[SA[i - 1]] + (rank[SA[i - 1]] != rank[SA[i]] ||
                                     rank[SA[i - 1] + k] != rank[SA[i] + k]);
  }
}
void work() {
  int p = 0, q = 1;
  for (int i = 1; i <= n; i++)
    v[a[i]]++;
  for (int i = 1; i <= 256; i++)
    v[i] += v[i - 1];
  for (int i = 1; i <= n; i++)
    sa[p][v[a[i]]--] = i;
  for (int i = 1; i <= n; i++)
    rank[p][sa[p][i]] =
        rank[p][sa[p][i - 1]] + (a[sa[p][i]] != a[sa[p][i - 1]]);
  k = 1;
  while (k < n) {
    calcsa(sa[p], rank[p], sa[q], rank[q]);
    p ^= 1;
    q ^= 1;
    k <<= 1;
  }
  for (int i = 1; i <= n; i++) {
    if (sa[p][i] <= n / 2)
      printf("%c", ch[sa[p][i] + n / 2 - 1]);
  }
  printf("\n");
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  init();
  work();
  return 0;
}
```