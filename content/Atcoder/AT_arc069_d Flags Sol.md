> 仿佛全世界 静止在这一秒

[题目链接](https://www.luogu.com.cn/problem/AT_arc069_d)

经典好题，使用线段树优化 2-SAT 建图。

看到这种只能二选一，还有限制的题目，肯定想要 2-SAT。

又看到这种最小值最大的问题，尝试二分。

但是现在的问题是二分得到的答案不好判断。如果直接建边，是 $O(n^2)$，我们不能接受，因此需要优化。

此时，可以使用线段树。考虑将 $x,y$ 进行排序，那么我们建边就是一个点对于一段区间进行建边，然后区间又可以拆分成线段树上的几个点，因此边的数量降到了 $O(n \log n)$ 级别，可以接受了。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e4 + 10;
struct node{
	int x,y;
}a[N];
int n;
pair<int,int> num[N<<1];
vector<int> G[N << 2];
int tot = 0;
struct segment_Tree{
	int tre[N << 2];
	void build(int pl,int pr,int p){
		if(pl == pr){
			tre[p] = num[pl].se;
			return ;
		}
		tre[p] = ++tot;
		int mid = pl + pr >> 1;
		build(pl,mid,p*2);
		build(mid+1,pr,p*2+1);
		G[tre[p]].push_back(tre[p*2]);
		G[tre[p]].push_back(tre[p*2+1]);
		return ;
	}
	void update(int L,int R,int pl,int pr,int p,int u){
		if(L > R) return ;
		if(L <= pl && pr <= R){
			G[u].push_back(tre[p]);
			return ;
		}
		int mid = pl + pr >> 1;
		if(L <= mid)
			update(L,R,pl,mid,p*2,u);
		if(R > mid)
			update(L,R,mid+1,pr,p*2+1,u);
		return ;
	}
}sgt;
int col[N << 2],cnt_col;
int dfn[N << 2],ins[N << 2],low[N << 2],top,sta[N << 2];
int idx;
void dfs(int u){
	dfn[u] = low[u] = ++idx;
	sta[++top] = u;
	ins[u] = 1;
	for(int v : G[u]){
		if(dfn[v] == 0){
			dfs(v);
			low[u] = min(low[u],low[v]);
		}
		else if(ins[v])
			low[u] = min(low[u],dfn[v]);
	}
	if(low[u] == dfn[u]){
		cnt_col ++ ;
		int v;
		do{
			v = sta[top--];
			ins[v] = 0;
			col[v] = cnt_col;
		}while(u != v);
	}
}
bool chk(int x){
	top = cnt_col = idx = 0;
	for(int i=1;i<=tot;i++){
		G[i].clear();
		col[i] = 0;
		dfn[i] = low[i] = ins[i] = 0;
	}
	tot = n * 2;
	sgt.build(1,n<<1,1);
	for(int i=1;i<=n*2;i++){
		int L,R;
		L = (n<<1)+1,R = 0;
		int l,r;
		l = 1,r = i-1;
		while(l <= r){
			int mid = l + r >> 1;
			if(num[i].fi - num[mid].fi < x){
				L = mid;
				r = mid - 1;
			}
			else l = mid + 1;
		}
		l = i + 1,r = n << 1;
		while(l <= r){
			int mid = l +  r >> 1;
			if(num[mid].fi - num[i].fi < x){
				R = mid;
				l = mid + 1;
			}
			else r = mid - 1;
		}
		int id;
		if(num[i].se > n)
			id = num[i].se - n;
		else
			id = num[i].se + n;
		sgt.update(L,i-1,1,n<<1,1,id);
		sgt.update(i+1,R,1,n<<1,1,id);
	}
	for(int i=1;i<=tot;i++){
		if(dfn[i] == 0)
			dfs(i);
	}
	for(int i=1;i<=n;i++)
		if(col[i] == col[i+n])
			return 0;
	return 1;
}
int main(){
	IOS;
	cin >> n;
	for(int i=1;i<=n;i++)
		cin >> a[i].x >> a[i].y;
	for(int i=1;i<=n;i++){
		num[i*2-1].fi = a[i].x;
		num[i*2-1].se = i + n;
		num[i*2].fi = a[i].y;
		num[i*2].se = i;
	}
	sort(num+1,num+1+n*2);
	int l = 1,r = int(1e9);
	int ans = 0;
	while(l <= r){
		int mid = l + r >> 1;
		if(chk(mid)){
			ans = mid;
			l = mid + 1;
		}
		else
			r = mid - 1;
	}
	cout << ans;
	return 0;
}
```