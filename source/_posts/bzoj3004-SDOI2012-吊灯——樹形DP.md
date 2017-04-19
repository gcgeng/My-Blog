---
title: '[bzoj3004][SDOI2012]吊灯——樹形DP'
tags: [动态规划, 树形DP]
date: 2017-03-19 09:36:00
---

# Brief Description
給定一棵樹, 判斷是否可以將其分成$\frac{n}{k}$個聯通塊, 其中每個聯通塊的大小均爲k.

# Algorithm Design
我們有一個結論: k可行iff存在$\frac{n}{k}$個點, 以這些點爲根的子樹大小爲k或k的倍數.
讀者可以自行yy一下證明.
有了這個結論之後, 我們可以算出每個size, 用一個桶統計一下就好了.

# Code
```c++
#include <algorithm>
#include <cctype>
#include <cstdio>
#include <cstring>
const int maxn = 1200000;
int fa[maxn], n, divide[maxn], size[maxn], f[maxn], tot = 0;
void fuck(int n) {
  int i;
  for (i = 1; i * i < n; i++) {
    if (n % i == 0) {
      divide[tot++] = i;
      divide[tot++] = n / i;
    }
  }
  if (i * i == n)
    divide[tot++] = i;
  std::sort(divide, divide + tot);
}
int main() {
  //  freopen("sdoi12_divide.in", "r", stdin);
  //  freopen("sdoi12_divide.out", "w", stdout);
  scanf("%d", &n);
  char ch = getchar();
  int cnt = 0;
  while (cnt < (n - 1)) {
    int x = 0;
    while (!isdigit(ch))
      ch = getchar();
    while (isdigit(ch)) {
      x = x * 10 + ch - '0';
      ch = getchar();
    }
    cnt++;
    fa[cnt + 1] = x;
  }
  fuck(n);
  for (int T = 1; T <= 10; T++) {
    printf("Case #%d:\n", T);
    memset(size, 0, sizeof(size));
    memset(f, 0, sizeof(f));
    for (int i = 2; i <= n; i++) {
      if (T != 1) {
        fa[i] = (fa[i] + 19940105) % (i - 1) + 1;
      }
    }
    for (int i = n; i; i--)
      size[fa[i]] += ++size[i];
    for (int i = 1; i <= n; i++)
      f[size[i]]++;
    for (int i = 0; i < tot; i++) {
      int tmp = 0;
      for (int j = divide[i]; j <= n; j += divide[i])
        tmp += f[j];
      if (tmp == n / divide[i]) {
        printf("%d\n", divide[i]);
      }
    }
  }
}
```