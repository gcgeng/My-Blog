---
title: '[uva12170]Easy Climb'
tags: []
date: 2017-01-04 17:54:00
---

还是挺难的一个题，看了书上的解析以后还是不会写，后来翻了代码仓库，发现lrj又用了一些玄学的优化技巧。
```
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

typedef long long LL;

const int maxn = 100 + 5;
const int maxx = maxn * maxn * 2;
const LL INF = (1LL << 60);

LL h[maxn], x[maxx], dp[2][maxx];

int main() {
  int T;
  cin >> T;
  while (T--) {
    int n;
    LL d;
    cin >> n >> d;
    for (int i = 0; i < n; i++)
      cin >> h[i];
    if (abs(h[0] - h[n - 1]) > (n - 1) * d) { //-specially judge impossible
      cout << "impossible\n";
      continue;
    }

    // useful heights  //-?
    int nx = 0;
    for (int i = 0; i < n; i++)
      for (int j = -n + 1; j <= n - 1; j++)
        x[nx++] = h[i] + j * d;
    sort(x, x + nx);
    nx = unique(x, x + nx) - x;

    // dp
    int t = 0;
    for (int i = 0; i < nx; i++) {
      dp[0][i] = INF;
      if (x[i] == h[0])
        dp[0][i] = 0;
    }
    for (int i = 1; i < n; i++) {
      int k = 0;
      for (int j = 0; j < nx; j++) {
        while (k < nx && x[k] < x[j] - d)
          k++;
        while (k + 1 < nx && x[k + 1] <= x[j] + d && dp[t][k + 1] <= dp[t][k])
          k++; // min in sliding window
        if (dp[t][k] == INF)
          dp[t ^ 1][j] = INF; // (t, k) is not reachable
        else
          dp[t ^ 1][j] = dp[t][k] + abs(x[j] - h[i]);
      }
      t ^= 1;
    }
    for (int i = 0; i < nx; i++)
      if (x[i] == h[n - 1])
        cout << dp[t][i] << "\n";
  }
  return 0;
}

```
首先这是一个背包类型的问题，朴素算法不仅会tle，还会mle，所有我们可以分析问题，得到一个O(n2)的状态设计。
其次，为了方便枚举，lrj使用了一个数组存储了所有可能用到的更新状态，这是一种坐标离散化的技巧，不然数组会过大。
然后，为了节省枚举时所花费的复杂度，lrj使用了单调队列来优化转移，同时，考虑到f[i-1]先单调减，又单调加，不需要维护完整的队列，只需要维护最小值即可。
最后使用了滚动数组进行优化。
这是一道经典题。