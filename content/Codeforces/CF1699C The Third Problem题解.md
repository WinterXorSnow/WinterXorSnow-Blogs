---
tags:
  - 题解
  - 思维
---

[洛谷](https://www.luogu.com.cn/problem/CF1699C)
[Codeforces](https://codeforces.com/problemset/problem/1699/C)
# 解题思路
解决本题，需要惊人的注意力
## 关键观察

观察以下样例：
```text
6
1 2 4 0 5 3
```
我们不难发现：0的位置是定死的，1的位置也是定死的

而2却可以活动，但是活动范围又是有限的，观察一下样例解释，发现只能在0，1之间，思考原因

假设2在0，1之外，那么显然是无法移动的

而2在0，1之内，则也很容易理解，如果其来到了0，1之外，那么会干扰结果

由此我们得出，2必须要在0，1之内，否则无法移动
## 推至全体
根据2的情况，我们可以猜测，对于任意数字，都必须要在比它小的数的之间活动

具体的：

设$l,r$，分别表示比$x$小最左边的数的位置和最右边数的位置，那么根据上面的思路，如果$x$在$l,r$范围内，那么对答案就能有所贡献，为$r - l + 1 - x$，否则无贡献

# Code
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
#define int long long
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define pii pair<int,int>
#define fi first
#define se second
namespace WinterXorSnow{
	const int N = 1e5 + 10;
	const int mod = 1e9 + 7;
	int a[N];
	int pos[N];
	int n;
	void work(){
		cin >> n;
		for(int i=1;i<=n;i++){
			cin >> a[i];
			pos[a[i]] = i;
		}
		int l,r;
		l = pos[0],r = pos[0];
		int ans = 1;
		for(int i=1;i<n;i++){
			if(pos[i] < l)
				l = pos[i];
			else if(pos[i] > r)
				r = pos[i];
			else
				ans = ans * (r - l + 1 - i);
			ans %= mod;
		}
		cout << ans << "\n";
	}
}
signed main()
{
	int T;
	T = 1;
	cin >> T;
	while(T -- ) WinterXorSnow::work();
	return 0;
}
```
