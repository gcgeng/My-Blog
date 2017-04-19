---
title: '[bzoj1087][scoi2005]互不侵犯king'
tags: [动态规划, 状压DP]
date: 2017-01-06 17:35:00
---

#题目大意
在N×N的棋盘里面放K个国王，使他们互不攻击，共有多少种摆放方案。国王能攻击到它上下左右，以及左上
左下右上右下八个方向上附近的各一个格子，共8个格子。
#思路
首先，搜索可以放弃，因为这是一个计数问题，正解几乎不可能是搜索。
我们考虑这样一个决策过程：对于每一行，我们决定放哪些格子。这个决策过程很明显满足无后效性和最优子结构，同时，根据上一行可以递推出这一行的所有可行方案。
所以，我们考虑使用动态规划。
怎么划分阶段呢？根据我们以上的推理，很显然可以根据行来划分阶段。
怎么转移呢？在转移的时候，我们要考虑放的king的个数，所以要把king的个数加入状态。其次，为了让king互不侵犯，我们要存储这一行里哪些格子放了king，用一个二进制状态存储，写入状态。
很容易写出转移方程。
f[i][j][s] += f[i-1][j-cnt[j]][pre] 事实上，这更像一个递推。
下面给出代码。
#Code

```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 10;
const int maxs = (1 << 10) + 1;
int n, k, stat[maxs], m = 0;
bool mp[maxs][maxs];
long long f[maxn][maxn * maxn][maxs];
int ct[maxs];
void dfs(int p, int cnt, int x) {
  if (p >= n || cnt > k)
    return;
  stat[m++] = x;
  ct[m - 1] = cnt;
  for (int i = p + 1; i < n; i++) {
    if (abs(p - i) > 1 || p == -1)
      dfs(i, cnt + 1, x | (1 << i));
  }
}
int main() {
  scanf("%d%d", &n, &k);
  dfs(0 - 1, 0, 0);
  for (int i = 0; i < m; i++) {
    for (int j = 0; j < m; j++) {
      mp[i][j] = mp[j][i] = (stat[i] & stat[j]) || (stat[i] >> 1 & stat[j]) ||
                                    (stat[j] >> 1 & stat[i])
                                ? 0
                                : 1;
    }
  }
  for (int i = 0; i < m; i++) {
    f[0][ct[i]][i] = 1LL;
  }
  for (int i = 1; i < n; i++) {
    for (int j = 0; j <= k; j++) {
      for (int now = 0; now < m; now++) {
        if (ct[now] > j)
          continue;
        for (int l = 0; l < m; l++) {
          if (mp[l][now] && ct[l] + ct[now] <= j) {
            f[i][j][now] += f[i - 1][j - ct[now]][l];
          }
        }
      }
    }
  }
  long long ans = 0;
  for (int i = 0; i < m; i++) {
    ans += f[n - 1][k][i];
  }
  printf("%lld", ans);
  return 0;
}

```