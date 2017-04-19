---
title: '[bzoj1833][ZJOI2010]count 数字计数——数位dp'
tags: [动态规划, 数位DP]
date: 2017-02-16 20:54:00
---

#题目：
(传送门)[http://www.lydsy.com/JudgeOnline/problem.php?id=1833]
#题解：
第一次接触数位dp，真的是恶心。
首先翻阅了很多很多一维dp，因为要处理前缀0，所以根本搞不懂。
查询了dalaolidaxin的博客，又查阅了资料：
[初探数位dp](http://wenku.baidu.com/link?url=9OS5Ybpw5wx00ahrH8ED2oyIlR1uWwrxT8N4pEg27GgBt2T2hLe4sd_h1rmpY7P0HmeHIEDw9h6_K98dPhhjoMhD2TpKcS8w1X8cC_dkPp_)
才完全弄懂这个题。
具体的，我们设
f[i][j][k]为考虑所有i位数，最高位为j数，之中k的数目。
我们可以得出方程：
$$f[i][j][k] = \sum f[i-1][l][k]    (j!=k)$$
$$f[i][j][k] = \sum f[i-1][l][k] + 10^{i-1}     (j==k)$$
我们对这个方程作出解释：
前一项非常好理解，后一项的话就是前(i-1)位数共有$10^{i-1}$个，对于其中每一个，我们都可以在前面加k。
这样我们预处理出来了f。
然后我们考虑对于n分块计算。
以n = 4321为例。
首先统计3位及以下的数，这些数字没有限制，直接加就好。
然后统计4位数。
对于一个4位数，我们一位一位向下考虑，如果最高位<k，直接加，如果=k，加上n+1
具体见代码。
#代码
```
#include <cstdio>
#include <cstring>
using namespace std;
#define ll long long
const int N = 25;
struct node {
  ll a[N];
  node() { memset(a, 0, sizeof(a)); }
  ll &operator[](const int &x) { return a[x]; }
};
node operator+(const node &x, const node &y) {
  node tmp;
  for (int i = 0; i <= 9; i++)
    tmp.a[i] = x.a[i] + y.a[i];
  return tmp;
}
int len, a[N];
ll pow[N];
node f[N][N];
void init(ll n) {
  len = 0;
  while (n) {
    a[++len] = n % 10;
    n /= 10;
  }
  for (int i = 0; i <= 9; i++)
    f[1][i][i] = 1;
  for (int i = 2; i <= 14; i++) {
    for (int j = 0; j <= 9; j++) {
      for (int k = 0; k <= 9; k++)
        f[i][j] = f[i][j] + f[i - 1][k];
      f[i][j][j] += pow[i - 1];
    }
  }
}
node calc(ll n) {
  node ans;
  if (!n)
    return ans;
  memset(f, 0, sizeof(f));
  init(n);
  //统计前len-1位
  for (int i = 1; i <= len - 1; i++) {
    for (int j = 1; j <= 9; j++) {
      ans = ans + f[i][j];
    }
  }
  //开始统计len位数
  for (int i = 1; i <= a[len] - 1; i++)
    ans = ans + f[len][i];
  n %= pow[len - 1];
  ans[a[len]] += n + 1; //对于每一个最高位都可以统计一发
  for (int i = len - 1; i; i--) {
    for (int j = 0; j < a[i]; j++)
      ans = ans + f[i][j];
    n %= pow[i - 1];
    ans[a[i]] += n + 1;
  }
  return ans;
}
int main() {
  pow[0] = 1;
  for (int i = 1; i <= 14; i++)
    pow[i] = pow[i - 1] * 10;
  ll x, y;
  scanf("%lld %lld", &x, &y);
  node ans1 = calc(y), ans2 = calc(x - 1);
  for (int i = 0; i <= 8; i++)
    printf("%lld ", ans1[i] - ans2[i]);
  printf("%lld\n", ans1[9] - ans2[9]);
  return 0;
}
```