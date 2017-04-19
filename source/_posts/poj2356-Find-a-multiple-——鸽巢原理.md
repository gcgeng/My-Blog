---
title: '[poj2356]--Find a multiple ——鸽巢原理'
tags: []
date: 2017-02-01 20:48:00
---

#题意：
给定n个数，从中选取m个数，使得$\sum | n$。本题使用Special Judge.

#题解：
既然使用special judge，我们可以直接构造答案。
首先构造在mod N剩余系下的前缀和。
$$sum_i = (a_i + sum_{i-1}) mod n$$
剩余系N的完系中显然共有N-1个元素，我们有N个前缀和。
根据***鸽巢原理***，一定有$sum_j = sum_i$
所以这样构造是可行的。

#TRICK
**具体实现的时候用了一个技巧：
从前往后扫描sum数组，记录一个pos数组，这样就可以把时间复杂度降到了$\Theta (n)$**

#代码
```
#include <cstdio>
#include <cstring>
const int maxn = 10005;
int N, a[maxn], sum[maxn], pos[maxn];
int main() {
    scanf("%d", &N);
    memset(pos, -1, sizeof(pos));
    for(int i = 1; i <= N; i++) { 
        scanf("%d", &a[i]);
        sum[i] = (a[i] + sum[i-1]) % N;
    }
    pos[0] = 0;
    for(int i = 1; i <= N; i++) {
        if(pos[sum[i]] == -1) {
            pos[sum[i]] = i;
        }
        else {
            printf("%d\n", i-pos[sum[i]]);
            for(int j = pos[sum[i]]+1; j <= i; j++) {
                printf("%d\n", a[j]);
            }
            return 0;
        }
    }
    return 0;
}
```