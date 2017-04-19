---
title: '[hdu1695] GCD ——欧拉函数+容斥原理'
tags: []
date: 2017-02-01 23:40:00
---

#题目
给定两个区间[1, b], [1, d]，统计数对的个数(x, y)满足：
1\. $x \in [1, b]$, $y \in [1, d]$ ;
2\. $gcd(x, y) = k$
[HDU1695](http://acm.hdu.edu.cn/showproblem.php?pid=1695)

#题解
我们观察式子$gcd(x,y)=k$
很显然，$gcd(x/k, y/k) = 1$
我们令b < d，令x<y（避免重复计数）
分类讨论。
1) y < b
可以看出答案就是$\sum_{i \in [1, b]} \phi(i)$
2)$y \in [b, d]$
可以看出答案就是calc(b, i)，calc函数就是在区间[1,b]中与i互素的个数。

怎么计算calc函数呢？
首先我们计算出i的因数，运用容斥原理。
$$| \overline{A_1} \cap \overline {A_2} ... \cap \overline{A_n}| = | S | - |A_1| - |A_2| ... + |A_1 \cap A_2| ....$$
具体计算见代码。

#代码
```
#include <bits/stdc++.h>
#define ll long long
using namespace std;
const int maxn = 100000;
int prime[maxn+5], phi[maxn+5], check[maxn+5];
int T, a, b, c, d, k;
int cnt = 0;
void get_phi(int n) {
    memset(check, 0, sizeof(check));
    cnt = 0;
    phi[1] = 1; 
    for(int i = 2; i <= n; i++) {
        if(!check[i]) {
           phi[i] = i-1;
           prime[cnt++] = i;
        }
        for(int j = 0; j < cnt; j++) {
            if(i * prime[j] > n) break;
            check[i*prime[j]] = 1;
            if(i % prime[j] == 0) {
                phi[i * prime[j]] = phi[i] * prime[j];
                break;
            } else {
                phi[i * prime[j]] = phi[i] * (prime[j] - 1);
            }
        }
    }
}
ll factor[maxn];
int ct = 0;
void get_factor(int x) {
    ct = 0;
    ll tmp = x;
    for(int i = 0; prime[i] * prime[i] <= x; i++) {
        if(tmp % prime[i] == 0) {
            factor[ct] = prime[i];
            while(tmp % prime[i] == 0) {
                tmp /= prime[i];
            }
            ct++;
        }
    }
    if(tmp != 1) 
        factor[ct++] = tmp;

} 

int calc(int n, int m) {
    get_factor(m);
    int ans = 0;
    for(int i = 1; i < (1 << ct); i++) {
        int cnt = 0;
        int tmp = 1;
        for(int j = 0; j < ct; j++) {
            if(i & (1 << j)) {
                cnt++;
                tmp *= factor[j];
            }
        }
        if(cnt & 1) ans += n / tmp;
        else ans -= n/tmp;
    }
    return n - ans;
}

int main() {
    //freopen("input", "r", stdin);
    scanf("%d", &T);
    get_phi(maxn);
    int kase = 0;
    while(T--) {
        kase++;
        scanf("%d %d %d %d %d", &a, &b, &c, &d, &k);
        if((k == 0) | (k > b) | (k > d)) {
            printf("Case %d: 0\n", kase);
            continue;
        }
        if(b > d) swap(b, d);
        b /= k;
        d /= k;
        ll ans = 0;
        for(int i = 1; i <= b; i++) ans += phi[i];
        for(int i = b+1; i <= d; i++)
            ans += calc(b, i);
        printf("Case %d: %lld\n", kase, ans);
    }
    return 0;
}
```