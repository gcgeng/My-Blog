---
title: '[bzoj2111][ZJOI2010]Perm 排列计数 ——问题转换，建立数学模型'
tags: [模型转化, 动态规划, 树形DP]
date: 2017-01-25 19:18:00
---

#题目大意
>称一个1,2,...,N的排列P1,P2...,Pn是Magic的，当且仅当2<=i<=N时，Pi>Pi/2\. 计算1，2，...N的排列中有多少是Magic的，答案可能很大，只能输出模P以后的值。
#题解
1) 问题转换，建立模型。
可以发现，本题就是要求小根完全二叉树的个数。
2) 树上dp
定义f[n]为以n为根的完全二叉树个数。
根据乘法原理，
f[n] = f[i<<1] * f[i<<1|1] * C(s[i]-1, i << 1)
可以知道，n可以从后向前递推。

#代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 5e6+5;
#define ll long long
int n, p;
int f[maxn], s[maxn];
int fact[maxn], ifact[maxn];
int pow(int a, int b, int p) {
    int ans = 1;
    while(b) {
        if(b & 1) ans = (ll) ans * a % p;
        b >>= 1;
        a = (ll)a * a %p;
    }
    return ans;
}
int inv(int n, int p) {
    return pow(n, p-2, p);
}
void init() {
    fact[1] = 1;
    ifact[1] = 1;
    for(int i = 2; i <= n; i++) {
        fact[i] = (ll)i * fact[i-1] % p;
        ifact[i] = inv(fact[i], p);
    }
}
int C(int n, int m, int p) {
    if(n < m) return 0;
    return (ll)fact[n] * ifact[m] % p * ifact[n-m] % p;
}
int lucas(int n, int m, int p) {
    if(!n && !m) return 1;
    return (ll)C(n%p, m%p, p) * lucas(n/p, m/p, p) % p;
}
int main() {
    ifact[0] = 1;
    scanf("%d %d", &n, &p);
    init();
    for(int i = n; i; i--) {
        s[i] = s[i<<1] + s[i << 1|1] + 1;
        f[i] = lucas(s[i]-1, s[i<<1], p);
        if(i << 1 <= n) f[i] = (ll)f[i] * f[i<<1] % p;
        if((i << 1 | 1) <= n) f[i] = (ll)f[i] * f[i<<1|1] % p;
    }
    printf("%d", f[1]);
}
```