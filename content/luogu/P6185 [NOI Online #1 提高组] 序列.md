#题解 
# P6185 [NOI Online #1 提高组] 序列 Sol
一道比较有启发性的题。

[题目链接](https://www.luogu.com.cn/problem/P6185)

先想到构造，但发现根本没法写。但是考虑到这些操作连接了两个点，因此考虑使用图论的思路。

把每个位置建模成点，先考虑2操作，注意到这实际上就是在两个点之间分权值，因此实际上2操作就是将若干点形成若干个连通块，每个连通块的节点的总和不变，各点可以任意加减。

然后就可以把连通块缩成一个点了。

再考虑操作1,显然其不会改变奇偶性。并且如果各节点形成的是一个二分图，那么就会导致其左右两边的权值的差不变，否则，就是这依托图的节点总和的奇偶性不变。

然后完了。

## 小结
看到这种位置之间关系的“构造题”，不妨想下图论。
## Code
```
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 1e5 + 10;
int n,m;
int a[N],b[N];
int fa[N];
int col[N];
vector<int> G[N];
LL sum[N];
LL s[2];
int find(int x){
	if(x == fa[x]) return x;
	return fa[x] = find(fa[x]);
}
struct Ques{
	int t,u,v;
}ques[N];
void merge(int u,int v){
	u = find(u),v = find(v);
	if(u != v) fa[v] = u;
	return ;
}
bool dfs(int u,int nw_col){
	bool chk = 1;
	col[u] = nw_col;
	s[nw_col] += sum[u];
	for(int v : G[u]){
		if(col[v] == col[u]) chk = 0;
		if(col[v] == 0 && dfs(v,3-nw_col) == 0) chk = 0;
	}
	return chk;
}
void solve(){
	for(int i=1;i<=n;i++) G[i].clear();
	cin >> n >> m;
	for(int i=1;i<=n;i++)
		cin >> a[i];
	for(int i=1;i<=n;i++)
		cin >> b[i];
	for(int i=1;i<=n;i++){
		fa[i] = i;
		col[i] = sum[i] =0;
	}
	for(int i=1;i<=m;i++)
		cin >> ques[i].t >> ques[i].u >> ques[i].v;
	for(int i=1;i<=m;i++)
		if(ques[i].t != 1) merge(ques[i].u,ques[i].v);
	for(int i=1;i<=n;i++)
		sum[find(i)] += a[i] - b[i];
	for(int i=1;i<=m;i++)
		if(ques[i].t == 1){
			int u = ques[i].u,v = ques[i].v;
			u = find(u),v = find(v);
			G[u].push_back(v);
			G[v].push_back(u);
		}
	for(int i=1;i<=n;i++){
		if(col[i] == 0 && find(i) == i){
			s[1] = s[2] = 0;
			bool flag = dfs(i,1);
			if(flag == 1 && s[1] != s[2]){
				cout << "NO\n";
				return ;
			}
			if(flag == 0 && ((s[1] + s[2]) % 2 != 0)){
				cout << "NO\n";
				return ;
			}
		}
	}
	cout << "YES\n";
	return ;
}
int main()
{
	IOS;
	int T;
	cin >> T;
	while(T -- ) solve();
	return 0;
}
```
