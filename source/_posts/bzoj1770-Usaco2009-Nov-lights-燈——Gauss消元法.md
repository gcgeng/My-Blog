---
title: '[bzoj1770][Usaco2009 Nov]lights 燈——Gauss消元法'
tags: [高斯消元]
date: 2017-03-04 15:28:00
---

# 题意
给定一个无向图，初始状态所有点均为黑，如果更改一个点，那么它和与它相邻的点全部会被更改。一个点被更改当它的颜色与之前相反。
# 题解
第一道Gauss消元题。所谓gauss消元，就是使用初等行列式变换把原矩阵转化为上三角矩阵然后回套求解。
给定一个矩阵以后，我们考察每一个变量，找到它的系数最大的一行，然后根据这一行去消除其他的行。具体地代码如下面所示。
#Code
```c++
double a[N][N]
void Gauss(){
    for(int i=1;i<=n;i++){
        int r=i;
        for(int j=i+1;j<=n;j++)
            if(abs(a[j][i])>abs(a[r][i])) r=j;
        if(r!=i) for(int j=1;j<=n+1;j++) swap(a[i][j],a[r][j]);

        for(int j=i+1;j<=n;j++){
            double t=a[j][i]/a[i][i];
            for(int k=i;k<=n+1;k++) a[j][k]-=a[i][k]*t;
        }
    }
    for(int i=n;i>=1;i--){
        for(int j=n;j>i;j--) a[i][n+1]-=a[j][n+1]*a[i][j];
        a[i][n+1]/=a[i][i];
    }
}
```
对于xor运算，我们可以使用同样的方法消元。
另外，xor的话可以使用bitset压位以加速求解。
#代码（附有详细注释）
```c++
#include <algorithm>
#include <cstdio>
const int maxn = 45;
int a[maxn][maxn], b[maxn];
int n, m, tot, mn = 0x3f3f3f;
void gauss() {
  for (int i = 1; i <= n; i++) { //依次考察每一个未知数
    int j = i;                   //开始选中第i行
    while (j <= n && !a[j][i]) //选中系数最大的一行（减小精度误差）
      j++;
    if (j > n)
      continue;
    if (i != j)
      for (int k = 1; k <= n + 1; k++) //交换两行，使得第i行成为最大系数
        std::swap(a[j][k], a[i][k]);
    for (int j = 1; j <= n;
         j++) // gauss消元核心代码：使用第i行消除所有行的第i个未知数
      if (i != j && a[j][i]) //以此来形成一个上三角矩阵，为之后的消元作准备
        for (int k = 1; k <= n + 1; k++)
          a[j][k] ^=
              a[i][k]; //如果是普通的线性方程组，这里需要使用别的方法把系数置零
  }
}
void dfs(int now) { //由于gauss消元后有一些自由元，我们需要进行最优解暴力搜索
  if (tot >= mn)
    return;
  if (!now) {
    mn = std::min(mn, tot);
    return;
  }
  if (a[now][now]) { //确定的情况
    int t = a[now][n + 1];
    for (int i = now + 1; i <= n; i++)
      if (a[now][i])
        t ^= b[i]; //由于是上三角矩阵，所以逆向消元
    b[now] = t;
    if (t)
      tot++;
    dfs(now - 1);
    if (t)
      tot--; //回溯
  } else {   //自由元的情况，随意确定
    b[now] = 0;
    dfs(now - 1);
    b[now] = 1;
    tot++;
    dfs(now - 1);
    tot--;
  }
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d %d", &n, &m);
  for (int i = 1; i <= n; i++)
    a[i][i] = 1, a[i][n + 1] = 1;
  for (int i = 1; i <= m; i++) {
    int x, y;
    scanf("%d %d", &x, &y);
    a[x][y] = a[y][x] = 1;
  }
  gauss(); //判定是否无解：系数矩阵全0,常数矩阵不全为0
  dfs(n);
  printf("%d\n", mn);
  return 0;
}
```
#附：如何使用bitset
首先，声明bitset:
```
#include <bitset>
using std::bitset;
```
初始化：
```
bitset<n> b;
bitset<n> b(unsigned long u);
```
上述语句声明了一个n位全部为0的bitset，第二个语句用一个unsigned long long变量去初始化bitset。
bitset的更多操作：
![](http://images2015.cnblogs.com/blog/890886/201703/890886-20170304154243048-670536588.png)
```c++
b1 = b2 & b3;//按位与
b1 = b2 | b3;//按位或
b1 = b2 ^ b3;//按位异或
b1 = ~b2;//按位补
b1 = b2 << 3;//移位
```