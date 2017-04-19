---
title: P1079 Vigenère 密码
tags: []
date: 2016-11-13 09:12:00
---

```
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1005;
int main() {
	freopen("input.in", "r", stdin);
	char mi[maxn], key[105];
	scanf("%s%s", key, mi);
	char ming[maxn];
	for(int i = 0; i < strlen(mi); i++) {
		int x = tolower(key[i%strlen(key)]) - 'a';
		int y;
		if(mi[i] >= 'a' && mi[i] <= 'z') {
			y = mi[i] - 'a' - x;
			if(y < 0) y += 26; 
			y += 'a';
		}
		if(mi[i] >= 'A' && mi[i] <= 'Z') {
			y = mi[i] - 'A' - x;
			if(y < 0) y += 26; 
			y += 'A';
		}
		ming[i] = y;
	}
	for(int i = 0; i < strlen(mi); i++) cout << ming[i];
	return 0;
}
```
开始wa，以后要注意输出字符串要注意