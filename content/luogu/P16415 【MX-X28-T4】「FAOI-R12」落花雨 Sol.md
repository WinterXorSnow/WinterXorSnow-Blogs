#DP 
> 在那耀眼的午后
>      追逐微风

[题目链接](https://www.luogu.com.cn/problem/P16415)

强强题目。

观察到 $T$ 中的每一个字符都是通过 $S$ 转化而得，因此问题变成判定性问题，即 $T$ 能否被 $S$ 转化而来。

处于贪心，假设对于一个串 $T$，我们肯定希望能通过最少的 $S$ 使其满足，给后面的位置更多的空间发挥。但是我们发现这个是错的。反例不难举出。但是根据打表的结果，我们可以发现只需要四个字符，我们就可以获得任意 `0/1`。这个性质很强，考虑从此入手。

因此，一开始的初始状态 $f_i$ 表示 $S$ 的前缀能生成多少种 $T$ 是倒闭的。

我们重新设计 DP 状态，新加两个 $x,y$，表示 $i+1$ 的位置能否形成 $T$ 以及 $i+2$ 的位置能否形成 $T$。因为到了 $i+3$ 的位置，我们在转移时，我们考虑的就是 $i+4$ 的位置的情况了，但是这显然可以通过 $i$ 位置得到。

然后就可以开始大力 DP 了。

做完了。

## Code

```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e5 + 10;
const LL mod = 998244353;
int n;
LL f[N][2][2];
string s;
string no0[3] = {"1","00","010"};
string no1[7] = {"0","01","10","11","000","101","111"};
LL ans = 0;
bool chk(int l,int r,int op){
	if(r > n) return 0;
	if(l > r) return 0;
	if(r - l + 1 >= 4) return 1;
	string t = s.substr(l,r-l+1);
	if(op == 1){
		for(int i=0;i<7;i++)
			if(t == no1[i]) return 0;
		return 1;
	}
	else{
		for(int i=0;i<3;i++)
			if(t == no0[i]) return 0;
		return 1;
	}
}
array<int,3> get(int st,int flag1,int flag2,int op){
	int x,y,z;
	x = -1,y = 0,z = 0;
	for(int i=st+1;i<=st+4;i++)
		if(chk(st+1,i,op) ||(flag1 && chk(st+2,i,op)) || (flag2 && chk(st+3,i,op))){
			x = i;
			break;
		}
	if(x == -1) return {x,y,z};
	if(chk(st+1,x+1,op) || (flag1 && chk(st+2,x+1,op)) || (flag2 && chk(st+3,x+1,op))) y = 1;
	if(chk(st+1,x+2,op) || (flag1 && chk(st+2,x+2,op)) || (flag2 && chk(st+3,x+2,op))) z = 1;
	return {x,y,z};
}
void solve(){
	for(int i=1;i<=n;i++)
		f[i][0][0] = f[i][0][1] = f[i][1][0] =  f[i][1][1] = 0;
	cin >> n;
	cin >> s;
	s = " " + s;
	ans = 0;
	f[0][0][0] = 1;
	for(int i=0;i<=n;i++)
		for(int j=0;j<2;j++)
			for(int k=0;k<2;k++){
				for(int o=0;o<2;o++){
					auto [x,y,z] = get(i,j,k,o);
					if(x <= n && x > i)
						f[x][y][z] = (f[x][y][z] + f[i][j][k]) % mod;
				}
				if((i == n) || (j && i == n-1) ||(k && i == n-2) || (i < n-2))
					ans = (ans + f[i][j][k]) % mod;
			}
	if(n <= 2) ans ++ ;
	cout << (ans - 1 + mod) % mod << "\n";
}
int main(){
	IOS;
	int T;
	cin >> T;
	while(T -- )
		solve();
	return 0;
}
```