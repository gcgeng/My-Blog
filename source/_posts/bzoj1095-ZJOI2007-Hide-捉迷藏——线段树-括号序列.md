---
title: '[bzoj1095][ZJOI2007]Hide 捉迷藏——线段树+括号序列'
tags: [线段树, 括号序列, 动态点分治]
date: 2017-03-04 11:42:00
---

#题目大意
给定一棵所有点初始值为黑的无权树，你需要支援两种操作：
1\. 把一个点的颜色反转
2\. 统计最远黑色点对。

#题解
本题是一个树上的结构。对于树上的结构，我们可以采用点分治、树链剖分等方法处理，这个题用了一个巧妙的方法，化树为线性数列，从而解决了问题。
定义一种对一棵树的括号编码。这种编码方式很直观，所以，这里不给出严格的定义，用以下这棵树为例：
![](http://images2015.cnblogs.com/blog/890886/201703/890886-20170304113106126-1381317123.png)
它的括号序列就是$(A(B)(C(D)(E)))$
括号序列有着非常好的性质。对于一个括号序列，两个点之间的距离就是他们中间的括号成对消除之后剩余括号的数量。证明非常容易，直观地来看地话就是在树中，从 E 向上爬 2 步就到了A。
对于一段括号编码，我们使用数对$(a,b)$来描述它，表示它在消除后有a个左括号，b个右括号。所有，我们只需要设计一种数据结构支持单点修改，区间查询就好辣。
这让我们联想到线段树。那么下一步我们就是考虑如何从两个字节点合并成一个父节点。这让我们想起最长连续和。
考察一个合法的序列，如果它有贡献，那么序列的左右两边一定都有一个黑点，那么，父节点的最长序列有这样几种情况。
1\. 子序列在左边
2\. 子序列在右边
3\. 子序列跨过中间。

对于前两种情况，我们递归处理，第三种情况的话，分析一下：
 也就是说，题目只需要动态维护：max{a+b | S’(a, b) 是 S 的一个子串，且 S’ 介于两个黑点之间}，
这里 S 是整棵树的括号编码。我们把这个量记为 dis(s)。

现在如果可以通过左边一半的统计信息和右边一半的统计信息，得到整段编码的统计，这道题就可以用熟悉的线段树解决了。

这需要下面的分析。
考虑对于两段括号编码 S1(a1, b1) 和 S2(a2, b2)，如果它们连接起来形成 S(a, b)。
注意到 S1、S2 相连时又形成了 min{b1, a2} 对成对的括号，合并后它们会被抵消掉。
所以：

当 a2 < b1 时第一段 [ 就被消完了，两段 ] 连在一起，例如：
] ] [ [  +  ] ] ] [ [  =  ] ] ] [ [
当 a2 >= b1 时第二段 ] 就被消完了，两段 [ 连在一起，例如：
] ] [ [ [  +  ] ] [ [  =  ] ] [ [ [ （？..反了？。。。

这样，就得到了一个十分有用的结论：

当 a2 < b1 时，(a,b) = (a1-b1+a2, b2)，
当 a2 >= b1 时，(a,b) = (a1, b1-a2+b2)。

由此，又得到几个简单的推论：

(i) a+b = a1+b2+|a2-b1| = max{(a1-b1)+(a2+b2), (a1+b1)+(b2-a2)}
(ii) a-b = a1-b1+a2-b2
(iii) b-a = b2-a2+b1-a1

由 (i) 式，可以发现，要维护 dis(s)，就必须对子串维护以下四个量：

right_plus：max{a+b | S’(a,b) 是 S 的一个后缀，且 S’ 紧接在一个黑点之后}
right_minus：max{a-b | S’(a,b) 是 S 的一个后缀，且 S’ 紧接在一个黑点之后}
left_plus：max{a+b | S’(a,b) 是 S 的一个前缀，且有一个黑点紧接在 S 之后}
left_minus：max{b-a | S’(a,b) 是 S 的一个前缀，且有一个黑点紧接在 S 之后}

这样，对于 S = S1 + S2，其中 S1(a, b)、S2(c, d)、S(e, f)，就有

(e, f) = b < c ? (a-b+c, d) : (a, b-c+d)
dis(S) = max{dis(S1), left_minus(S2)+right_plus(S1), left_plus(S2)+right_minus(S1), dis(S2)}

那么，增加这四个参数是否就够了呢？
是的，因为：

right_plus(S) = max{right_plus(S1)-c+d, right_minus(S1)+c+d, right_plus(S2)}
right_minus(S) = max{right_minus(S1)+c-d, right_minus(S2)}           
left_plus(S) = max{left_plus(S2)-b+a, left_minus(S2)+b+a, left_plus(S1)}   
left_minus(S) = max{left_minus(S2)+b-a, left_minus(S1)}               

这样一来，就可以用线段树处理编码串了。实际实现的时候，在编码串中加进结点标号会更方便，对于底层结点，如果对应字符是一个括号或者一个白点，那 么right_plus、right_minus、left_plus、left_minus、dis 的值就都是 -maxlongint；如果对应字符是一个黑点，那么 right_plus、right_minus、left_plus、left_minus 都是 0，dis 是 -maxlongint。

现在这个题得到圆满解决，回顾这个过程，可以发现用一个串表达整棵树的信息是关键，这一“压”使得线段树这一强大工具得以利用.. . 
以上内容总结来自[岛娘的博客](http://www.shuizilong.com/house/archives/bzoj-1095-zjoi2007hide-%E6%8D%89%E8%BF%B7%E8%97%8F/)
代码借鉴了[hzwer的博客](http://hzwer.com/5247.html)

#代码
```c++
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <vector>
#ifdef D
const int maxn = 100;
#else
const int maxn = 300500;
#endif
const int inf = 0x3f3f3f;
using std::max;
std::vector<int> G[maxn];
int sz = 0;
int seq[maxn << 1];
int pos[maxn];
int color[maxn];
int n, m;
char opt;
struct seg {
  int l, r, dis, right_plus, right_minus, left_plus, left_minus, c1, c2;
  void ini(int x) {
    dis = -inf;
    c1 = c2 = 0;
    if (seq[x] == 0)
      c2 = 1;
    if (seq[x] == -1)
      c1 = 1;
    if (seq[x] > 0 && color[seq[x]]) {
      right_minus = right_plus = left_plus = left_minus = 0;
    } else {
      right_minus = right_plus = left_plus = left_minus = -inf;
    }
  }
} t[maxn << 3];
void update(int k) {
  int a = t[k << 1].c1, b = t[k << 1].c2, c = t[k << 1 | 1].c1,
      d = t[k << 1 | 1].c2;
  t[k].dis = max(t[k << 1].dis, t[k << 1 | 1].dis);
  t[k].dis = max(t[k].dis, t[k << 1 | 1].left_minus + t[k << 1].right_plus);
  t[k].dis = max(t[k].dis, t[k << 1 | 1].left_plus + t[k << 1].right_minus);
  t[k].right_plus =
      max(t[k << 1].right_plus - c + d, t[k << 1].right_minus + c + d);
  t[k].right_plus = max(t[k].right_plus, t[k << 1 | 1].right_plus);
  t[k].left_plus =
      max(t[k << 1 | 1].left_plus + a - b, t[k << 1 | 1].left_minus + a + b);
  t[k].left_plus = max(t[k].left_plus, t[k << 1].left_plus);
  t[k].right_minus =
      max(t[k << 1].right_minus + c - d, t[k << 1 | 1].right_minus);
  t[k].left_minus = max(t[k << 1 | 1].left_minus + b - a, t[k << 1].left_minus);
  if (b < c) {
    t[k].c1 = a - b + c, t[k].c2 = d;
  } else {
    t[k].c1 = a, t[k].c2 = b - c + d;
  }
}
void dfs(int x, int rt) {
  seq[++sz] = 0;
  seq[++sz] = x;
  pos[x] = sz;
  for (int i = 0; i < G[x].size(); i++) {
    if (G[x][i] != rt)
      dfs(G[x][i], x);
  }
  seq[++sz] = -1;
}
void change(int k, int ps) {
  int l = t[k].l, r = t[k].r, mid = (l + r) >> 1;
  if (l == r) {
    t[k].ini(l);
    return;
  }
  if (ps <= mid)
    change(k << 1, ps);
  else
    change(k << 1 | 1, ps);
  update(k);
}
void build(int k, int l, int r) {
  t[k].l = l, t[k].r = r;
  int mid = (l + r) >> 1;
  if (l == r) {
    t[k].ini(l);
    return;
  }
  build(k << 1, l, mid);
  build(k << 1 | 1, mid + 1, r);
  update(k);
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  scanf("%d", &n);
  for (int i = 1; i < n; i++) {
    int x, y;
    scanf("%d %d", &x, &y);
    G[x].push_back(y);
    G[y].push_back(x);
  }
  dfs(1, 0);
  for (int i = 1; i <= n; i++)
    color[i] = 1;
  build(1, 1, sz);
  scanf("%d", &m);
  int black = n;
  while (m--) {
    opt = getchar();
    while (!((opt == 'G') || (opt == 'C')))
      opt = getchar();
    if (opt == 'G') {
      if (!black)
        puts("-1");
      else if (black == 1)
        puts("0");
      printf("%d\n", t[1].dis);
    }
    if (opt == 'C') {
      int x;
      scanf("%d", &x);
      if (color[x])
        black--;
      else
        black++;
      color[x] ^= 1;
      change(1, pos[x]);
    }
  }
}
```