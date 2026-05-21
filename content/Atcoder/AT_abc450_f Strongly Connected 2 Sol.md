#线段树 #DP 

[题目链接](https://www.luogu.com.cn/problem/AT_abc450_f)

到现在连这种题都做不出来是不是可以退役了。

不难发现本题的问题实际上为选出若干条边，使得可以从 `1` 走到 `n`。

开题的时候尝试了容斥，思考后发现难以解决。

实际上本题的正解是使用线段树优化 DP。记 $f_{i,j}$ 表示考虑到第 $i$ 条边，**最远**能够走到 $j$。要记录的是最远是显然的，走的更远一定更优。尝试转移。假设所选区间为 $[l,r]$，则 $[1,l-1]$ 不受影响，为 $f_{i,j} = f_{i-1,j} \times 2$，$[r+1,n]$ 也不受影响，转移同理。在 $[l,r-1]$ 区间中的点都可以到 $r$，而不会留在原地，所以有 $f_{i,j} = f_{i-1,j}$，而对于 $r$，则为 $f_{i,j} = \sum\limits_{j=l}^{r} f_{i-1,j}$。这个显然是能使用线段树维护的。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const LL mod = 998244353;
const int N = 2e5 + 10;
class Modint{
static constexpr LL mod = (LL)(998244353);
LL val;
public:
	Modint(long long v = 0) : val(v % mod){
		if(val < 0) val += mod;
	}
	long long value() const {return val;}
	Modint& operator = (long long v){
		val = (v % mod + mod) % mod;
		return *this;
	}

	Modint operator + (const Modint &o) const{
		return Modint(val + o.val);
	}
	Modint& operator += (const Modint &o){
		val = (val + o.val) % mod;
		return *this;
	}

	Modint operator - (const Modint &o) const{
		return Modint(val - o.val);
	}
	Modint& operator -= (const Modint &o){
		val = (val - o.val + mod) % mod;
		return *this;
	}
	Modint operator*(const Modint& o) const {
		return Modint(val * o.val);
    }
    Modint& operator*=(const Modint& o){
    	val = val * o.val % mod;
    	return *this;
    }
    static LL qpow(LL a,LL b = mod-2){
    	if(b == 0) return 1;
    	if(b == 1) return a;
    	LL res = qpow(a,b/2);
    	res = res * res % mod;
    	if(b & 1ll) res *= a;
    	return res % mod;
    }
    static LL inv(LL a){
    	return qpow(a);
    }
    Modint operator / (const Modint& o) const{
    	return Modint(val * inv(o.val));
    }
    Modint& operator /= (const Modint& o){
    	val = val * inv(o.val) % mod;
    	return *this;
    }
    bool operator == (const Modint &other) const{return val == other.val;}
    bool operator != (const Modint &other) const{return val != other.val;}
    friend ostream& operator <<(ostream& os ,const Modint &m){
    	return os << m.val;
    }
    friend istream& operator >>(istream& is,Modint &m){
    	long long v;
    	is >> v;
    	m = Modint(v);
    	return is;
    }
};

// <=== Modint Above ===>
struct segment_tree{
	Modint tre[N << 2],lzy[N << 2];
	void pushup(int p){tre[p] = tre[p*2] + tre[p*2+1];}
	void pushdown(int p){
		if(lzy[p] == 1) return ;
		tre[p*2]  *= lzy[p];tre[p*2+1] *= lzy[p];
		lzy[p*2] *= lzy[p];lzy[p*2+1] *= lzy[p];
		lzy[p] = 1;
		return ;
	}
	void add(int pl,int pr,int p,int x,Modint k){
		if(pl == pr){
			tre[p] += k;
			lzy[p] = 1;
			return ;
		}
		pushdown(p);
		int mid = pl + pr >> 1;
		if(x <= mid) add(pl,mid,p*2,x,k);
		if(x > mid) add(mid+1,pr,p*2+1,x,k);
		pushup(p);
	}
	void build(int pl,int pr,int p){
		tre[p] = 0,lzy[p] = 1;
		if(pl == pr){
			return ;
		}
		int mid = pl + pr >> 1;
		build(pl,mid,p*2);
		build(mid+1,pr,p*2+1);
		return ;
	}
	void mul(int L,int R,int pl,int pr,int p,LL k){
		if(L > R) return ;
		if(L <= pl && pr <= R){
			tre[p] *= k;
			lzy[p] *= k;
			return ;
		}
		pushdown(p);
		int mid = pl + pr >> 1;
		if(L <= mid) mul(L,R,pl,mid,p*2,k);
		if(R > mid) mul(L,R,mid+1,pr,p*2+1,k);
		pushup(p);
	}
	Modint query(int L,int R,int pl,int pr,int p){
		if(L > R) return 0;
		if(L <= pl && pr <= R) return tre[p];
		pushdown(p);
		int mid = pl + pr >> 1;
		Modint res = 0;
		if(L <= mid) res += query(L,R,pl,mid,p*2);
		if(R > mid) res += query(L,R,mid+1,pr,p*2+1);
		return res;
	}
}sgt;
int n,m;
pair<int,int> edge[N];
int main(){
	IOS;
	cin >> n >> m;
	for(int i=1;i<=m;i++)
		cin >> edge[i].fi >> edge[i].se;
	sort(edge+1,edge+1+m);
	sgt.build(1,n,1);
	sgt.add(1,n,1,1,1);
	for(int i=1;i<=m;i++){
		int l,r;
		l = edge[i].fi,r = edge[i].se;
		if(l > r){
			sgt.mul(1,n,1,n,1,2);
			continue;
		}
		sgt.mul(1,l-1,1,n,1,2);
		sgt.mul(r+1,n,1,n,1,2);
		sgt.add(1,n,1,r,sgt.query(l,r,1,n,1));
	}
	cout << sgt.query(n,n,1,n,1);
	return 0;
}
```