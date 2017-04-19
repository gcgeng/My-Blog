---
title: '[bzoj2301][HAOI2011]Problem B —— 莫比乌斯反演+容斥原理'
tags: [数论, 默比乌斯反演, 容斥原理]
date: 2017-02-03 18:15:00
---

#题意
给定a, b, c, d, k，求出：
$$\sum_{i=a}^b\sum_{j=c}^d[gcd(i, j) = k]$$

#题解
为方便表述，我们设
$$calc(\alpha, \beta) = \sum_{i=1}^{\alpha}\sum_{j=1}^{\beta}[gcd(i, j) = k]$$
令$A = \{ (x, y) | x < a\}$, $B = \{(x, y)|y < c\}$，

根据容斥原理，
$$|S| = |U| - |A| - |B| + |A \cap B|$$
所以，原式就是：
$$calc(b, d) - calc(a-1 ,d) - calc(b, c-1) + calc(a-1, c-1)$$
这样我们就把一个询问拆分成了四个询问，即，问题就转换成了计算$calc(\alpha, \beta)$

令
$$f(x) = \sum_{i=1}^b\sum_{j=1}^d[gcd(i, j) = x] $$
显然，f(x)并不方便计算，但是如果我们设
$$F(x) = \sum_{i=1}^b\sum_{j=1}^d[gcd(i, j) = \lambda， x|\lambda]$$
我们可以得出F(x)与f(x)的关系，
$$F(x) = \sum_{x|d} f(d)$$
F(x)就相对好计算的多，我们很容易有：
$$F(x) =\lfloor \frac{b}{i}\rfloor \lfloor\frac{d}{i} \rfloor$$
但是这一点对于我这种蒟蒻来说并不显然，所以这里给出一个证明。

>同样地，令$\lambda = gcd(i, j)$，如果$x|\lambda$，那么我们可以得出：
>1.$x|i$
>2.$x|j$
>反过来证明必要性：
>如果$x|i \&\& x|j$，那么x一定是i和j的公约数，所以一定有
>$x \leq \lambda$
>又因为x和$\lambda$都是公约数，所以$x|\lambda$，所以必要性得证。
>所以x是i和j的公约数是数对(i, j)可以对F(x)的充分必要条件。
>我们使用分步原理，首先在[1,n]中寻找x的倍数个数，然后在[1,m]里找，乘起来就可以了。

然后，根据mobius反演*（《具体数学》P113 4.9 $\phi$函数与$\mu$函数）*：
$$f(x) =\sum_{d|x} \mu(d) F(\frac{x}{d})$$
但是这种反演形式并不适合解此题，我们采取另外一种形式：
$$f(x) = \sum_{x|d} \mu(\frac{d}{x}) F(d) = \sum_{x|d}\mu(\frac{d}{x}) \lfloor \frac{n}{d} \rfloor \lfloor \frac{m}{d} \rfloor$$
由于枚举倍数显然只需要枚举到min(n, m)，所以复杂度为$\Theta(n+m)$
根据[神犇的课件](http://wenku.baidu.com/link?url=oyx4eqDOR5tbnCgIm8EAiXk4mAY4XEe2ZswI78dr66UsrQunJOakvOde7WigrgygsCvBtIEw0Vd0zSXbkJp6VPa96IZtGCUE2Axa3LxDnre)。
观察式子，我们发现：
$\lfloor \frac{n}{d} \rfloor$的取值最多有$2 \sqrt n$种（约数的个数），所以如果我们枚举$\lfloor \frac{n/m}{d} \rfloor$的取值，只需要枚举$2(\sqrt n + \sqrt m)$即可，复杂度就成了$\Theta (\sqrt n + \sqrt m)$
对于同一个取值，$\mu$函数是不同的，但是属于一个区间，我们可以统一求和，维护一个前缀和即可。

#代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 50005;
int T, a, b, c, d, k;
int mu[maxn+5], sumu[maxn+5], prime[maxn+5], check[maxn+5];
int tot = 0;
void get_mu() {
    memset(check, 0, sizeof(check));
    mu[1] = 1;
    for(int i = 2; i <= maxn; i++) {
        if(!check[i]) {
            prime[tot++] = i;
            mu[i] = -1;
        }
        for(int j = 0; j < tot; j++) {
            if(i * prime[j] > maxn) break;
            check[i * prime[j]] = 1;
            if(i % prime[j] == 0) {
                mu[i * prime[j]] = 0;
                break;
            } else {
                mu[i * prime[j]] = -mu[i];
            }
        }
    }
}

void init() {
    get_mu();
    for(int i = 1; i <= maxn; i++) sumu[i] = sumu[i-1] + mu[i];
}    

int calc(int n, int m) {
    n/=k;
    m/=k;
    int ret = 0;
    int last;
    if(n > m) swap(n, m);
    for(int i = 1; i <= n; i = last + 1) {
        last = min(n / (n/i), m / (m/i));
        ret += (n / i) * (m / i) * (sumu[last] - sumu[i-1]);
    }
    return ret;
}
int main() {
    init();
    scanf("%d", &T);
    while(T--) {
        scanf("%d %d %d %d %d", &a, &b, &c, &d, &k);
        int ans = calc(b, d) - calc(a-1, d) - calc(b, c-1) + calc(a-1, c-1);
        printf("%d\n", ans);
    }
    return 0;
} 
```
觉得自己好蠢。。。