---
title: 'Codeforces Round #395 Div.2 题解'
tags: []
date: 2017-02-03 10:40:00
---

#感受
第一次参加CF的rating比赛，感觉还是非常exciting，前18分钟把AB切掉之后一直在考虑C题，结果最后还是没有想出来Orz
[传送门](http://codeforces.com/contest/764)

#A
比较水的模拟，就是计算：$\frac{z}{lcm(a,b)}$
```
#include <bits/stdc++.h>
#define ll long long
using namespace std;
int n, m, z;
int gcd(int n, int m) {
    return m == 0 ? n : gcd(m, n%m);
}

int main() {
    cin >> n >> m >> z;
    cout << (z/((ll)n*m/gcd(n, m)));
    return 0;
}
```

#B
水题。
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 200000;
int main() {
    int n, a[maxn];
    cin >> n;
    for(int i = 1; i <= n; i++) cin >> a[i];
    if(n&1){
        for(int i = 1; i <= n; i++) {
        if(i & 1) {
            cout << a[n-i+1];
        }
        else cout << a[i];
        cout << ' ';
        }
    }
    else {
        for(int i = 1; i <= n/2; i++) {
            if(i & 1) cout << a[n-i+1];
            else cout << a[i];
            cout << ' ';
        }
        for(int i = n/2+1; i <= n; i++) {
            if(i & 1) cout << a[i];
            else cout << a[n-i+1];
            cout << ' ';
        }
    } 
    return 0;
}
```

#C
考试的时候并没有切出来。。。
后来发现很简单。。。
##题目大意
给出一颗树及各个点的颜色，求一个点，使得以该点为根时，其所有子树（不包括整棵树）颜色相同。
##题解
解法1:$O(n+m)$：如果有一条边，其两端颜色不同，那么不难得到必然有一个点是根，分别使用dfs检查即可。
解法2:$O(n)$：统计所有颜色不同的边，如果有解，那么这些边一定有一个共同的端点，统计一下即可。

##代码
###考场tle代码
```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 100005;
vector<int> G[maxn];
int n, c[maxn];
int color;
inline int read() {
    int x = 0, f = 1; char ch = getchar();
    while(ch < '0' || ch > '9') {if(ch == '-') f = -1; ch = getchar();}
    while(ch >= '0' && ch <= '9') { x = x * 10 + ch - '0'; ch = getchar();}
    return x * f;
}
int vis[maxn];
bool dfs(int x) {
    vector<int>::iterator it;
    for(it = G[x].begin(); it != G[x].end(); it++) {
        int &v = *it;
        if(!vis[v]) {
            vis[v] = 1;
            if(c[v] != color) return false;
            if(!dfs(v)) return false;
        }
    }
    return true;
}

bool check(int x) {
    vector<int>::iterator it;
    for(it = G[x].begin(); it != G[x].end(); it++) {
        color = c[*it];
        vis[x] = 1;
        if(!dfs(*it)) return false;
    }
    return true;
}
int main() {
    scanf("%d", &n);
    for(int i = 1; i < n; i++) {
        int u = read(), v = read();
        G[u].push_back(v);
        G[v].push_back(u);
    }
    set<int> col;
    for(int i = 1; i <= n; i++) {c[i] = read(); col.insert(c[i]);}
    for(int i = 1; i <= n; i++) {
        if(G[i].size() < col.size() - 1) continue;
        memset(vis, 0, sizeof(vis));
        vis[i] = 1;
        if(check(i)) {
            cout << "YES" << endl << i;
            return 0;
        }
    }
    cout << "NO" << endl;
    return 0;
} 
```
###正解（解法2）
```
#include <bits/stdc++.h>
using namespace std;
#define FOR(i,a,b) for (int i = (a); i <= (b); i++)
#define FORD(i,a,b) for (int i = (a); i >= (b); i--)
#define REP(i,a) FOR(i,0,(int)(a)-1)
#define reset(a,b) memset(a,b,sizeof(a))
#define BUG(x) cout << #x << " = " << x << endl
#define PR(x,a,b) {cout << #x << " = "; FOR (_,a,b) cout << x[_] << ' '; cout << endl;}
#define CON(x) {cout << #x << " = "; for(auto i:x) cout << i << ' '; cout << endl;}
#define mod 1000000007
#define pi acos(-1)
#define eps 0.00000001
#define pb push_back
#define sqr(x) (x) * (x)
#define _1 first
#define _2 second

int n, u, v, lis[100005], cnt[100005], tot;
vector<int> adj[100005];

int main() {
  ios::sync_with_stdio(false);
  cin >> n;
  REP (i, n - 1) {
  	cin >> u >> v;
  	adj[v].pb(u);
  	adj[u].pb(v);
  }
  FOR (i, 1, n) cin >> lis[i];
  FOR (i, 1, n) {
  	for (int nex: adj[i]) if (lis[i] != lis[nex]) {
  		cnt[i]++;
  		cnt[nex]++;
  		tot++;
  	}
  }
  FOR (i, 1, n) if (cnt[i] == tot) {
  	cout << "YES" << endl << i;
  	return 0;
  }
  cout << "NO";
}
```

#D
##题意：
给出一些***边长为odd**的矩形，用最多***四种颜色***给矩形染色，使得相邻颜色不同。
##题解：
首先边长为odd，那么如果两个矩形相邻，那么他们左上顶点的***奇偶性***一定不同。
所以我们根据左上顶点的奇偶染色即可。
具体见代码。
##代码：
```
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
const int MAXN = 500011;
int n;
struct edge{
	int sx,sy;
	int xx,xy;
}a[MAXN];
inline int getint(){
    int w=0,q=0; char c=getchar(); while((c<'0'||c>'9') && c!='-') c=getchar();
    if(c=='-') q=1,c=getchar(); while (c>='0'&&c<='9') w=w*10+c-'0',c=getchar(); return q?-w:w;
}

inline void work(){
	n=getint();
	for(int i=1;i<=n;i++) {
		a[i].xx=getint(); a[i].xy=getint();
		a[i].sx=getint(); a[i].sy=getint();
		a[i].xx=abs(a[i].xx); a[i].xy=abs(a[i].xy);
	}
	printf("YES\n");
	for(int i=1;i<=n;i++) {
		if(a[i].xx%2==1 && a[i].xy%2==1) printf("1");
		else if(a[i].xx%2==1 && a[i].xy%2==0) printf("2");
		else if(a[i].xx%2==0 && a[i].xy%2==1) printf("3");
		else printf("4");
		printf("\n");
	}
}

int main()
{
    work();
    return 0;
}
```

#E
不会做。