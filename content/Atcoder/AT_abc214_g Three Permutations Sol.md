#DP #容斥原理
[题目链接](https://www.luogu.com.cn/problem/AT_abc214_g)

## 问题转化

考虑容斥，设 $f_i$ 表示 $i$ 个位置不合法的方案数。则答案为 $n! - \sum\limits_{i=1}^n {(-1)}^i \cdot f_i \cdot (n-i)!$。所以问题转化成如何快速求出 $f_i$。

不难发现对于每一个不合法的位置，我们都有取 $p_i,q_i$ 两种方案。但是如果直接选，肯定会出现两个位置取了一个值的情况。那么我们就可以对于每一个 $i$，将 $p_i$ 向 $q_i$ 连边。则对于每一个 $i$，我们都有三种选择，取 $p_i$ 或者取 $q_i$，或者都不取。

## DP
容易发现这样子建边会形成若干个环。让我们对单个环进行考虑。先把自环给判掉，这是容易的。而对于真正的环，我们考虑 DP。设置 $dp[i][j][k]$ 表示环上第 $i$ 个位置，选择 $j$ 个不合法的位置，不选/选了当前位置($p_i$)/选了下一位置($q_i$)，分别用 $k=0/1/2$ 进行表示。

我们可以枚举 $i=1$ 位置的状态，然后进行暴力的转移，最后再以类似卷积的方式累计到 $f_i$ 中，时间复杂度是 $O(n^2)$ 的。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 3050;
const LL mod = 1e9 + 7;
LL dp[N][N][3];
int p[N],q[N];
int fa[N],siz[N];
int n;
int find(int x){
	if(x == fa[x]) return x;
	return fa[x] = find(fa[x]);
}
void merge(int u,int v){
	u = find(u),v = find(v);
	if(u == v) return ;
	if(siz[u] < siz[v]) swap(u,v);
	fa[v] = u;
	siz[u] += siz[v];
}
LL f[N],g[N];
LL fac[N],inv[N];
LL qpow(LL a,LL b){
	if(b == 0)return 1;
	if(b == 1) return a;
	LL res = qpow(a, b/2);
	(res *= res) %= mod;
	if(b & 1ll) res *= a;
	return res % mod;
}
LL c(int a,int b){
	return fac[a] * inv[b] % mod * inv[a-b] % mod;
}
int main(){
	IOS;
	cin >> n;
	fac[0] = 1;
	for(int i=1;i<=n;i++)
		fac[i] = fac[i-1] * 1ll * i % mod;
	inv[n] = qpow(fac[n],mod-2);
	for(int i=n-1;i>=0;i--)
		inv[i] = 1ll * inv[i+1] * (i+1) % mod;
	for(int i=1;i<=n;i++)
		fa[i] = i,siz[i] = 1;
	for(int i=1;i<=n;i++)
		cin >> p[i];
	for(int i=1;i<=n;i++)
		cin >> q[i];
	for(int i=1;i<=n;i++)
		merge(p[i],q[i]);
	int total = 0;
	f[0] = 1;
	for(int pos=1;pos<=n;pos++){
		if(pos == find(pos)){
			int len = siz[pos];
			if(len == 1){
				for(int j=0;j<=total;j++){
					g[j] = f[j];
					f[j] = 0;
				}
				total += len;
				for(int j=0;j<=total;j++){
					f[j] = g[j] + (j > 0 ? g[j-1] : 0);
					f[j] %= mod;
				}
				continue;
			}
			for(int j=0;j<=total;j++)
				g[j] = f[j],f[j] = 0;
			total += len;
			for(int _=0;_<=2;_++){
				dp[1][0][0] = dp[1][1][1] = dp[1][1][2] = 0;
				dp[1][(_>=1)][_] = 1;
				for(int i=2;i<=len;i++){
					for(int j=0;j<=len;j++){
						dp[i][j][0] = dp[i][j][1] = dp[i][j][2] = 0;
						dp[i][j][0] = (dp[i-1][j][0] + dp[i-1][j][1] + dp[i-1][j][2]) % mod;
						if(j == 0) continue;
						dp[i][j][1] = (dp[i-1][j-1][0] + dp[i-1][j-1][1]) % mod;
						dp[i][j][2] = (dp[i-1][j-1][0] + dp[i-1][j-1][1] + dp[i-1][j-1][2]) % mod;
					}
				}
				for(int j=0;j<=len;j++){
					LL val =0 ;
					val += dp[len][j][0] + dp[len][j][1];
					if(_ != 1)
						val += dp[len][j][2];
					val %= mod;
					for(int k=0;k<=total;k++){
						f[k+j] += g[k] * val % mod;
						f[k+j] %= mod;
					}
				}
			}
		}
	}
	LL ans = 0;
	ans = fac[n];
	for(int i=1;i<=n;i++){
		LL fh;
		if(i & 1) fh = -1;
		else fh = 1;
		fh = (fh + mod) % mod;
		ans += fh * f[i] % mod * fac[n-i] % mod;
		ans = (ans + mod) % mod;
	}
	cout << ans;
	return 0;
}
```

## 后记
这里为了方便读者学习，详细给出 DP 的转移式子。

- $dp[i][j][0]=dp[i-1][j][0]+dp[i-1][j][1]+dp[i-1][j][2]$
- $dp[i][j][1]=dp[i-1][j-1][0]+dp[i-1][j][2]$
- $dp[i][j][2]=dp[i-1][j-1][0]+dp[i-1][j-1][1]+dp[i-1][j-1][2]$

然后需要注意的是，我们在完成环结尾的计算后，需要判断环结尾是否合法。也就是如果 `1` 号位选择的 $k=1$，则最后一个位置不能选择 $k=2$。