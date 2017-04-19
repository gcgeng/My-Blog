---
title: '[bzoj1030][JSOI2007]文本生成器——AC自动机'
tags: [字符串, AC自动机]
date: 2017-03-13 11:45:00
---

# Brief Description
给定一些模式串，您需要求出满足以下要求的字符串的个数。
1\. 长度为m
2\. 包含任意一个模式串

<!--more-->

# Algorithm Design 
>以下内容来自[神犇博客](http://blog.csdn.net/thchuan2001/article/details/57463291)
>首先运用补集转换，转而求不含这些串的个数，最后用26^M减掉就行
>根据输入的字符串建立AC自动机
>dp[i][j]表示当前考虑了i位，当前停留在AC自动机的j号节点
>每一次可以由dp[i][j]转移到dp[i+1][k]，k是枚举第i+1为后作为j的儿子在AC自动机上的编号
>枚举k，就是第i+1为填什么，然后进行下列操作：
>首先看看这位能不能填k，判断方法是从j开始向fail[j]跳，看是不是有一个j有一个k儿子，并且k儿子上还有结束标记，只要有一个就证明如果i+1位填k就会让整个字符串出现AC自动机上的字符串，所以不能填k
>如果能放，再看看要修改哪个dp数组。
>还是从j开始向fail[j]跳，如果j有k这个儿子就直接修改dp[i+1][j的k儿子]就好
>每次修改要对修改目标加上dp[i][j]
>答案是所有dp[m][x]（x是所有AC自动机上的节点）的和

# Code
```cpp
#include <cstdio>
#include <cstring>
#include <queue>
#define mod 10007
const int maxn = 6010;
int a[maxn][27], f[101][maxn], point[maxn];
int n, m, sz = 1;
char s[101];
bool leaf[maxn];
void insert(char s[101]) {
  int now = 1, c;
  for (int i = 0; i < strlen(s); i++) {
    c = s[i] - 'A' + 1;
    if (a[now][c])
      now = a[now][c];
    else
      now = a[now][c] = ++sz;
  }
  leaf[now] = 1;
}
void ac() {
  std::queue<int> q;
  q.push(1);
  point[1] = 0;
  while (!q.empty()) {
    int u = q.front();
    q.pop();
    for (int i = 1; i <= 26; i++) {
      if (!a[u][i])
        continue;
      int k = point[u];
      while (!a[k][i])
        k = point[k];
      point[a[u][i]] = a[k][i];
      if (leaf[a[k][i]])
        leaf[a[u][i]] = 1;
      q.push(a[u][i]);
    }
  }
}
int main() {
#ifndef ONLINE_JUDGE
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= 26; i++)
    a[0][i] = 1;
  for (int i = 1; i <= n; i++) {
    scanf("%s", s);
    insert(s);
  }
  ac();
  f[0][1] = 1;
  for (int x = 1; x <= m; x++)
    for (int i = 1; i <= sz; i++) {
      if (leaf[i] || !f[x - 1][i])
        continue;
      for (int j = 1; j <= 26; j++) {
        int k = i;
        while (!a[k][j])
          k = point[k];
        f[x][a[k][j]] = (f[x][a[k][j]] + f[x - 1][i]) % mod;
      }
    }
  int ans1 = 0, ans2 = 1;
  for (int i = 1; i <= m; i++)
    ans2 = (ans2 * 26) % mod;
  for (int i = 1; i <= sz; i++) {
    if (!leaf[i])
      ans1 = (ans1 + f[m][i]) % mod;
  }
  // printf("%d %d\n", ans2, ans1);
  printf("%d\n", (ans2 - ans1 + mod) % mod);
  return 0;
}
```