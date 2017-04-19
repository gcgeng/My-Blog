---
title: '[bzoj3098]Hash Killer 2——哈希'
tags: [字符串哈希, 字符串]
date: 2017-02-25 11:04:00
---

# 题目
这天天气不错，hzhwcmhf神犇给VFleaKing出了一道题：
给你一个长度为N的字符串S，求有多少个不同的长度为L的子串。
子串的定义是S[l]、S[l + 1]、… S[r]这样连续的一段。
两个字符串被认为是不同的当且仅当某个位置上的字符不同。

VFleaKing一看觉得这不是Hash的裸题么！于是果断写了哈希 + 排序。
而hzhwcmhf神犇心里自然知道，这题就是后缀数组的height中 < L的个数 + 1，就是后缀自动机上代表的长度区间包含L的结点个数，就是后缀树深度为L的结点的数量。
但是hzhwcmhf神犇看了看VFleaKing的做法表示非常汗。于是想卡掉他。

VFleaKing使用的是字典序哈希，其代码大致如下：
```cpp
u64 val = 0;
for (int i = 0; i < l; i++)
val = (val * base + s[i] – ‘a’) % Mod;
u64是无符号int64，范围是[0, 2^64)。
```
base是一个常量，VFleaKing会根据心情决定其值。
Mod等于1000000007。
VFleaKing还求出来了base ^ l % Mod，即base的l次方除以Mod的余数，这样就能方便地求出所有长度为L的子串的哈希值。
然后VFleaKing给哈希值排序，去重，求出有多少个不同的哈希值，把这个数作为结果。
其算法的C++代码如下：
```cpp
typedef unsigned long long u64;

const int MaxN = 100000;

inline int hash_handle(const char *s, const int &n, const int &l, const int &base)
{
const int Mod = 1000000007;

u64 hash_pow_l = 1;
for (int i = 1; i <= l; i++)
hash_pow_l = (hash_pow_l * base) % Mod;

int li_n = 0;
static int li[MaxN];

u64 val = 0;
for (int i = 0; i < l; i++)
val = (val * base + s[i] – ‘a’) % Mod;
li[li_n++] = val;
for (int i = l; i < n; i++)
{
val = (val * base + s[i] – ‘a’) % Mod;
val = (val + Mod – ((s[i – l] – ‘a’) * hash_pow_l) % Mod) % Mod;
li[li_n++] = val;
}

sort(li, li + li_n);
li_n = unique(li, li + li_n) – li;
return li_n;
}
```
hzhwcmhf当然知道怎么卡啦！但是他想考考你。
# 题解
>如果一个房间里有23个或23个以上的人，那么至少有两个人的生日相同的概率要大于50%。
>生日攻击：如果你在n个数中随机选数，那么最多选√n次就能选到相同的数（不考虑Rp broken)
>同样的，这题的Hash值在0到1000000007.
>那就要选差不多10^5次
>唯一注意的是l要取大，使得方案数超过Mod
>否则就不可能有2个数有相同的Hash值

引自[hzwer](http://hzwer.com/1861.html)
# 代码
```cpp
#include<bits/stdc++.h>
using namespace std;
const int maxn = 100000;
int n=MAXN,l=20;
int main()
{
	cout<<n<<' '<<l<<endl;
	for(int i=1;i<=n;i++) cout<<char(rand()%26+'a');
	cout<<endl;
	return 0;
}
```