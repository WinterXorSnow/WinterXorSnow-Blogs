#构造 
[题目链接](https://www.luogu.com.cn/problem/AT_arc145_d)。

~~我说构造蓝=常规紫你耳朵隆吗？~~

先考虑如何构造不会产生等差数列。不妨思考一个美妙的数列，其任意两个数字相加都不会产生进位。而且，再利用一下题目中的已知信息 $x + z \ne 2y$。先在十进制内想想，如果每一位只有 0/1，不就不会出现进位了吗？推广到三进制，也是同样的道理。

实际上，正常情况这确实难想，尤其对于没见过的人。那么我们知道结论了，怎么构造？显然的，每一个位置加上 $d$，序列依然合法。那么假设我们构造出来了一个最基础的数列，则我们希望 $s,m$ 在 $\mod n$ 意义下同余。而序列长度又恰好为 $n$，因此我们不妨空出第 `0` 位，专门拿来调整，显然成立。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 1e4 + 10;
const int K = 14;
LL m,a[N];
int n;
int main(){
	IOS;
	cin >> n >> m;
	LL sum = 0;
	for(int i=1;i<=n;i++){
		for(int j=0,pw=3;j<=K;j++,pw*=3)
			if((i >> j) & 1)
				a[i] += 1ll * pw;
		sum += a[i];
	}
	LL cnt = ((sum - m) % n + n) % n;
	for(int i=1;i<=cnt;i++){
		a[i] -- ,sum -- ;
	}
	LL d = (sum - m) / n;
	for(int i=1;i<=n;i++)
		cout << a[i] - d << " ";
	return 0;
}
```

## 后记
我拿这道题去问 ds，ds 讲了一大通，发现自己的做法假了。蚌埠住了。