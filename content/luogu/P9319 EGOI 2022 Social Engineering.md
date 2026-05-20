#交互 #思维

交互大人，拼尽全力无法战胜的。构造是构造不出来的，判定是不会判定的。

[题目链接](https://www.luogu.com.cn/problem/P9319)

## 思路
定义：与 `1` 直接有边相连的点为关键点。

那么我们不难证明的是，在删去与 `1` 有关的边后，所形成的联通块内如果有一个联通块满足关键点个数为奇数，则玛丽亚必胜。因为每一次选择 $u,v$ 两个关键点删去，必然还会有一点剩余。然后就输了。

因此，我们可以发现 “删去 `1` 号点后，联通块内关键点的个数为偶数” 是我们胜利的必要条件，接下来将从构造角度说明这一条件同样是充分的。

考虑在联通块内建出生成树。那么我们不难发现每一条边都只会有一组点使用这条边。具体的，考虑建出树后，从下往上尝试配对，那么对于节点 $u$，不难发现对于其任意一棵子树，其尝试来的关键点最多只有一个（剩下的都在里面匹配掉的），同时，继承上去的节点也至多为 `1`。而根据上面的判定条件，这样是一定能够完成配对的。

具体可以看代码。

## Code

```cpp
#include<bits/stdc++.h>
#include "socialengineering.h"
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e5 + 10;
int GetMove();
void MakeMove(int v);
int fa[N],cnt[N];
int to[N],ned[N];
int fath[N];
int find(int u){
	if(u == fa[u]) return u;
	return fa[u] = find(fa[u]);
}
int merge(int u,int v){
	u = find(u),v = find(v);
	if(u != v){
		cnt[u] += cnt[v];
		fa[v] = u;
		return 1;
	}
	return 0;
}
vector< vector<int> > G;
vector< bool > vis;
int dep[N];
void dfs(int u,int rt){
	if(vis[u]) ned[u] = u;
	else ned[u] = -1;
	fath[u] = rt;
	dep[u] = dep[rt] + 1;
	for(int v : G[u]){
		if(v == rt) continue;
		dfs(v,u);
		if(ned[v] > 0){
			if(ned[u] > 0){
				to[ned[u]] = ned[v];
				to[ned[v]] = ned[u];
				ned[u] = -1;
			}
			else
				ned[u] = ned[v];
		}
	}
	return ;
}
int LCA(int u,int v){
	if(dep[u] < dep[v]) swap(u,v);
	while(dep[u] > dep[v]) u = fath[u];
	while(u != v){
		u = fath[u],v = fath[v];
	}
	return u;
}
void print(int x){
	int u,v;
	u = x;
	v = to[x];
	int lca = LCA(u,v);
	while(u != lca){
		MakeMove(fath[u]);
		u = fath[u];
	}
	vector<int> vec;
	vec.clear();
	while(v != lca){
		vec.push_back(v);
		v = fath[v];
	}
	reverse(vec.begin(),vec.end());
	for(int num : vec)
		MakeMove(num);
	MakeMove(1);
	return ;
}
void SocialEngineering(int n, int m, vector<pair<int,int>> edges){
	for(int i=1;i<=n;i++)
		fa[i] = i;
	G = vector<vector<int>>(n+1);
	vis = vector< bool > (n+1);
	for(auto [u,v] : edges){
		if(u == 1) cnt[v] = 1,vis[v] = 1;
		if(v == 1) cnt[u] = 1,vis[u] = 1;
	}
	for(auto [u,v] : edges){
		if(u == 1 || v == 1) continue;
		if(merge(u,v)){
			G[u].push_back(v);
			G[v].push_back(u);
		}
	}
	for(int i=2;i<=n;i++)
		if(vis[i])
			if(cnt[find(i)] & 1)
				return ;
	for(int i=2;i<=n;i++){
		if(dep[i]) continue;
		dfs(i,0);
	}
	int x;
	while(1){
		x = GetMove();
		if(x == 0) return ;
		print(x);
	}
}
```