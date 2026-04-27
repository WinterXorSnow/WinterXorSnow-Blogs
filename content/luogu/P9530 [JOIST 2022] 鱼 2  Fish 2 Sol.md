#题解 
# P9530 [JOIST 2022]Fish 2 Sol

[题目链接](https://www.luogu.com.cn/problem/P9530)

## 关键性质

先开发一下性质，发现对于每条鱼 $i$ 都有一个存活区间 $[l_i,r_i]$ ，表示其能吃完这个区间内的所有鱼。显然所有鱼的存活区间只能包含或者无交，证明：假设存在两个存活区间 $[l_i,r_i]$，$[l_j,r_i]$ 满足 $l_i \le l_j \le r_i \le r_j$，那么阻挡 $i$ 进行扩展的鱼 $k$ 就存在于 $[l_j,r_j]$ 中，则又因为鱼 $j$ 没有被这条鱼阻挡，所以鱼 $j$ 在处理到 $k$ 时，其大小一定比 $i$ 的大小大（否则鱼 $i$ 就可以突破 $k$ 了），则 $l_j \le l_i$，矛盾了，故一定不会存在这种情况。

那么对于整个序列所有的存活区间 $[l_i,r_i]$，其包含任意一条鱼 $i$ 的区间个数必然不超过 $logA$个，其中 $A$ 表示序列 $a$ 的最大值。证明很简单：由于包含 $i$ 的所有存活区间一定互相包含，所以小区间的大小一定小于大区间的两倍。

考虑对于一段区间 $[L,R]$ ，这个区间内的鱼可能在后续存活当且仅当 $l_i\le L \le R \le r_i$，否则其一定无法扩张了。因此，对于一段区间，我们只需要记录以下三种区间：能够到 $L$ 的，能够到 $R$ 的，能够覆盖 $[L,R]$ 的，其他的一定不用考虑，而根据前面的结论，区间个数一定是 $logA$ 级别的，可以接受。

## 维护区间

令 $M = (L+R)/2$，则区间 $[L,R]$ 的信息可以从区间 $[L,M]$ 和区间 $[M+1,R]$ 转移而来。具体的，尝试合并已知存活区间 $[l,M]$ 和  $[M+1,r]$，显然，直接合并会超时，考虑优化。观察到小区间可以扩展到的，大区间一定能扩展到，所以可以使用双指针，从小的区间开始，尝试扩展，如果无法扩展了，再使用大区间进行，直到所有区间处理完。这样复杂度就是 $log A$ 的，可以接受。

时间复杂度 $O(n log n + q logn log A)$。

## Code

参考的 Alex_Wei 的题解

```c++
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int inf = 1e9 + 7;
const int N = 1e5 + 10;
struct node{
	int sum,cnt,out;//区间总重，鱼的数目，挡住扩张的鱼的大小
};
struct range{
	int l,r;//区间左右的值
	int sum;//区间的总和
	int cnt;//区间的可以覆盖整个区间的鱼的数目
	vector<node> L,R;//区间内只能够到L,R的存活区间
range operator + (const range &oth) const{//合并区间
	range x = {l,oth.r,min(inf,sum + oth.sum),0,L,oth.R};
	vector<node> fl = R,fr = oth.L;
	fl.push_back({sum,cnt,0});//加入能扩张完左区间的鱼，使之继续尝试扩展右区间
	fr.push_back({oth.sum,oth.cnt,0});
	int pl = 0,pr = -1;
	int tot = fl[0].cnt;
	int sum,bnd;//当前区间的和，可能挡住区间扩展的鱼的大小
	vector<node> tml, tmr;//暂时存储能到L,R的存活区间
	while(1){
		sum = min(inf,fl[pl].sum + (pr == -1 ? 0 : fr[pr].sum));
		bnd = (pr == -1 ? oth.l : fr[pr].out);//判断-1的原因是为了避免左区间连右区间都进不去
		if(sum >= bnd && pr < int(fr.size()) - 1) pr ++ ;
		else if(pl == fl.size() - 1) break;
		else{
			if(sum < fl[pl].out){//依旧无法向左扩展，结算
				if(pr == int(fr.size()) - 1)
					tmr.push_back({sum,tot,fl[pl].out});
				tot = 0;//能到R的，已经被加入，不能到R的，不被需要了，因此tot=0
			}
			tot += fl[++pl].cnt;
		}
	}
	if(pr == int(fr.size()) - 1) x.cnt += tot;
	else x.L.push_back({sum,tot,bnd});
	swap(fl,fr);
	pl = 0,pr = -1;
	tot = fl[0].cnt;
	while(1){
		sum = min(inf,fl[pl].sum + (pr == -1 ? 0 : fr[pr].sum));
		bnd = (pr == -1 ? r : fr[pr].out);
		if(sum >= bnd && pr < int(fr.size()) - 1) pr ++ ; 
		else if(pl == fl.size() - 1) break;
		else{
			if(sum < fl[pl].out){
				if(pr == int(fr.size()) - 1)
					tml.push_back({sum,tot,fl[pl].out});
				tot = 0;
			}
			tot += fl[++pl].cnt;
		}
	}
	if(pr == int(fr.size()) - 1) x.cnt += tot;
	else x.R.push_back({sum,tot,bnd});
	for(node it : tml) x.L.push_back(it);
	for(node it : tmr) x.R.push_back(it);
    /*
    这里简单给出tml，tmr的必要性的证明：考虑从左向右扩展，之所以要先进行else x.R.push_back({sum,tot,bnd});的加入，再进行tmr的加入，是因为前者表示的是从右区间来的鱼产生的存活区间，后者表示的是从左区间来的鱼产生的存活区间，显然右区间的鱼形成的存活区间的扩展能力是比从左区间来的鱼的扩展能力要弱的，因此需要先加入前者，再加入后者，也就需要使用tml，tmr
    */
	return x;
}
}tre[N << 2];
int n;
int a[N];
void build(int pl,int pr,int p){
	if(pl == pr){
		tre[p] = {a[pl],a[pl],a[pl],1};
		return ;
	}
	int mid = pl + pr >> 1;
	build(pl,mid,p*2);
	build(mid+1,pr,p*2+1);
	tre[p] = tre[p*2] + tre[p*2+1];
	return ;
}
void update(int pl,int pr,int p,int x){
	if(pl == pr){
		tre[p] = {a[pl],a[pl],a[pl],1};
		return ;
	}
	int mid = pl + pr >> 1;
	if(x <= mid)
		update(pl,mid,p*2,x);
	else update(mid+1,pr,p*2+1,x);
	tre[p] = tre[p*2] + tre[p*2+1];
	return ;
}
range query(int L,int R,int pl,int pr,int p){
	if(L <= pl && pr <= R) return tre[p];
	int mid = pl + pr >> 1;
	if(L <= mid && mid < R) return query(L,R,pl,mid,p*2) + query(L,R,mid+1,pr,p*2+1);
	if(L <= mid) return query(L,R,pl,mid,p*2);
	else return query(L,R,mid+1,pr,p*2+1);
}
void print(int pl,int pr,int p){
	cout << pl << " " << pr << " " << tre[p].l << " " << tre[p].r << " " << tre[p].sum <<" " <<  tre[p].cnt << "\n";
	if(pl == pr) return ;
	int mid = pl + pr >> 1;
	print(pl,mid,p*2);
	print(mid+1,pr,p*2+1);
	return ;
}
int main()
{
	IOS;
	cin >> n;
	for(int i=1;i<=n;i++)
		cin >> a[i];
	build(1,n,1);
	// print(1,n,1);
	int Q;
	cin >> Q;
	while(Q -- ){
		int op;
		cin >> op;
		if(op == 1){
			int x,y;
			cin >> x >> y;
			a[x] = y;
			update(1,n,1,x);
		}
		else{
			int l,r;
			cin >> l >> r;
			cout << query(l,r,1,n,1).cnt << "\n";
		}
	}
	return 0;
}
```