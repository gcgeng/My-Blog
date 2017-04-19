---
title: '[bzoj1072] [SCOI2007]排列perm'
tags: [动态规划, 状压DP]
date: 2016-12-26 18:09:00
---

有一种暴力算法就是直接枚举。
正解就是状压dp
令f[i][j]：i：使用的数位的状态j:当前的模数
边界：f[0][0] = 1;
f[i|1<<k][j*10+k % n] += f[i][j] | !(i&(1<<k))
答案就是f[i<<len-1][0]
进行了一些重复处理。
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 15;
int main() {
  //  freopen("input", "r", stdin);
  int T;
  scanf("%d", &T);
  while (T--) {
    int f[1024 + 5][1000 + 10];
    char s[maxn];
    int n;
    long long Ans;
    scanf("%s %d", s, &n);
    int a[maxn], Cnt[maxn];
    int len = strlen(s);
    f[0][0] = 1;
    for (int i = 0; i < len; i++) {
      a[i] = s[i] - '0';
      ++Cnt[a[i]];
    }
    for (int i = 0; i < 1 << len; i++) {
      for (int j = 0; j < len; j++) {
        for (int k = 0; k < len; k++) {
          if (!(i & 1 << k)) {
            f[i | (1 << k)][(j * 10 + k) % n] += f[i][j];
          }
        }
      }
    }
    Ans = f[(1 << len) - 1][0];
    for (int i = 0; i <= 9; i++) {
      for (int j = 1; j <= Cnt[i]; j++) {
        Ans /= j;
      }
    }
    printf("%lld\n", Ans);
    /*  char s[maxn];
      int n;
      scanf("%s %d", s, &n);
      int a[maxn];
      int len = strlen(s);
      for (int i = 0; i < len; i++)
        a[i] = s[i] - '0';
      sort(a, a + len);
      set<long long> st;
      st.clear();
      int ans = 0;
      while (1) {
        long long num = 0;
        for (int j = 0; j < len; j++)
          num = num * 10 + a[j];
        if (st.count(num))
          continue;
        else if (!(num % n)) {
          st.insert(num);
          ans++;
        }
        if (!next_permutation(a, a + len))
          break;
      }
      printf("%d\n", ans);*/
  }
  return 0;
}

```