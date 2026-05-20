#珂朵莉树 #树状数组 
[题目链接](https://www.luogu.com.cn/problem/CF1638E)

使用 BIT + ODT 维护的模版题。

## 套路

不妨将每一个点的答案拆成 $a_x + val_{col}$，表示 $x$ 之前累计起来的答案加上当前节点在当前颜色所产生的答案。

这个东西看似不好维护，实则可以通过一个巧妙的变化完成：记 $tag_c$ 动态的维护当前颜色加上的值的总和。

考虑刻画一下一个点更换颜色的过程（假设节点 $u$ 的颜色从 $x$ 变成了 $y$）：

1. 加上 $tag_x$（下面的操作会说明这样为什么是正确的）
2. 减去 $tag_y$。

上面的东西显然是可以使用树状数组维护的。

然后在查询时，输出 $a_x + tag_{col}$。发现我们通过这样的操作，成功以一种类似于将前缀和拆开的方式，成功让 $x$ 能够正确的记录答案。

## 操作实现
回到本题，具体讲讲这道题怎么操作。

对于操作1，可以转化为 ODT 的区间推平，区间修改。

对于操作2，直接将对应 tag 增加。

对于操作3，按上面所讲方式输出即可。

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
const int N = 1e6 + 10;
LL tag[N];
int n,Q;
struct BIT{
	LL tre[N];
	int lowbit(int x){ return x & (-x);}
	void update(int x,LL k){
		while(x <= n){
			tre[x] += k;
			x += lowbit(x);
		}return ;
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
struct ODT{
	struct node{
		int l,r;
		mutable LL v;
		node(int L,int R=-1,int v=0):l(L),r(R),v(v){};
		bool operator < (const node &o) const{
			return l < o.l;
		}
	};
	set<node> st;
	IT split(int pos){
		IT it = st.lower_bound(node(pos));
		if(it != st.end() && it -> l == pos) return it;
		it -- ;
		int L,R;LL val;
		L = it -> l;
		R = it -> r;
		val = it -> v;
		st.erase(it);
		st.insert(node(L,pos-1,val));
		return st.insert(node(pos,R,val)).fi;
	}
	void assign(int l,int r,LL c){
		IT itr = split(r+1),itl = split(l);
		IT it = itl;
		for(;itl!=itr;itl++){
			int L = itl->l,R = itl->r,val = itl->v;
			tre.update(L,tag[val]);
			tre.update(R+1,-tag[val]);
		}
		tre.update(l,-tag[c]);
		tre.update(r+1,tag[c]);
		st.erase(it,itr);
		st.insert(node(l,r,c));
		return ;
	}
	int get_tag(int x){
		IT it = st.lower_bound(node(x));
		if(it != st.end() && it->l == x) return it -> v;
		it -- ;
		return it -> v;
	}
}odt;
int main(){
	IOS;
	cin >> n >> Q;
	odt.st.insert(ODT::node(1,n,1));
	while(Q -- ){
		string s;
		cin >> s;
		if(s == "Color"){
			int l,r,c;
			cin >> l >> r >> c;
			odt.assign(l, r, c);
		}
		else if(s == "Add"){
			int c;
			LL x;
			cin >> c >> x;
			tag[c] += x;
		}
		else{
			int x;
			cin >> x;
			cout << tre.query(x) + tag[odt.get_tag(x)] << "\n";
		}
	}
	return 0;
}
```

## 练习
[P9320 EGOI 2022 Tourists / 乌托邦旅行团](https://www.luogu.com.cn/problem/P9320)[[P9320 EGOI 2022 Tourists 乌托邦旅行团 Sol|Sol]]。