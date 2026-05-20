#根号分治 #ST表 
[题目链接](https://www.luogu.com.cn/problem/CF2206E)

~~疑似遇到天敌了。~~

注：以下下标均从 `0` 开始。

## 转化

这题一开始的转化是显然的，我们可以对于任意一个数字，转化成 $a_j + w_i(j \equiv i \pmod m)$。因此，我们可以记录 $y_j = \max\limits_{i=l,j \equiv i \pmod m}^r w_i$。然后假设答案为 $x$，则应当满足对于任意的 $j$，有 $a_j + y_j \le x$，即 $a_j \le x - y_j$，又因为 $\sum\limits_{j=0}^{m-1} a_j = s_0$，所以有 $m\cdot x - \sum\limits_{j=0}^{m-1} y_j \ge s_0$，也就得到了 $x$ 的取值范围。

问题变成如何快速获取区间内的 $y$。当 $r-l+1<m$ 时是显然的，答案为 `unbounded`。

## m 较小时

当 $m$ 较小时，我们可以使用 ST表 进行维护。具体的，对于每一个 $m$，建一个 ST表 进行维护。这样的复杂度是 $O(qm)$ 的，可以接受。

## m较大时

对于 $m$ 较大时，维护就稍微复杂点了。在这种情况下，我们的项目种类很多但是每一项的数目很少。此时，不妨看看具体的情况。

假设 $n=25,m=5$，显然如下：

```text
ooooo
ooooo
ooooo
ooooo
ooooo
```

我们假设需要查询下面这一段：

```text
ooxxx
xxxxx
xxxxx
xxxxx
xxxoo
```

那么为了快速计算，我们可以这样拆分:

```text
oo211
33211
33211
33211
332oo
```

而对于这三块，我们是可以通过预处理，以 $O(1)$ 的复杂度得到这三块的答案。

具体的，在预处理时，由于 $m$ 较大，因此我们至多有 $n/m$ 的长度为 $m$ 的块。因此我们可以记录从 $u$ 到 $v$ 的块的 $y$ 的前缀和。具体的，套一个类似于区间DP的东西就好了。

接下来就是分讨了。有两种情况。

```text
oooxx
xxxxx
xxooo
```

和

```text
oxxxx
xxxxx
xxxxo
```

手玩一下就好了。

## Code
这里我的第二部分发现空间复杂度是 $O(n\sqrt n)$。因此需要当 $m \ge 1000$ 时才能进第二部分，否则可能 MLE。
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const LL INF = 1e15;
void solve1(int n,int m){
	vector<LL> s(n);
	for(int i=0;i<n-m+1;i++)
		cin >> s[i];
	vector<LL> w(n+5);
	for(int i=0;i<m;i++)
		w[i] = 0;
	for(int i=0;i<n-m;i++)
		w[i+m] = w[i] + s[i+1] - s[i];
	vector<int> lg(n+1);
	lg[1] = 0;
	for(int i=2;i<=n;i++)
		lg[i] = lg[i>>1] + 1;
	vector< vector< vector<LL> > > ST(m+5,vector<vector<LL>> (n/m+5,vector<LL>(20,-INF)));
	for(int i=0;i<n;i++)
		ST[i%m][i/m+1][0] = w[i];
	for(int k=0;k<m;k++){
		int siz = (n-1) / m + 1;
		for(int j=1;j<=lg[siz];j++)
			for(int i=1;i<=siz-(1<<j)+1;i++)
				ST[k][i][j] = max(ST[k][i][j-1],ST[k][i+(1<<j-1)][j-1]);
	}
	auto query = [&](int l,int r,int k) -> LL{
		int siz = (n-1) / m + 1;
		int x = lg[r-l+1];
		return max(ST[k][l][x],ST[k][r-(1<<x)+1][x]);
	};
	int Q;
	cin >> Q;
	while(Q -- ){
		int l,r;
		cin >> l >> r;
		LL sum = 0;
		l -- ;
		r -- ;
		if(r - l + 1 < m){
			cout << "unbounded\n";
			continue;
		}
		for(int i=l;i<l+m;i++){
			int k = i % m;
			int R = (r - k) / m + 1;
			sum += query(i/m+1,R,i%m);
		}
		sum += s[0];
		if(sum > 0){
			if(sum % m)
				cout << sum / m + 1 << "\n";
			else
				cout << sum / m << "\n";
		}
		else
			cout << sum / m << "\n";
	}
}
void solve2(int n,int m){	
	vector<LL> s(n);
	for(int i=0;i<n-m+1;i++)
		cin >> s[i];
	vector<LL> w(n+5);
	for(int i=0;i<m;i++)
		w[i] = 0;
	for(int i=0;i<n-m;i++)
		w[i+m] = w[i] + s[i+1] - s[i];
	int siz = n / m + 1;
	vector< vector< vector<LL> > > f(siz+5,vector< vector<LL> >(siz+5,vector<LL>(m+5,-INF)));
	for(int i=0;i<n;i++)
		f[i/m+1][i/m+1][i%m]  = w[i];
	for(int k=0;k<m;k++){
		int L = 1,R = (n - k) / m + 1;
		for(int l=2;l<=R;l++){
			for(int i=1;i<=R;i++){
				int j = i + l - 1;
				if(j > R) break;
				f[i][j][k] = max(f[i][j-1][k],f[j][j][k]);
			}
		}
	}
	for(int l=1;l<=(n-1)/m+1;l++)
		for(int r=l;r<=((n-1)/m+1);r++){
			for(int i=1;i<m;i++)
				f[l][r][i] += f[l][r][i-1];
		}
	auto query = [=](int u,int v,int l,int r) -> LL{
		if(u > v) return 0;
		if(l > r) return 0;
		return f[u][v][r] - ((l == 0) ? 0 : f[u][v][l-1]);
	};
	int Q;
	cin >> Q;
	while(Q -- ){
		int l,r;
		cin >> l >> r;
		if(r - l + 1 < m){
			cout << "unbounded\n";
			continue;
		}
		l -- ;
		r -- ;
		int q,p,a,b;
		q = l / m + 1,a = l % m,p = r / m + 1,b = r % m;
		LL sum = 0;
		if(a <= b)
			sum += query(q,p,a,b) + query(q+1,p,0,a-1) + query(q,p-1,b+1,m-1);
		else
			sum += query(q+1,p-1,b+1,a-1) + query(q+1,p,0,b) + query(q,p-1,a,m-1);
		sum += s[0];
		if(sum > 0){
			if(sum % m)
				cout << sum / m + 1 << "\n";
			else
				cout << sum / m << "\n";
		}
		else
			cout << sum / m << "\n";
	}
}
int main(){
	IOS;
	// File("sum");
	int n,m;
	cin >> n >> m;
	if(m <= 1000)
		solve1(n,m);
	else
		solve2(n,m);
	return 0;
}
```

做完了，不想再见。