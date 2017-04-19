---
title: '[bzoj4513][SDOI2016]储能表——数位dp'
tags: [动态规划, 数位DP]
date: 2017-02-18 21:00:00
---

# 题目大意
求
$$\sum_{i = 0}^{n-1}\sum_{j=0}^{m-1} max((i\ xor\ j)\ -\ k,\ 0)\ mod\ p$$

# 题解
首先，开始并没有看出来这是数位dp。
后来发现可以一位一位的处理，每一位可以从前一位推出，所以可以考虑数位dp。
我们把要统计的数变为二进制表示，先考虑n位二进制的数，再考虑n-1位的数……，加起来就好辣。
定义f[i][1/0][1/0][1/0]为已经考虑到了第i位，第i位是否比n（第i位）小，第i位是否比m小，是否比k小的总共分数。
定义g[i][1/0][1/0][1/0]为已经考虑到了第i位，第i位是否比n（第i位）小，第i位是否比m小，是否比k小的所有情况总数。
我们从大到小考虑每一位，可以有，
g[i][aa][bb][cc] += g[i+1][a][b][c];
如果从状态(i,a,b,c)可以转移到状态(i-1, aa, bb, cc);
对于
f[i][aa][bb][cc] += f[i+1][a][b][c] + (zz-z)\*(1<<i)\*g[i+1][a][b][c];
前一项不必多说，后一项就是对于每一种可能都可以在前面加上1/0。
详细可以见代码。
有问题可以在评论区吐槽。。。
# 代码

```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;
ll f[62][2][2][2], g[62][2][2][2]; //log2(10^18) = 60
ll n, m, k, T, p;
int main() {
  scanf("%lld", &T);
  while (T--) {
    memset(f, 0, sizeof(f));
    memset(g, 0, sizeof(g));
    scanf("%lld %lld %lld %lld", &n, &m, &k, &p);
    g[61][1][1][1] = 1;
    for (int i = 60; i >= 0; i--) {
      int x = (n >> i) & 1, y = (m >> i) & 1, z = (k >> i) & 1;
      for (int a = 0; a < 2; a++) {
        for (int b = 0; b < 2; b++) {
          for (int c = 0; c < 2; c++) {
            if (f[i + 1][a][b][c] || g[i + 1][a][b][c]) {
              for (int xx = 0; xx < 2; xx++) {
                for (int yy = 0; yy < 2; yy++) {
                  int zz = xx ^ yy;
                  if ((a && x < xx) || (b && y < yy) || (c && z > zz))
                    continue;
                  int aa = (a && x == xx), bb = (b & y == yy),
                      cc = (c && z == zz);
                  g[i][aa][bb][cc] = (g[i][aa][bb][cc] + g[i + 1][a][b][c]) % p;
                  f[i][aa][bb][cc] = (f[i][aa][bb][cc] + f[i + 1][a][b][c] +
                                      ((zz - z) + p) % p * ((1ll << i) % p) %
                                          p * g[i + 1][a][b][c] % p) %
                                     p;
                }
              }
            }
          }
        }
      }
    }
    printf("%lld\n", f[0][0][0][0]);
  }
}
```