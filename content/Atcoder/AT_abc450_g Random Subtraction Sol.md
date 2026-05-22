#DP #期望
[题目链接](https://www.luogu.com.cn/problem/AT_abc450_g)

难难期望。

注意到 $x = \sum\limits_{i=1}^n(c_ia_i)$，则 $x^2=\sum\limits_{i=1}^n {a_i}^2+2 \times \sum\limits_{i=1}^n \sum\limits_{j=i+1}^n c_ic_ja_ia_j$。则期望显然为 $\mathbb{E}[x^2]=\sum\limits_{i=1}^n {a_i}^2+2 \times \sum\limits_{i=1}^n \sum\limits_{j=i+1}^n\mathbb{E}[c_ic_j]a_ia_j$。然后观察到 $c_i,c_j$ 的期望全部相等，不妨设这个为 $f$。

不难发现 $f$ 的值只与 $n$ 有关，因此设 $f_k$ 表示长度为的序列 $k$ 的上述期望。尝试从 $f_{k-1}$ 转移到 $f_k$。不妨假设 $A_1,A_2$ 进行合并，考虑 $c$ 的变化。但是如果直接考虑不太可行，考虑总和。显然两个位置贡献了一个 $-1$，其他位置不产生贡献。然后考虑 $[i+3,k]$ 范围内的贡献，显然是 $C_{k-2}^2 \cdot f_{k-1}$。最后再除上 $C_k^2$ 就好了。

## Code
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const LL mod = 998244353;
const int N = 2e5 + 10;
LL fac[N],inv[N];
LL f[N];
LL a[N];
int n;
LL qpow(LL a,LL b){
	if(b == 0) return 1;
	if(b == 1) return a;
	LL res = qpow(a, b/2);
	res = res * res % mod;
	if(b & 1ll) res *= a;
	return res % mod;
}
int main(){
	IOS;
	cin >> n;
	for(int i=1;i<=n;i++)
		cin >> a[i];
	fac[0] = 1;
	for(int i=1;i<=n;i++)
		fac[i] = fac[i-1] * i % mod;
	inv[n] = qpow(fac[n],mod-2);
	for(int i=n-1;i>=0;i--)
		inv[i] = inv[i+1] * (i+1) % mod;
	f[2] = (mod - 1);
	for(int i=3;i<=n;i++){
		f[i] = (mod-1) + 1ll * (i-2) * (i - 3) % mod * inv[2] % mod * f[i-1] % mod;
		f[i] = f[i] * inv[i] % mod * fac[i-2] % mod * fac[2] % mod;
	}
	LL ans = 0;
	for(int i=1;i<=n;i++)
		ans = (ans + a[i] * a[i] % mod) % mod;
	LL sum = 0;
	for(int i=1;i<=n;i++)
		sum = (sum + a[i]) % mod;
	sum = sum * sum % mod;
	for(int i=1;i<=n;i++)
		sum = (sum - a[i] * a[i] % mod + mod) % mod;
	sum = sum * f[n] % mod;
	cout << (ans + sum) % mod;
	return 0;
}
```