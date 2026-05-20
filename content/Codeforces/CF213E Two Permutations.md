#哈希

[题目链接](https://www.luogu.com.cn/problem/CF213E)

赛后看了下，其实是一道糖糖题。

发现这题要求更为高效的匹配，考虑哈希。

观察到这道题哈希的维护有很多可以继承的地方，可以上数据结构进行维护。

具体的，肯定是要求一段值域连续的 b_i 的哈希值与 a+x 之后相等。因此使用线段树维护哈希。

做完了，$O(m \log m)$。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define LL long long
#define ull unsigned long long
#define fi first
#define se second
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
const int N = 2e5 + 10;
const ull base = 31;
ull pw[N],hsh;
int n,m;
int a[N],b[N];
int idx[N];
ull sum;
struct sgt{
	struct node{
		ull val;
		int len;
		node operator + (const node & b){
			node c;
			c.val = val * pw[b.len] + b.val;
			c.len = len + b.len;
			return c;
		}
	}tre[N<<2];
	void update(int x,int pl,int pr,int p,int k){
		if(pl == pr){
			if(k) tre[p].len = 1;
			else tre[p].len = 0;
			tre[p].val = k;
			return ;
		}
		int mid = pl + pr >> 1;
		if(x <= mid)
			update(x,pl,mid,p*2,k);
		else
			update(x,mid+1,pr,p*2+1,k);
		tre[p] = tre[p*2] + tre[p*2+1];
	}
}Hash;
int main(){
	IOS;
	File("seqmatch");
	cin >> n >> m;
	pw[0] = 1;
	for(int i=1;i<=n;i++){
		cin >> a[i];
		pw[i] = pw[i-1] * base;
		hsh = hsh * base + a[i];
		sum += pw[i-1];
	}
	for(int i=1;i<=m;i++){
		cin >> b[i];
		idx[b[i]] = i;
	}
	int ans = 0;
	for(int i=1;i<=m;i++){
		if(i > n)
			Hash.update(idx[i-n],1,m,1,0);
		Hash.update(idx[i],1,m,1,i);
		if(i >= n)
			if(Hash.tre[1].val == sum * (i-n) + hsh)
				ans ++ ;
	}
	cout << ans;
	return 0;
}
```