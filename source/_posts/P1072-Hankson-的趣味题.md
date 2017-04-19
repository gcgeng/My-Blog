---
title: P1072 Hankson 的趣味题
tags: []
date: 2016-11-16 19:36:00
---

```
#include<bits/stdc++.h>
#define inf 1000000000
#define ll long long
using namespace std;
int read()
{
    int x=0,f=1;char ch=getchar();
    while(ch<'0'||ch>'9'){if(ch=='-')f=-1;ch=getchar();}
    while(ch>='0'&&ch<='9'){x=x*10+ch-'0';ch=getchar();}
    return x*f;
}
int n,ans;
int a0,a1,b0,b1;
int gcd(int a,int b)
{
    return b==0?a:gcd(b,a%b);
}
ll lcm(int a,int b)
{
    return (ll)a*b/gcd(a,b);
}
void cal(int x)
{
    if(gcd(x,a0)==a1)
        if(lcm(x,b0)==b1)ans++;
}
int main()
{
    n=read();
    while(n--)
    {
        ans=0;
        a0=read();a1=read();b0=read();b1=read();
        for(int i=1;i<=sqrt(b1);i++)
            if(b1%i==0)
            {
                cal(i);
                if(i*i!=b1)cal(b1/i);
            }
        printf("%d\n",ans);
    }
    return 0;
}
```