---
title: '如何使用矩阵乘法加速动态规划——以[SDOI2009]HH去散步为例'
tags: []
date: 2017-01-21 16:55:00
---

#题目描述
摘自[BZOJ 1875](http://www.lydsy.com/JudgeOnline/problem.php?id=1875)
##Description
HH有个一成不变的习惯，喜欢饭后百步走。所谓百步走，就是散步，就是在一定的时间 内，走过一定的距离。 但是同时HH又是个喜欢变化的人，所以他不会立刻沿着刚刚走来的路走回。 又因为HH是个喜欢变化的人，所以他每天走过的路径都不完全一样，他想知道他究竟有多 少种散步的方法。 现在给你学校的地图（假设每条路的长度都是一样的都是1），问长度为t，从给定地 点A走到给定地点B共有多少条符合条件的路径
##Input
第一行：五个整数N，M，t，A，B。其中N表示学校里的路口的个数，M表示学校里的 路的条数，t表示HH想要散步的距离，A表示散步的出发点，而B则表示散步的终点。 接下来M行，每行一组Ai，Bi，表示从路口Ai到路口Bi有一条路。数据保证Ai = Bi，但 不保证任意两个路口之间至多只有一条路相连接。 路口编号从0到N − 1。 同一行内所有数据均由一个空格隔开，行首行尾没有多余空格。没有多余空行。 答案模45989。
##Output
一行，表示答案。
##Sample Input
4 5 3 0 0
0 1
0 2
0 3
2 1
3 2
##Sample Output
4
##HINT
对于30%的数据，N ≤ 4，M ≤ 10，t ≤ 10。 对于100%的数据，N ≤ 20，M ≤ 60，t ≤ 2^30，0 ≤ A,B
#对这个题目的最初理解
开始看到这个题，觉得很水，直接写了一个最简单地动态规划，就是定义
f[i][j]为到了i节点路径长度为j的路径总数，
转移的话使用Floyd算法的思想去转移，借助这个题目也理解了为什么floyd要把k放在最外面，也是类似的道理。
这样就写了下面代码中的version1。但是连样例也无法通过。
我又重新仔仔细细读了一遍题，发现不可以走回头路。
然后我就一直在考虑如何避免走回头路，但是想了一个小时，也想不出一个合理的猜想，每一个猜想有非常大的局限性。
**然后就上网翻题解，发现可以使用边来定义状态，进行转移。**
这样就写了第二个version，得到了30分，tle。
又仔细看了一下题目，发现：t<=$2^{30}$！如此庞大的数据，我的$O(n^3)$的超级朴素算法根本无法通过。
于是上网看题解，发现了矩阵优化这一玄学的技巧。
#什么是矩阵乘法
矩阵优化dp这个问题很早之前就听说过，最开始是在快速求解fibonacci数列时知道的。
我去[学堂在线](http://www.xuetangx.com)上上了几节线性代数课程，大概了解了矩阵乘法。
定义矩阵A(n*m), 矩阵B(m*k)，定义矩阵C是一个(n*k)的矩阵，为A*B
对于矩阵C中的每一个元素，为$\sum (a[i][k]*b[k][j])$；
这就是矩阵乘法。
矩阵乘法有什么应用呢？一大应用就是去*表示*线性方程组。
比如有一个方程组:
$$
\begin{equation}
\left\{
\begin{array}{lll}
a_1x_1+a_2x_2+...+a_nx_n = z_1\\
b_1x_1+b_2x_2+...+b_nx_n = z_2 \\
...
\end{array}
\right.
\end{equation}
$$
我们写出他的系数矩阵A：
$$
\begin{equation}
A=
\left[
\begin{array}{cccc}
a_1 & a_2 & ... & a_n \\
b_1 & b_2 & ... & b_n \\
....&...&...&...
\end{array}
\right]
\end{equation}
$$
写出增广系数矩阵B
$$
\begin{equation}
B=
\left[
\begin{array}{c}
z_1&
z_2&
...&
z_n
\end{array}
\right]
\end{equation}
$$
写出未知数矩阵X
$$
\begin{equation}
X=
\left[
\begin{array}{c}
x_1&
x_2&
...&
x_n
\end{array}
\right]
\end{equation}
$$
根据矩阵乘法的定义，我们可以用AX=B来表示这个线性方程组。
***所以矩阵乘法可以被认为是把一个列向量向另外一个列向量的映射变化***。

#矩阵乘法的应用---几个dp问题

所以在动态规划中，当我们需要求解一个阶段划分明显，一个阶段可以由一个函数整体映射到下一个阶段时，我们就可以使用矩阵乘法进行优化。
我们看两个经典的dp问题，观察他们是否符合我们说明的这个规律。
首先是经典的0-1背包问题，我们写出方程:
f[i][j] = max(f[i-1][j-w[i]]+v[i], f[i-1][j]);
很显然，在前一个状态和后一个状态之间，并不是一个*线性关系*。
再看LIS问题：
f[i] = max(f[j]+1);
这就更不是一种线性关系了。

#使用矩阵乘法加速fibonacci数列的求解

我们再看经典的fibonacci数列：

$$
\begin{equation}
\left\{
\begin{array}{c}
f[n] = f[n-1] + f[n-2] \\
f[n-1] = f[n-1]
\end{array}
\right.
\end{equation}
$$
很显然，这就是一个线性关系；
我们可以写出这个线性方程组对应的映射：
$$
\begin{equation}
\left[
\begin{array}{c}
1&1\\
1&0
\end{array}
\right]
\end{equation}
$$

然后可以有：
$$
\begin{equation}
A=
\left[
\begin{array}{c}
f[n-1]&f[n-2]
\end{array}
\right]
\end{equation}
$$

$$
\begin{equation}
B=
\left[
\begin{array}{c}
1&1\\1&0
\end{array}
\right]
\end{equation}
$$

$$
\begin{equation}
C=
\left[
\begin{array}{c}
f[n]&f[n-1]
\end{array}
\right]
\end{equation}
$$
有$A*B = C$
所以我们只需要计算矩阵A*B就可以计算出C
现在我们知道
$$
\begin{equation}
\left[
\begin{array}{c}
f[1]&f[2]
\end{array}
\right]
\end{equation}
$$
等于
$$
\begin{equation}
\left[
\begin{array}{c}
1&1
\end{array}
\right]
\end{equation}
$$
所以$C = A * B ^ {n-1}$
答案就是C[1][1]。
上面便是使用矩阵乘法解决fibonacci数列求解的一个例子。

#回到这个问题

现在，我们考虑这个题。
方程容易写出:
$$f[i][k] = \sum_{\substack{to[j] == from[i] \\ i != j\wedge1} }f[j][k-1]$$

这里，我们使用邻接表存储，边从2开始编号，这样一个边i的反向边就是i^1
可以看出，前一个阶段到新一个阶段的映射是线性的，并且这个映射是不变的。
我们可以预先求解这个映射。
根据矩阵乘法的定义，这个映射可以被表示成一个0-1矩阵(n*n)
对于位置i,j如果满足to[i] == from[j] && i != j^1这个条件，那么这个位置是1，反之就是0；
这样我们只需要求解出映射的（K-1）幂就可以了。
至于初始矩阵，如果某个点可以一步到达，那么这个点为1，否则为0.
这样，我们就圆满的解决了这个问题。

#代码
下面上代码。
```
/*#include <bits/stdc++.h> //////version 1 : 10 WA
using namespace std;
const int maxn = 11;
int g[maxn][maxn];
int n, m, k, s, t;
int f[maxn][maxn];
int main() {
    scanf("%d %d %d %d %d", &n, &m, &k, &s, &t);
    memset(g, 0, sizeof(g));
    for(int i = 1; i <= m; i++) {
        int x, y;
        scanf("%d %d", &x, &y);
        g[x][y] = g[y][x] = 1;
    }
    memset(f, 0, sizeof(f));
    f[s][0] = 1;
    for(int kk = 0; kk <= k; kk++) {
       for(int i = 0; i < n; i++) {
          for(int j = 0; j < n; j++) {
             if(g[i][j]) {
                bool ok = (g[i][j]) && (f[j][kk-1]) && f[i][kk];
                if(i!= j)if(!ok)f[j][kk+1] += f[i][kk];
                else f[j][kk+1] = f[j][kk+1]+f[i][kk]-f[j][kk-1];

             }
          }
       }
    }
    for(int i = 0; i < n; i++) {
        cout << i << endl;
        for(int kk = 0; kk <= k; kk++) {
            cout << f[i][kk] << ' ';
        }
        cout <<  endl;
    }
    printf("%d", f[t][k]);
   return 0;
}*/
/*   ////////////////////////////version 2 : 30  TLE/RE
#include <bits/stdc++.h>
using namespace std;
const int maxn = 11;
int cnt = 0;
int to[maxn], last[maxn], nex[maxn], from[maxn];
int n, m, K, s, t;
int f[maxn][maxn];
vector<int> ans;
void insert(int fr, int tt) {
   cnt++;
   nex[last[fr]] = cnt;
    last[fr] = cnt;
    to[cnt] = tt;
    from[cnt] = fr;
    if(fr == s) {
        f[cnt][1] = 1;
    }
    if(tt == t) {
       ans.push_back(cnt);
    } 
}
bool ok(int a, int b) { //judge if it's ok to transfer f[a][k] to f[b][k+1];
    return to[a] == from[b] && !(from[a] == to[b]); 
}
int main() {
  scanf("%d %d %d %d %d", &n, &m, &K, &s, &t);
  memset(f, 0, sizeof(f));
  memset(last, 0, sizeof(last));
  for(int i = 0; i < m; i++) {
     int a, b;
     scanf("%d %d", &a, &b);
     insert(a, b);
     insert(b, a);
  }   
  for(int k = 1; k <= K; k++) {
      for(int i = 1; i <= cnt; i++) {
          for(int j = 1; j <= cnt; j++) {
              if(ok(i, j)) {
                  f[j][k+1] += f[i][k];
              }
          }
      }
  }
  vector<int>::iterator it;
  int an = 0;
  for(it = ans.begin(); it != ans.end(); it++) {
    an = (an+f[*it][K]) % 45989;
  }
  printf("%d", an);
    return 0;
}*/
#include <bits/stdc++.h>
using namespace std;
const int mod = 45989;
const int maxn = 21;
int cnt = 1;
int last[maxn], nex[1005], to[1005], from[1005];
int n, m, K, s, t;
void insert(int a, int b) {
    cnt++;
    nex[cnt] = last[a];
    last[a] = cnt;
    to[cnt] = b;
    from[cnt] = a;
}
struct Matrix {
    int v[125][125];
    Matrix(){
        memset(v, 0, sizeof(v));
    }
    friend void print(Matrix a) {
        for(int i = 1; i <= cnt; i++) {
            for(int j = 1; j <= cnt; j++) {
                cout << a.v[i][j] << ' ';
            }
            cout << endl;
        }
    }
    friend Matrix operator * (Matrix a, Matrix b) {
        Matrix ans;
        for(int i = 1; i <= cnt; i++) {
            for(int j = 1; j <= cnt; j++) {
                for(int k = 1; k <= cnt; k++) {
                    ans.v[i][j] = (ans.v[i][j] + a.v[i][k] * b.v[k][j]) % mod;
                }
            }
        }
        return ans;
    }
    friend Matrix operator ^ (Matrix a, int b) {
        Matrix ans;
        for(int i = 1; i <= cnt; i++) ans.v[i][i] = 1;
        for(int i = b; i;i >>= 1, a = a * a) {
           if(i & 1) ans = ans * a;
        }
        return ans;
    }
}a, b;
int main() {
    scanf("%d %d %d %d %d", &n, &m, &K, &s, &t);
    memset(last, 0, sizeof(last));
    for(int i = 1; i <= m; i++) {
        int x, y;
        scanf("%d %d", &x, &y);
        insert(x, y);
        insert(y, x);
    }
    for(int i = last[s]; i; i = nex[i]) {
        a.v[1][i] = 1;
    }
    for(int i = 2; i <= cnt; i++) {
        for(int j = 2; j <= cnt; j++) {
            if(to[i] == from[j]) {
                if(i != (j ^ 1)) {
                    b.v[i][j] = 1;
                }
            }
        }
    }
    a = a * (b ^ (K-1));
    /*for(int i = 1; i <= 3; i ++) {
        for(int j = 2; j <= cnt; j++) {
            cout << a.v[1][j] << ' ';
        }
        cout << endl;
        a = a * b;
    }*/
    int ans = 0;
    for(int i = last[t]; i; i = nex[i]) {
       ans = ans + a.v[1][i^1];
    }
    printf("%d\n", ans % mod);
    return 0;
} 
```
#注
1.代码实现时出了一些问题：边存储时由于拆成了两条边，所以65是不够的。