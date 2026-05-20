#珂朵莉树 #树状数组 
[题目链接](https://www.luogu.com.cn/problem/P9320)

前置题目：[CF1638E](https://www.luogu.com.cn/problem/CF1638E)。

和 CF1638E 一样的套路。

不妨将所在的节点转变为颜色，同样的维护区间推平，颜色加，单点询问，唯一要变的就是还有树上距离的事情了。

如果不会，贴个[[CF1638E Colorful Operations Sol|我的题解]]。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
#define IT set<node>::iterator
const int N = 2e5 + 10;
vector<int> G[N];
LL dep[N];
int fa[N][20];
void dfs(int u,int rt){
	fa[u][0] = rt;
	dep[u] = dep[rt] + 1;
	for(int i=1;i<=19;i++)
		fa[u][i] = fa[fa[u][i-1]][i-1];
	for(int v : G[u]){
		if(v == rt) continue;
		dfs(v,u);
	}
	return ;
}
int LCA(int x,int y){
	if(dep[x] < dep[y]) swap(x,y);
	for(int i=19;i>=0;i--)
		if(dep[fa[x][i]] >= dep[y]) x = fa[x][i];
	if(x == y) return x;
	for(int i=19;i>=0;i--)
		if(fa[x][i] != fa[y][i]){
			x = fa[x][i];
			y = fa[y][i];
		}
	return fa[x][0];
}
LL dist(int x,int y){
	int lca = LCA(x,y);
	return dep[x] + dep[y] - 2ll * dep[lca];
}
int n,m,Q;
struct BIT{
	LL tre[N];
	int lowbit(int x){return x & (-x);}
	void update(int x,LL k){
		while(x <= m){
			tre[x] += k;
			x += lowbit(x);
		}return ;
	}
	void update(int L,int R,LL x){
		update(L,x);update(R+1,-x);
	}
	LL query(int x){
		LL sum = 0;
		while(x){
			sum += tre[x];
			x -= lowbit(x);
		}
		return sum;
	}
}tre;
LL tag[N];
struct ODT{
	struct node{
		int l,r;
		mutable int c;
		node(int L,int R=-1,int c=0):l(L),r(R),c(c){};
		bool operator < (const node &o) const {return l < o.l;}
	};
	set<node> st;
	IT split(int pos){
		IT it = st.lower_bound(node(pos));
		if(it != st.end() && it -> l == pos) return it;
		it -- ;
		int L,R;LL col;
		L = it -> l,R = it -> r,col = it -> c;
		st.erase(it);
		st.insert(node(L,pos-1,col));
		return st.insert(node(pos,R,col)).fi;
	}
	void assign(int l,int r,int c){
		IT itr = split(r+1),itl = split(l);
		IT it = itl;
		for(;it!=itr;it++){
			int L,R,col;
			L = it -> l,R = it -> r,col = it -> c;
			tre.update(L,R,tag[col]);
			tre.update(L,R,-dist(c,col));
		}
		tre.update(l,r,-tag[c]);
		st.erase(itl,itr);
		st.insert(node(l,r,c));
	}
	int get_tag(int pos){
		IT it = st.lower_bound(node(pos));
		if(it != st.end() && it -> l == pos) return it -> c;
		return (--it) -> c;
	}
}odt;
int main(){
	IOS;
	// File("travel");
	cin >> n >> m >> Q;
	for(int i=1;i<=m;i++){
		int x;
		cin >> x;
		odt.st.insert(ODT::node(i,i,x));
	}
	for(int i=1;i<n;i++){
		int u,v;
		cin >> u >> v;
		G[u].push_back(v);
		G[v].push_back(u);
	}
	dfs(1,0);
	while(Q -- ){
		char c;
		cin >> c;
		if(c == 't'){
			int l,r,c;
			cin >> l >> r >> c;
			odt.assign(l, r, c);
		}
		if(c == 'e'){
			int c,d;
			cin >> c >> d;
			tag[c] += d;
		}
		if(c == 'q'){
			int x;
			cin >> x;
			cout << tre.query(x) + tag[odt.get_tag(x)] << "\n";
		}
	}
	return 0;
}
```