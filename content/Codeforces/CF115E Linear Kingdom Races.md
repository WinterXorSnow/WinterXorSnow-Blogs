#DP #线段树
[题目链接](https://www.luogu.com.cn/problem/CF115E)。

线段树优化DP。

赛时连 DP 状态都设计错了，放个假大脑自动退化这一块。

先尝试进行 $O(n^2)$ 的DP。一开始尝试对着比赛进行设，但是发现可行度很差。因此这里选用对着道路设计状态。

令 $f_i$ 表示处理到 $i$ 时的答案。则 $f_i = \max(f_{i-1},\max_{j=0}^{i-1} f_j - cost(j+1,i) + val(j+1,i))$，其中，$cost(x,y)$ 表示从 $x$ 修路到 $y$ 需要的花费，$val(x,y)$ 表示比赛带来的价值。

考虑使用线段树优化后面的东西。将里面的值拆开看。$f_j$ 的处理是显然的，cost则可以改变 $(0,i-1)$ 范围内的东西，而对于右端点，则可以改变 $(0,l-1)$ 范围内的东西。

做完了，时间复杂度 $O(n\log n)$。

## Code
```c++
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e5 + 10;
struct segment_tree{
LL tre[N << 2],lzy[N << 2];
void pushup(int p){
	tre[p] = max(tre[p*2],tre[p*2+1]);
	return ;
}
void pushdown(int p){
	if(lzy[p] == 0) return ;
	tre[p*2] += lzy[p];
	tre[p*2+1] += lzy[p];
	lzy[p*2] += lzy[p];
	lzy[p*2+1] += lzy[p];
	lzy[p] = 0;
	return ;
}
void change(int x,int pl,int pr,int p,LL k){
	if(pl == pr){
		tre[p] = k;
		return ;
	}
	int mid = pl + pr >> 1;
	pushdown(p);
	if(x <= mid) change(x,pl,mid,p*2,k);
	else change(x,mid+1,pr,p*2+1,k);
	pushup(p);
}
void update(int L,int R,int pl,int pr,int p,LL k){
	if(L <= pl && pr <= R){
		tre[p] += k;
		lzy[p] += k;
		return;
	}
	pushdown(p);
	int mid = pl + pr >> 1;
	if(L <= mid)
		update(L,R,pl,mid,p*2,k);
	if(R > mid)
		update(L,R,mid+1,pr,p*2+1,k);
	pushup(p);
}
LL query(int L,int R,int pl,int pr,int p){
	if(L <= pl && pr <= R)
		return tre[p];
	int mid = pl + pr >> 1;
	pushdown(p);
	LL res = 0;
	if(L <= mid)
		res = max(res,query(L, R, pl, mid, p*2));
	if(R > mid)
		res = max(res,query(L,R,mid+1,pr,p*2+1));
	return res;
}
}sgt;
int n,m;
LL f[N],a[N];
vector< pair<int,LL> > vec[N];
int main(){
	IOS;
	File("race");
	cin >> n >> m;
	for(int i=1;i<=n;i++)
		cin >> a[i];
	for(int i=1;i<=m;i++){
		int l,r;
		LL val;
		cin >> l >> r >> val;
		vec[r].push_back({l,val});
	}
	for(int i=1;i<=n;i++){
		f[i] = f[i-1];
		sgt.update(0,i-1,0,n,1,-a[i]);
		for(pair<int,LL> x : vec[i])
			sgt.update(0,x.fi-1,0,n,1,x.se);
		f[i] = max(f[i],sgt.query(0,i-1,0,n,1));
		sgt.change(i,0,n,1,f[i]);
	}
	cout << f[n];
	return 0;
}
```

