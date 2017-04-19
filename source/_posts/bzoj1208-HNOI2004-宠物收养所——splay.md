---
title: '[bzoj1208][HNOI2004]宠物收养所——splay'
tags: [数据结构, splay]
date: 2017-03-01 09:39:00
---

#题目大意
Description

最近，阿Q开了一间宠物收养所。收养所提供两种服务：收养被主人遗弃的宠物和让新的主人领养这些宠物。每个领养者都希望领养到自己满意的宠物，阿Q根据领养者的要求通过他自己发明的一个特殊的公式，得出该领养者希望领养的宠物的特点值a（a是一个正整数，a<2^31），而他也给每个处在收养所的宠物一个特点值。这样他就能够很方便的处理整个领养宠物的过程了，宠物收养所总是会有两种情况发生：被遗弃的宠物过多或者是想要收养宠物的人太多，而宠物太少。 1． 被遗弃的宠物过多时，假若到来一个领养者，这个领养者希望领养的宠物的特点值为a，那么它将会领养一只目前未被领养的宠物中特点值最接近a的一只宠物。（任何两只宠物的特点值都不可能是相同的，任何两个领养者的希望领养宠物的特点值也不可能是一样的）如果有两只满足要求的宠物，即存在两只宠物他们的特点值分别为a-b和a+b，那么领养者将会领养特点值为a-b的那只宠物。 2． 收养宠物的人过多，假若到来一只被收养的宠物，那么哪个领养者能够领养它呢？能够领养它的领养者，是那个希望被领养宠物的特点值最接近该宠物特点值的领养者，如果该宠物的特点值为a，存在两个领养者他们希望领养宠物的特点值分别为a-b和a+b，那么特点值为a-b的那个领养者将成功领养该宠物。 一个领养者领养了一个特点值为a的宠物，而它本身希望领养的宠物的特点值为b，那么这个领养者的不满意程度为abs(a-b)。【任务描述】你得到了一年当中，领养者和被收养宠物到来收养所的情况，希望你计算所有收养了宠物的领养者的不满意程度的总和。这一年初始时，收养所里面既没有宠物，也没有领养者。
Input

第一行为一个正整数n，n<=80000，表示一年当中来到收养所的宠物和领养者的总数。接下来的n行，按到来时间的先后顺序描述了一年当中来到收养所的宠物和领养者的情况。每行有两个正整数a, b，其中a=0表示宠物，a=1表示领养者，b表示宠物的特点值或是领养者希望领养宠物的特点值。（同一时间呆在收养所中的，要么全是宠物，要么全是领养者，这些宠物和领养者的个数不会超过10000个）
Output

仅有一个正整数，表示一年当中所有收养了宠物的领养者的不满意程度的总和mod 1000000以后的结果。
#题解
显然可以用splay维护信息。我比较蠢，开了两个splay。
#注意的地方
1\. 还是模板不熟，在del时候忘记更改fa的拓扑结构，导致一直tle。
2\. 被mod坑了。明明我模的比较多，却一直WA，黄学长mod的很少，却过了Orz。
以后要注意减法的mod。能不mod就不要mod。

#代码
```c++
#include <cstdio>
#include <cstdlib>
const int maxn = 160000;
const int mod = 1000000;
int data[maxn], fa[maxn], ch[maxn][2];
int sz[3], rt1 = 0, rt2 = 0; // splay1:pets||splay2:men
int n, a, b, ans = 0;
void zig(int x) {
  int y = fa[x], z = fa[y], l = (ch[y][1] == x), r = l ^ 1;
  fa[ch[y][l] = ch[x][r]] = y;
  fa[ch[x][r] = y] = x;
  fa[x] = z;
  if (z)
    ch[z][ch[z][1] == y] = x;
}
void splay(int x, int aim, int &rt) {
  for (int y; (y = fa[x]) != aim; zig(x)) {
    if (fa[y] != aim)
      zig((ch[fa[y]][0] == y) == (ch[y][0] == x) ? y : x);
  }
  if (aim == 0)
    rt = x;
}
void insert(int v, int &rt, int arg) {
  int x = rt;
  if (rt == 0) {
    rt = x = ++sz[arg];
    data[x] = v;
    fa[x] = ch[x][0] = ch[x][1] = 0;
    return;
  }
  while (x) {
    if (data[x] == v)
      return;
    int &y = ch[x][v > data[x]];
    if (y == 0) {
      y = ++sz[arg];
      data[y] = v;
      fa[y] = x;
      ch[y][0] = ch[y][1] = 0;
      x = y;
      break;
    }
    x = y;
  }
  splay(x, 0, rt);
}
int pre(int v, int &rt) {
  int x = rt;
  int y = -1;
  while (x) {
    if (v >= data[x]) {
      y = x;
      x = ch[x][1];
    } else
      x = ch[x][0];
  }
  return y;
}
int succ(int v, int &rt) {
  int x = rt;
  int y = -1;
  while (x) {
    if (v <= data[x]) {
      y = x;
      x = ch[x][0];
    } else
      x = ch[x][1];
  }
  return y;
}
void del(int x, int &rt) {
  splay(x, 0, rt);
  int y = ch[x][0];
  while (ch[y][1])
    y = ch[y][1];
  if (y == 0) {
    rt = ch[x][1];
    fa[ch[x][1]] = 0;
    return;
  }
  splay(y, x, rt);
  ch[y][1] = ch[x][1];
  rt = y;
  fa[y] = 0;
  fa[ch[x][1]] = rt;
  return;
}
int main() {
#ifdef D
  freopen("input", "r", stdin);
#endif
  sz[1] = 0;
  sz[2] = 80000;
  scanf("%d", &n);
  for (int i = 1; i <= n; i++) {
    scanf("%d %d", &a, &b);
    if (a == 0) {
      if (rt1 == 0) { // splay is empty
        insert(b, rt2, 2);
      } else {
        int x = pre(b, rt1);
        int y = succ(b, rt1);
        if (x != -1 && y != -1 && (b - data[x]) <= (data[y] - b)) {
          del(x, rt1);
          ans += b - data[x];
          ans %= mod;
        } else if (y != -1) {
          del(y, rt1);
          ans += data[y] - b;
          ans %= mod;
        } else if (x != -1 && y == -1) {
          del(x, rt1);
          ans += b - data[x];
          ans %= mod;
        } else if (x == -1 && y != -1) {
          del(y, rt1);
          ans += data[y] - b;
          ans %= mod;
        }
      }
    }
    if (a == 1) {
      if (rt2 == 0) { // splay is empty
        insert(b, rt1, 1);
      } else {
        int x = pre(b, rt2);
        int y = succ(b, rt2);
        if (x != -1 && y != -1 && (b - data[x]) <= (data[y] - b)) {
          del(x, rt2);
          ans += b - data[x];
          ans %= mod;
        } else if (x != -1 && y != -1) {
          del(y, rt2);
          ans += data[y] - b;
          ans %= mod;
        } else if (x != -1 && y == -1) {
          del(x, rt2);
          ans += b - data[x];
          ans %= mod;
        } else if (x == -1 && y != -1) {
          del(y, rt2);
          ans += data[y] - b;
          ans %= mod;
        }
      }
    }
  }
  printf("%d\n", ans);
}
```