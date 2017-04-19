---
title: '[UVa 1619]Feel Good'
tags: []
date: 2016-12-25 17:43:00
---

```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1000010;
struct node {
  int num;
  int pos;
};
int main() {
  //  freopen("input", "r", stdin);
  int n;
  int a[maxn], s[maxn];
  while (scanf("%d", &n) != EOF) {
    for (int i = 1; i <= n; i++)
      cin >> a[i];
    s[1] = a[1];
    for (int i = 2; i <= n; i++)
      s[i] = s[i - 1] + a[i];
    stack<node> aa, b;

    int fro[maxn], beh[maxn];
    //    aa.push((node){a[1], 1});
    //  b.push((node){a[n], n});
    fro[1] = 1;
    beh[n] = n;
    for (int i = 1; i <= n; i++) {
      node x;
      if (!aa.empty())
        x = aa.top();
      else
        x = (node){0, 0};
      while (x.num >= a[i] && x.pos != i && !aa.empty()) {
        aa.pop();
        if (!aa.empty())
          x = aa.top();
        else
          x = (node){0, 0};
      }
      aa.push((node){a[i], i});
      if (x.pos != 0)
        fro[i] = x.pos + 1;
      else
        fro[i] = 1;
    }
    for (int i = n; i >= 1; i--) {
      node x;
      if (!b.empty())
        x = b.top();
      else
        x = (node){0, 0};
      while (x.num >= a[i] && x.pos != i && !b.empty()) {
        b.pop();
        if (!b.empty())
          x = b.top();
        else
          x = (node){0, 0};
      }
      b.push((node){a[i], i});
      if (x.pos != 0)
        beh[i] = x.pos - 1;
      else
        beh[i] = n;
    }
    long long ans = 0;
    int L;
    int R;
    for (int i = 1; i <= n; i++) {
      long long x = a[i] * (s[beh[i]] - s[fro[i]] + a[fro[i]]);
      if (x > ans) {
        ans = x;
        L = fro[i];
        R = beh[i];
      }
    }
    cout << ans << endl << L << ' ' << R << endl;
  }
}
```