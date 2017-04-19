---
title: P2661 信息传递 TODO-TARJAN算法
tags: []
date: 2016-11-01 21:01:00
---

http://www.cnblogs.com/zbtrs/p/5762788.html
http://blog.csdn.net/loi_yzs/article/details/52795093 
都是不错的解析，我在这里就不多说了
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 500005;
int to[maxn], in[maxn], vis[maxn];
queue<int> q;
int main() {
    int n;
    cin >> n;
    memset(in, 0, sizeof(in));
    memset(vis, 0, sizeof(vis));
    for(int i = 1; i <= n; i++) {
        cin >> to[i];
        in[to[i]]++;
    }
    for(int i = 1; i <= n; i++) {
        if(in[i] == 0) {
            q.push(i);
            vis[i] = 1;
        }
    }
    while(!q.empty()) {
        int u = q.front();
        q.pop();
        in[to[u]]--;
        if(in[to[u]] == 0) {
            vis[to[u]] = 1;
            q.push(to[u]);
        }
    }
    int ans = maxn;
    for(int i = 1; i <= n; i++) {
    	int tmp = maxn;
        if(!vis[i]) {
            int j = to[i];
			tmp = 0;
            while(!vis[j]) {
                tmp++;
                vis[j] = 1;
                j = to[j];
            }
        }
        ans = min(ans, tmp);
    }
    cout << ans;
}
```