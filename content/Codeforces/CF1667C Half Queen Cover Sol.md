#构造 

[题目链接](https://www.luogu.com.cn/problem/CF1667C)。

假设答案为 $k$，则可以证明的是，我们有一种方式，能够在 $k\times k$ 的范围内完成答案。

那么考虑 $k$ 的下界。显然的，右下角会形成 $(n-k)\times (n-k)$ 的空间。这些空间只能使用对角线进行覆盖，而不同的对角线个数是 $2 \times (n-k)-1$ 的。因此有 $k \ge 2 \times n - 2\times k - 1$的，解不等式，可以得到 $k$ 的下界为 $\lfloor \frac{2n+1}{3} \rfloor$。

接下来将通过构造证明这个下界是可行的。这里先给出代码：

```cpp
int nw = 1;
for(int i=1;i<=k;i++){
	cout << i << " " << nw << "\n";
	nw += 2;
	if(nw  > k) nw = 2;
}
```

这个代码可以分成两个部分，第一部分的马走日，覆盖了右下角一半的对角线。然后通过计算，可以发现第二部分的马走日，也是正确的。读者可以通过画图解决。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 1e5 + 10;
int main(){
	IOS;
	int n;
	cin >> n;
	int k = (2 * n - 1) / 3;
	if((2 * n - 1) % 3) k ++ ;
	cout << k << "\n";
	int nw = 1;
	for(int i=1;i<=k;i++){
		cout << i << " " << nw << "\n";
		nw += 2;
		if(nw  > k) nw = 2;
	}
	return 0;
}
```