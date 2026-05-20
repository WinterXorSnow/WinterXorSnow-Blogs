[题目链接](https://www.luogu.com.cn/problem/CF1061F)

怎么那么喜欢放随机化的题目啊。

考虑随机选择叶子节点，判断其是否在直径上是简单的，可以 $O(n)$ 解决。

不难发现每次至少有一半的概率选到叶子，而在最坏情况下（k=2），也能在 `8` 次之内选到满足条件的叶子（只算概率的话）。

然后找根又是显然的。赛时因为询问方式看错喜提 `0` 分。

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