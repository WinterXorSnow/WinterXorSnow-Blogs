#组合数学  #题解 
> 组合意义天地灭

考虑gank绝对值。

由于 $a_i=b_i$ 时，没有贡献，因此不妨令 $a_i>b_i$，最后答案乘上 $2^m$ 就好了。

接下来是一个我能想到的做法，考虑把球的状态变成两部分：
- 紫球：蓝球+红球，贡献为0
- 红球：贡献为1

对于紫球，可以使用插板法解决，假定有 $i/2$ 个紫球，则方案数为$C_{i/2+m-1}^{m-1}$

但是对于剩下的 $k=n-i$ 个红球，没有想到。实际上，不妨想下 $C_n^1 = n$，也就是考虑把每个盒子中的红球中挑选出一个变为特殊球的方案数。可是这样子还是很难想，不妨令特殊球也变成隔板，那么其顺序是固定的特殊球/板子/特殊球/板子……，因此就变成了$C_{k+m-1}^{2m-1}$。

为什么是$k+m-1$？因为去除非特殊球后，剩下的普通球数量为 $k-m$，然后就是常规的插板法了。

因此答案为$\sum_{i=0}^{n} C_{i/2+m-1}^{m-1} \times C_{k+m-1}^{2m-1} \times 2^m \times [i\mod2==0]$。

预处理一下，时间复杂度为$O(n)$，做完了。
## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e7 + 10;
const LL mod = 998244353;
int n,m;
namespace my_math{
LL fac[N],inv[N];
LL pw[N];
LL qpow(LL a,LL b){
	if(b == 0) return 1;
	if(b == 1) return a;
	LL ans = qpow(a,b/2);
	ans *= ans;
	ans %= mod;
	if(b & 1ll ) ans *= a;
	return ans % mod;
}
void init(){
	fac[0] = 1;
	pw[0] = 1;
	for(int i=1;i<=N-10;i++){
		pw[i] = pw[i-1] * 2ll;
		fac[i] = fac[i-1] * 1ll * i;
		pw[i] %= mod;
		fac[i] %= mod;
	}
	inv[N-10] = qpow(fac[N-10],mod-2);
	for(int i=N-11;i>=0;i--){
		inv[i] = inv[i+1] * 1ll * (i+1);
		inv[i] %= mod;
	}
	return ;
}
LL C(int a,int b){
	return (((fac[a] * inv[b]) % mod) * inv[a-b]) % mod;
}
}
using namespace my_math;
int main()
{
	IOS;
	cin >> n >> m;
	init();
	LL ans = 0;
	for(int i=0;i<=n;i+=2){
		int k = n - i;
		if(k < m) break;
		ans += (C(i/2+m-1,m-1) * C(k+m-1,2*m-1)) % mod * pw[m];
		ans %= mod;
	}
	cout << ans;
	return 0;
}
```