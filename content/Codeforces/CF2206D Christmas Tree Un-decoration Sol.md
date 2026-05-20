#线段树 #树链剖分 
[题目链接](https://www.luogu.com.cn/problem/CF2206D)

我还是不太擅长这种节点可能记录无用信息的线段树。

## DP
先想想 $O(n^2)$ 单次的DP。定义 $f_u$ 为处理完以 $u$ 为根的子树所需要的答案。那么显然答案为 $\max(\sum\limits_{v\in u} f_v,a_u)$。

但是我们看着数据范围，肯定想尝试上树剖进行维护，定义 $b_u$ 表示 $u$ 节点所有**轻儿子**的 $dp$ 之和。则答案变成 $max(b_u+dp_{son_v},a_u)$。

## 线段树维护
假设现在存在一条链 $c_1,c_2,c_3,c_4,\dots,c_{len}$，则 $dp_{c_1} = \max(a_{c_1},b_{c_1} + dp_{c_2})$，展开，有 $dp_{c_1} = \max(a_{c_1},b_{c_1} + b_{c_2} + dp_{c_3},b_{c_1}+a_{c_2})$。不难发现答案实际上为 $\max\limits_{i=1}^{len}(a_i + \sum\limits_{j=1}^{i-1} b_j)$。

因此我们尝试使用线段树和树剖进行维护，让线段树的每一个节点记录 $s$ 和 $mx$ 代表着 $b$ 和这段区间内答案的最大值，需要注意的是，这里的答案并不考虑不在这段区间内的下面的链，这是需要我们在查询时查询一段完整的链实现的。具体的，可以看代码。

当然，可能会有疑问，线段树上的节点不是可能记录错的信息吗？实际上我们不会用到错误的信息的，在每次查询是，我们将范围严格限制在一条链上，因此不会访问到错误的信息。

时间复杂度 $O(n \log^2 n )$。
## Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e5 + 10;
vector<int> G[N];
int n,Q;
int a[N];
int fa[N];
int siz[N],son[N];
int dfn[N],idfn[N],top[N],ed[N],tot;
void dfs1(int u){
	siz[u] = 1;
	for(int v : G[u]){
		dfs1(v);
		siz[u] += siz[v];
		if(siz[son[u]] < siz[v])
			son[u] = v;
	}
	return ;
}
LL b[N],dp[N];
void dfs2(int u,int tp){
	dfn[u] = ++tot;idfn[tot] = u;top[u] = tp;ed[tp] = u;
	if(son[u])
		dfs2(son[u],tp);
	for(int v : G[u]){
		if(v == son[u]) continue;
		dfs2(v,v);
		b[u] += dp[v];
	}
	if(son[u])
		dp[u] = max(1ll * a[u],dp[son[u]]+b[u]);
	else
		dp[u] = a[u];
	return ;
}
struct node{
	LL s,mx;
};
node merge(node x,node y){
	return {x.s+y.s,max(x.mx,x.s+y.mx)};
}
struct Segment_Tree{
	node tre[N<<2];
	void build(int pl,int pr,int p){
		if(pl == pr){
			tre[p] = {b[idfn[pl]],a[idfn[pl]]};
			tre[p].mx = max(tre[p].s,tre[p].mx);//注意：这里不能将mx设置为 dp_u，因为真正要得到 dp_u 需要通过链上的合并得到。
			return ;
		}
		int mid = pl + pr >> 1;
		build(pl,mid,p*2);
		build(mid+1,pr,p*2+1);
		tre[p] = merge(tre[p*2],tre[p*2+1]);
		return ;
	}
	void clear(int pl,int pr,int p){
		if(pl > pr) return ;
		tre[p] = {0,0};
		if(pl == pr)
			return ;
		int mid = pl + pr >> 1;
		clear(pl,mid,p*2);
		clear(mid+1,pr,p*2+1);
		return ;
	}
	void update(int x,int pl,int pr,int p,node t){
		if(pl == pr){
			if(t.s != -1) tre[p].s = t.s;
			if(t.mx != -1) tre[p].mx = t.mx;
			tre[p].mx = max(tre[p].s,tre[p].mx);
			return ;
		}
		int mid = pl + pr >> 1;
		if(x <= mid) update(x,pl,mid,p*2,t);
		else update(x,mid+1,pr,p*2+1,t);
		tre[p] = merge(tre[p*2],tre[p*2+1]);
		return ;
	}
	node query(int L,int R,int pl,int pr,int p){
		if(L <= pl && pr <= R) return tre[p];
		int mid = pl + pr >> 1;
		if(L <= mid && mid < R)
			return merge(query(L,R,pl,mid,p*2),query(L,R,mid+1,pr,p*2+1));
		if(L <= mid)
			return query(L,R,pl,mid,p*2);
		return query(L,R,mid+1,pr,p*2+1);
	}
}sgt;
void solve(){
	tot = 0;
	for(int i=1;i<=n;i++){
		G[i].clear();
		dp[i] = fa[i] = b[i] = 0;
		top[i] = dfn[i] = ed[i] = idfn[i] = 0;
		son[i] = 0;
	}
	sgt.clear(1,n,1);
	cin >> n >> Q;
	for(int i=2;i<=n;i++){
		int x;cin >> x;
		G[x].push_back(i);
		fa[i] = x;
	}
	for(int i=1;i<=n;i++)
		cin >> a[i];
	dfs1(1);
	dfs2(1,1);
	sgt.build(1,n,1);
	cout << sgt.query(dfn[top[1]],dfn[ed[1]],1,n,1).mx << "\n";
	while(Q -- ){
		int u,x;
		cin >> u >> x;
		a[u] = x;
		LL pre = sgt.query(dfn[top[u]],dfn[ed[top[u]]],1,n,1).mx;
		sgt.update(dfn[u],1,n,1,{-1,x});
		LL nw = sgt.query(dfn[top[u]], dfn[ed[top[u]]], 1, n, 1).mx;
		u = fa[top[u]];
		while(u){
			node sum = sgt.query(dfn[u],dfn[u],1,n,1);
			LL pree = sgt.query(dfn[top[u]],dfn[ed[top[u]]],1,n,1).mx;
			sgt.update(dfn[u],1,n,1,{sum.s-pre+nw,a[u]});
			pre = pree;
			nw = sgt.query(dfn[top[u]],dfn[ed[top[u]]],1,n,1).mx;
			u = fa[top[u]];
		}
		cout << sgt.query(dfn[top[1]],dfn[ed[1]],1,n,1).mx << "\n";
	}
}
int main(){
	IOS;
	// File("tree");
	int T;
	cin >> T;
	while(T -- )
		solve();
	return 0;
}
```