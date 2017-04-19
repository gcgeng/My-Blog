---
title: '[bzoj1026][SCOI2009]windy数——数位dp'
tags: [动态规划, 数位DP]
date: 2017-02-16 21:26:00
---

# 题目
求[a,b]中的windy数个数。
windy数指的是任意相邻两个数位上的数至少相差2的数，比如135是，134不是。
# 题解
感觉这个题比刚才做的那个简单多了。。。这个才真的应该是数位dp入门题嘛。
方程就是
$$f[i][j] = \sum f[i-1][k]$$
随便搞一搞就好辣。
# 代码
```
#include <algorithm>
#include <cstdio>
#define ll long long
using namespace std;
const int maxn = 13;
ll f[maxn][maxn], a[maxn], cnt, len;
void init(ll n) {
  len = 0;
  while (n) {
    a[++len] = n % 10;
    n /= 10;
  }
}
ll calc(ll x) {
  ll ans = 0;
  int flag = 1;
  if (!x)
    return 0;
  init(x);
  for (int i = 1; i < len; i++)
    for (int j = 1; j < 10; j++)
      ans += f[i][j];
  for (int j = 1; j < a[len]; j++)
    ans += f[len][j];
  for (int i = len - 1; i; i--) {
    for (int j = 0; j < a[i]; j++)
      if (abs(a[i + 1] - j) >= 2)
        ans += f[i][j];
    if (abs(a[i + 1] - a[i]) < 2) {
      flag = 0;
      break;
    }
  }
  if (flag)
    ans++;
  return ans;
}
int main() {
  for (int i = 0; i <= 9; i++)
    f[1][i] = 1;
  for (int i = 2; i <= 12; i++) {
    for (int j = 0; j <= 9; j++) {
      for (int k = 0; k <= 9; k++) {
        if (abs(j - k) >= 2)
          f[i][j] += f[i - 1][k];
      }
    }
  }
  ll x, y;
  scanf("%lld %lld", &x, &y);
  ll ans1 = calc(y);
  ll ans2 = calc(x - 1);
  printf("%lld", ans1 - ans2);
  return 0;
}
```