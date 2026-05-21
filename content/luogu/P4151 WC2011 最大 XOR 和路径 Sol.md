#线性基 
[题目链接](https://www.luogu.com.cn/problem/P4151)。

依旧看到异或尝试使用01trie或者线性基。

发现这道题目 01trie 大概是不好写的，因此考虑线性基。

不难将问题转化成链+环的形式，即在链上再加上若干环的贡献。

可以证明的是，环的贡献是可以任意加的，假设我们现在在某一位置上，我们就可以去到环上再回来，只会产生环的贡献。

但是这条链怎么找？假设现在存在 $A,B$ 两条链，其中 $A$ 劣于 $B$，但是我们当前选择的是 $A$，那么我么就会选择 $A,B$ 两条链形成的环，直接从 $A$ 切换到 $B$。因此我们可以找任意的链。

而后就是线性基的事情了。将环上的值全部插入线性基中，查找是板的。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 60,M = 5e4 + 10;
int n,m;
LL a[N];
void insert(LL x){
	for(int i=N-1;i>=0;i--){
		if(x == 0) return ;
		if(((x >> i) & 1) && (a[i] == 0)){a[i] = x;return ;}
		else if((x >> i) & 1) x ^= a[i];
	}
	return ;
}
LL query(LL x){
	for(int i=N-1;i>=0;i--)
		x = max(x,x^a[i]);
	return x;
}
vector<pair<int,LL>> G[M];
bool vis[M];
LL dis[M];
void dfs(int u,LL res){
	dis[u] = res;
	vis[u] = 1;
	for(auto[v,w] : G[u]){
		if(vis[v]) insert(dis[v]^res^w);
		else dfs(v,res^w);
	}
	return ;
}
int main(){
	IOS;
	cin >> n >> m;
	for(int i=1;i<=m;i++){
		int u,v;LL w;
		cin >> u >> v >> w;
		G[u].push_back({v,w});
		G[v].push_back({u,w});
	}
	dfs(1,0);
	cout << query(dis[n]);
	return 0;
}
```
