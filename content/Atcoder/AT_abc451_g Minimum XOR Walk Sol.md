#线性基 #Trie 

前置题目：[P4151](https://www.luogu.com.cn/problem/P4151)| [[P4151 WC2011 最大 XOR 和路径 Sol|题解]]。

也是在 ABC 尝到 WC 的菜了。

先套路的使用 P4151 的结论，可以发现环的贡献依然可以任意加到答案中，链也是可以任意选择的。

> **猜你想看**
> 这里说明一下上面的是什么意思。考虑先建出 dfs 树，然后的话从 $u$ 到 $v$ 的路径可以转化为树上从 $u$ 到 $v$ 的链以及若干个环。可以证明我们能够任意增加一个环的贡献。
> 可以选择任意的链，是因为两个链会形成一个环，我们可以通过链异或环得到另一个链。

但是这需要我们求出所有的链，还是不可行的。但是这里我们有结论，设 $dis_u,dis_v$ 表示 $u,v$ 到根的最短距离，$r_u$，$r_v$ 表示通过线性基求得的最小异或和。则有 ${dis_{u,v}}_{min}=r_u \oplus r_v$。

证明：假设 $r_u \oplus r_v$ 在第 $k$ 位上为 `1` 使得答案不是最小的，则有 $r_u,r_v$ 必有一个在第 $k$ 位上为 `1`，然而与前面的**最小**矛盾了！因此结论正确。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int M = 60,N = 2e5 + 10;
struct xor_basis{
	LL a[M];
	void clear(){for(int i=M-1;i>=0;i--) a[i] = 0;}
	void insert(LL x){
		for(int i=M-1;i>=0;i--){
			if(x == 0) return ;
			if(((x >> i) & 1) && (a[i] == 0)){a[i] = x;return ;}
			else if((x >> i) & 1) x ^= a[i];
		}
	}
	LL query(LL x){
		for(int i=M-1;i>=0;i--)
			x = min(x,x^a[i]);
		return x;
	}
}xb;
struct Trie{
	int tot;
	struct node{
		int ch[2];
		LL cnt;
	}tre[N * 60];
	void clear(){
		for(int i=0;i<=tot;i++)
			tre[i].ch[0] = tre[i].ch[1] = tre[i].cnt = 0;
		tot = 0;
	}
	void insert(LL x){
		int u = 0;
		tre[u].cnt ++ ;
		for(int i=M-1;i>=0;i--){
			int fh = (x >> i) & 1;
			if(tre[u].ch[fh] == 0) tre[u].ch[fh] = ++tot;
			u = tre[u].ch[fh];
			tre[u].cnt ++ ;
		}
	}
	LL query(LL x,LL lim){
		LL sum = 0;
		int u = 0;
		for(int i=M-1;i>=0;i--){
			int fh = (x >> i) & 1;
			int fh2 = (lim >> i) & 1;
			if(fh2){
				if(tre[u].ch[fh]) sum += tre[tre[u].ch[fh]].cnt;
				if(tre[u].ch[fh^1])
					u = tre[u].ch[fh^1];
				else break;
			}
			else{
				if(tre[u].ch[fh]) u = tre[u].ch[fh];
				else break;
			}
			if(i == 0)
				sum += tre[u].cnt;
		}
		return sum;
	}
}trie;
vector< pair<int,LL> > G[N];
int n,m;
LL lim;
bool vis[N];
LL dep[N];
void init(){
	xb.clear();
	trie.clear();
	for(int i=1;i<=n;i++) {
		G[i].clear();
		dep[i] = vis[i] = 0;
	}
}
void dfs(int u,LL res){
	vis[u] = 1;dep[u] = res;
	for(auto [v,w] : G[u]){
		if(vis[v]) xb.insert(dep[v]^res^w);
		else dfs(v,res^w);
	}
}
void solve(){
	init();
	cin >> n >> m >> lim;
	for(int i=1;i<=m;i++){
		int u,v;LL w;
		cin >> u >> v >> w;
		G[u].push_back({v,w});
		G[v].push_back({u,w});
	}
	dfs(1,0);
	LL ans = 0;
	for(int i=1;i<=n;i++){
		LL x = xb.query(dep[i]);
		ans += trie.query(x,lim);
		trie.insert(x);
	}
	cout << ans << "\n";
}
int main(){
	IOS;
	int T;
	cin >> T;
	while(T -- ) solve();
	return 0;
}
```
