---
title: '[bzoj1009][HNOI2008]GT考试——KMP+矩阵乘法'
tags: [字符串, KMP, 矩阵乘法]
date: 2017-03-13 09:26:00
---

# Brief Description

给定一个长度为m的禁止字符串，求出长度为n的字符串的个数，满足：
这个字符串的任何一个字串都不等于给定字符串。
本题是POJ3691的弱化版本。

# Algorithm Design
考察使用动态规划（递推）。
记录f[i][j]为当前已经做了i个字符，这个字符串长度为j的后缀与禁止字符串的前缀匹配，的字符串个数。
如果我们知道对于一个后缀而言加入一个字符之后可以转移到的状态我们就可以转移了。
我们可以知道KMP算法做的就是这样的事情。
又因為他满足矩阵乘法的一般方法，所以使用矩阵乘法加速。

# Code
```cpp
#include <cstdio>
const int maxn = 25;
int p[maxn], a[maxn][maxn], b[maxn][maxn];
int n, m, mod;
char ch[maxn];
void mul(int a[maxn][maxn], int b[maxn][maxn], int ans[maxn][maxn]) {
  int tmp[maxn][maxn];
  for (int i = 0; i < m; i++)
    for (int j = 0; j < m; j++) {
      tmp[i][j] = 0;
      for (int k = 0; k < m; k++)
        tmp[i][j] = (tmp[i][j] + a[i][k] * b[k][j]) % mod;
    }
  for (int i = 0; i < m; i++) {
    for (int j = 0; j < m; j++)
      ans[i][j] = tmp[i][j];
  }
}
int main() {
  // freopen("input", "r", stdin);
  scanf("%d %d %d", &n, &m, &mod);
  scanf("%s", ch + 1);
  int j = 0;
  for (int i = 2; i <= m; i++) {
    while (j > 0 && ch[j + 1] != ch[i])
      j = p[j];
    if (ch[j + 1] == ch[i])
      j++;
    p[i] = j;
  }
  for (int i = 0; i < m; i++) {
    for (int j = 0; j <= 9; j++) {
      int t = i;
      while (t > 0 && ch[t + 1] - '0' != j)
        t = p[t];
      if (ch[t + 1] - '0' == j)
        t++;
      if (t != m)
        b[t][i] = (b[t][i] + 1) % mod;
    }
  }
  for (int i = 0; i < m; i++)
    a[i][i] = 1;
  while (n) {
    if (n & 1)
      mul(a, b, a);
    mul(b, b, b);
    n >>= 1;
  }
  int sum = 0;
  for (int i = 0; i < m; i++)
    sum = (sum + a[i][0]) % mod;
  printf("%d\n", sum);
  return 0;
}
```