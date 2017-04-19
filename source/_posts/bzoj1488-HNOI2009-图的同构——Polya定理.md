---
title: '[bzoj1488][HNOI2009]图的同构——Polya定理'
tags: [组合数学, Polya定理]
date: 2017-02-11 22:24:00
---

#题目大意
>求两两互不同构的含n个点的简单图有多少种。
>简单图是关联一对顶点的无向边不多于一条的不含自环的图。
>a图与b图被认为是同构的是指a图的顶点经过一定的重新标号以后，a图的顶点集和边集能完全与b图一一对应。

#题解
这个题是学习了Polya定理和群论以后的练手题，但是推了好久并没有推出来。。。。真的是太难辣。。。
首先我先说一下我错误的想法：
很容易就把这个题转化成了给$K_n$的完全图上的边进行二着色的问题，然后，由于在组合数学课程中经常接触到多边形着色，所以我就把这个题错误的转化成了在一个正$\frac{n(n-1)}{2}$边形的顶点上进行二着色的问题。然而对于n=1,2,3这种方法都是可行的，但是到了n=4的情况，这种方法就不可行了。我仔细观察了一下，发现这个转化**不符合满足纯粹性和完备性**。。。
然后就说一下正解吧。
首先我们考虑n = 4的情况，对于$K_4$进行二着色，我们很容易发现，由于图是可以任意扭转的，所以它的置换群**实际上是一个对称群**！
那么对于点的每一个置换我们要计算对应的边的置换。
在一个置换中，考察一条边，如果这条边的两个节点位于相同的**循环**中，那么我们可以得出边的循环个数是点的循环个数的一半。
如果这条边的两个节点位于不同的循环中，那么我们画一画图就可以知道如果点的循环个数分别是a, b,那么边的循环个数就是gcd(a,b)。
根据这样的方法，我们就可以把点的置换转化为边的置换了。
下面的任务就是要枚举置换。
如果直接暴力，复杂度很高。
我们考虑这样的枚举（回溯）方法：
依次考虑每一种阶的循环的个数，然后暴力dfs即可。
现在假设我们已经枚举好了一个置换，那么这种置换的个数根据一些基本的排列组合知识，可以知道是：
$$\frac{n!}{\prod_{i = 1}^{cnt} Val_i * Num_i !}$$

>稍微解释一下这个式子。除以$Val_i$是因为圆形排列，除以$Num_i$是因为同阶循环的重复排列。

根据Polya定理，等价类的个数就是：
$$l = \frac{1}{N!} * \sum 2^m$$

[参考题解](http://blog.csdn.net/wzq_qwq/article/details/48035455)

>事实上，这个题还有一个变态的做法：
>就是上OEIS上查询通项公式。。。。
#代码
```
#include <bits/stdc++.h>
const int mod = 997;
const int maxn = 1010;
using namespace std;
int n, cnt, ans;
int two[maxn], factor[maxn], val[maxn], num[maxn];
int pow(int n, int m) {
    int ans = 1;
    int b = m;
    while(b) {
        if(b & 1) ans = (ans * n) % mod;
        b >>= 1;
        n = (n*n) % mod;
    }
    return ans;
}
int inv(int n) {
    return pow(n, mod-2);
}
int gcd(int a, int b) {
    if(b == 0) return a;
    else return gcd(b, a%b) % mod;
}
void init() {
    factor[0] = factor[1] = two[0] = 1;
    for(int i = 2; i <= 1000; i++) {
        factor[i] = ((i % mod) * factor[i-1]) % mod;
    }
    for(int i = 1; i <= 1000; i++) {
        two[i] = (two[i-1] * 2) % mod;
    }
}
void dfs(int now_num, int left) {
    if(left == 0) {
        int sum1 = 0, sum2 = 1;
        //sum1:这一种置换的循环个数
        //sum2:这一种置换的个数
        for(int i = 1; i <= cnt; i++) {
            sum1 += (num[i] * (num[i] - 1) / 2 * val[i]) + (val[i]/2 * num[i]); 
            //前一部分:对于同一种循环中的不同循环的边的处理
            for(int j = i + 1; j<= cnt; j++) {
                sum1 += num[i] * num[j] * gcd(val[i], val[j]);
            }
        }
        for(int i = 1; i <= cnt; i++) {
            sum2 = (sum2 * pow(val[i], num[i])%mod*factor[num[i]])%mod;
        }
        sum2 = inv(sum2) * factor[n] % mod;
        ans = (ans + pow(2, sum1) * sum2 % mod) % mod;
    }
    if(now_num > left) return;
    dfs(now_num+1, left);
    //这里的dfs放到外面可以降低常数：
    //如果放在for循环里面，那么num数组中会多出许多0
    //浪费时间。
    for(int i = 1; i * now_num <= left; i++) {
        val[++cnt] = now_num, num[cnt] = i;
        dfs(now_num+1, left - i * now_num);
        cnt--; //回溯
    }
}

int main() {
    scanf("%d", &n);
    init();
    dfs(1, n);
    ans = (ans * inv(factor[n])) % mod;
    printf("%d", ans);
    return 0;
}
```