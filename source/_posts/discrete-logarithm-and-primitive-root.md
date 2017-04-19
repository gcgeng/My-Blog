---
title: 离散对数与原根学习笔记
date: 2017-03-28 16:18:57
tags: [学习笔记, 数论]
---

# 定义

## 缩系

* 模$m$意义下与$m$互质的元素组成缩系.
* 缩系中任意两个元素的乘积还在缩系中.
* 缩系的大小是$\varphi(m)$.



## 阶

* 满足$x^r \equiv 1\pmod m$的最小整数$r$被称为$x$的阶, 记作$ord_m(x)$.
* 如果$x$和$m$互质且$m > 0$, 那么正整数$r$是$x^r\equiv 1$的一个解当且仅当$ord_m x|r$.
* 如果$x$和$m$互质且$m>0$, 那么$ord_m x|\varphi(m)$.
* 计算阶的$O(log^2n)$算法:
  * 首先分解$\varphi(n)$, 令$\varphi(n) = \prod p_i^{k_i}$.
  * 依次考察每一个质因子, 尝试一个一个地从答案中除去, 直到$x^{ans} \not \equiv 1$, 然后把之前除去的质因子加到答案中.
  * 因为要考虑$O(log n)$个因子, 每个因子使用$O(logn)$的时间快速幂验证, 也要考虑大数分解的复杂度, 总的复杂度就是$O(log^2 n + \varphi (n) ^ {\frac 14})$.
* 如果$x$和$m$互质且$m>0$, 那么:$x^i \equiv x^j \pmod m \Leftrightarrow i\equiv j  \pmod {ord_m x}$.



## 原根

* 如果$r$和$n$是互质且$n>0$, 那么当$ord_n a = \varphi(n)$时, 称$r$是模$n$的原根.

* 一个整数当它为2, 4, $p^t$, $2p^t$的时候才有原根, 这里$p$为奇素数且$t$为正整数.

* 令$ord_n(a)=t$, 那么$ord_n(a^k) = \frac t{(t, k)}$.

* 令$r$为模$m$的原根, 哪么$r^k$是模$m$的原根当且仅当$(k, \varphi(m))=1$.

  证明: $ord_m r^k = \frac{ord_m r}{(ord_m r, k)} =\frac{\varphi(m)}{(\varphi(m), k)}$

  由上面的推论可以证明: 如果正整数$n$有一个原根, 那么它一共有$\varphi(\varphi(n))$个不同的原根.

* 计算原根的算法:

  * 由上面计算阶的算法, 我们可以推得计算模$m$的一个原根的算法.
  * 首先, $ind_m g = \varphi(m) \pmod m$
  * 那么我们从小到大枚举所有与$m$互质的数, 那么我们令$\varphi(m) = \prod_{\omega(i)} p_i^{k_i}$, 根据原根的定义, 不难知道, 对于$\varphi(m)$的任何一个约数$q$, $g^q \not \equiv 1$, 又因为阶的性质, 我们可以得到一个更紧的条件: 对于任何一个$\frac{\varphi(m)}{p_i} = q$, $g^q \not \equiv 1$.
  * 复杂度为$O(nlog^2n)$. 虽然上界非常可怕, 但是实际上这个上界是非常非常松的.

计算原根的模板(51nod 1135):

```cpp
#include <cstdio>
#define ll long long
int n;
ll pow(ll a, ll b, ll n) {
  ll ans = 1;
  for (; b; b >>= 1, a = (a * a) % n)
    if (b & 1) ans = (ans * a) % n;
  return ans;
}
bool check(int i) {
  for (int j = 2; j * j < n; j++)
    if ((n - 1) % j == 0)
      if (pow(i, (n - 1) / j, n) == 1 || pow(i, j, n) == 1) return false;
  return true;
}
int proot() {
  for (int i = 2;; i++)
    if (pow(i, n - 1, n) == 1 && check(i)) return i;
}
int main() {
  scanf("%d", &n);
  printf("%d", proot());
}

```



## 离散对数

* 若有原根$g$, 则元素$x \equiv g^i \pmod m$关于$g$的离散对数(指标为)$ind_g(x) = i \pmod{\varphi(m)}$.
* 离散对数满足很多对数的性质.
* 指标: 以原根为底的离散对数, 记作$ind_{m,g}(x)$, 当模数固定时, 省略下标$m$.
* 根据阶的结论, 有:$ord_m x = \frac {\varphi(m)}{(\varphi(m), ind_g(x))}$.
* 若缩系没有原根, 那么模$m$缩系可以表示成一系列有原根的缩系的笛卡尔积.



# 概述

* 考虑解方程$A^B\equiv C \pmod M$:
  * 已知$ABM$求$C$是模幂问题.
  * 已知$ACM$求$B$是**离散对数**问题.
  * 已知$BCM$求$A$是**高次剩余**问题.
  * 已知$$ABC$$求$M$是大数分解问题.



# 模幂问题

快速幂.



# 离散对数问题

* 解方程$A^B \equiv C \pmod M$, 已知$ACM$求$B$.
* 不妨设一个原根是$g$, 那么问题等价于$ind_g(A)\times B \equiv ind_g(C) \pmod {\varphi(m)}$.

设$r=(ind_g(A), \varphi(m))=\frac{\varphi(M)}{ord_M(A)}$, 问题转化为$\frac{ind_g(A)}r \times B \equiv \frac{ind_g(C)}r$, 得到$B \equiv \frac{ind_g(C)}r (\frac{ind_g(A)}r)^{-1} \pmod{\frac{\varphi(M)}{r}}$.

* 大步小步算法: 可以看我的[这篇文章](http://mrmorning.coding.me/2017/01/25/%E6%A8%A1%E6%9D%BF-%E5%A4%A7%E6%AD%A5%E5%B0%8F%E6%AD%A5%E7%AE%97%E6%B3%95%E2%80%94%E2%80%94BSGS%E7%AE%97%E6%B3%95/)



## Pollard's rho Algorithm for Logarithms

* 把集合$G=\{A^i\ mod\ M|i \in \mathbb{N}\}$分为三个部分$S_0, S_1, S_2$, 并保证$1 \not \in S_1$.
* 生成一系列$x=A^iC^j$直到某个$x$找到了另一种表示方法$x=A^uC^v$, 则$(i-u)\equiv B(j-v) \pmod {|G|}$.
* 使用Floyd's Cycle-Finding Algorithm, 找环.
* 期望复杂度$O(\sqrt{\frac {\pi n}2})$.



## 扩展大步小步算法(Extended Baby-step Giant-step Algorithm)

* 枚举$\delta = 0, 1, \cdots$, 把方程化为$A^{B-\delta}A^{\delta} \equiv C \pmod M$, 直到$(A^{\delta}, M')=1$ , 消去公因子, 变成$A^{B-\delta}A' \equiv C' \pmod {M'}$.
* 如果在上面的过程中$C'=A'$, 那么就找到了一个解.
* 执行完上面的过程之后, 就变成了要解方程$\frac{A}{A^{\delta}}A^{B-\delta}\equiv \frac{C}{A^{\delta}} \pmod {\frac {M}{A^{\delta}}}$, 令$M' = \frac{M}{A^{\delta}}, C'=\frac{C}{A^{\delta}}(\frac{A}{A^{\delta}})^{-1}$, 那么方程就是$A^{B'} \equiv C' \pmod {M'}$, 执行类似BSGS的过程就好了.
* 复杂度$O(log_2 m +\sqrt m)$.

```c++
//spoj-Power Modulo Inverted
#include <cmath>
#include <cstdio>
#include <map>
#define ll long long
int a, c, m;
ll gcd(ll a, ll b) { return b ? gcd(b, a % b) : a; }
ll qpow(ll a, ll b, ll p) {
  ll ans = 1;
  for (; b; b >>= 1, a = (a * a) % p)
    if (b & 1) ans = (ans * a) % p;
  return ans;
}
int exbsgs(int a, int c, int m) {
  a %= m, c %= m;
  if (c == 1) return 0;
  int delta = 0;
  ll t = 1;
  for (int g = gcd(a, m); g != 1; g = gcd(a, m)) {
    if (c % g) return -1;
    m /= g, c /= g, t = t * a / g % m;
    ++delta;
    if (c == t) return delta;
  }
  std::map<int, int> hash;
  int M = int(sqrt(1.0 * m) + 1);
  ll tmp = c;
  for (int i = 0; i != M; i++) {
    hash[tmp] = i;
    tmp = tmp * a % m;
  }
  tmp = qpow(a, M, m);
  ll now = t;
  for (int i = 1; i <= m + 1; i++) {
    now = now * tmp % m;
    if (hash.count(now)) return i * M - hash[now] + delta;
  }
  return -1;
}
int main() {
  while (scanf("%d %d %d", &a, &m, &c), m) {
    if (m == 1) {
      printf("%s\n", c == 0 ? "0" : "No Solution");
      continue;
    }
    int ans = exbsgs(a, c, m);
    if (ans != -1)
      printf("%d\n", ans);
    else
      printf("No Solution\n");
  }
  return 0;
}

```



# 高次剩余问题

解方程$A^B \equiv C \pmod M$, 已知$BCM$求$A$.

如果$M$是质数的幂, 那么拆成若干个质数, 使用原根求解, 并使用中国剩余定理合并即可.

更难的部分先留坑Orz.



# 大数分解

Pollard $\rho$算法.

留坑.