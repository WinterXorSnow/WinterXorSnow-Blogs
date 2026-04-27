#题解 
[题目链接](https://www.luogu.com.cn/problem/P11306)

## 关键性质

本道题有两个的常用结论：

1.**二叉搜索树的中序遍历一段单调不降的序列**（这里的中序遍历，指的是按中序遍历所访问的节点的值形成的序列），这是关于二叉搜索树的常见结论，不做解释

2.为了使得结论1真正派上用场，我们还需要发现，**对于在同一棵子树内的节点，这个子树内的中序遍历的访问顺序一定是一段连续的区间**，具体一点的讲，定义$dfn_i$表示$i$节点在中序遍历中是第几个被访问的，那么对于同一棵子树内的所有节点，其$dfn$必然连续，这个显然。

这道题还需要一个观察，对于一个节点$u$，从根节点到$u$节点的路径上，必然存在一个节点$x$，满足以$x$节点的父亲为根的子树不为BST，但是以$x$为根的子树是BST

## 快速判断

使用性质1，改写一下，就是一棵子树为BST，当且仅当在这棵子树的中序遍历所得到序列不存在任意一个位置$i$，满足$a_i > a_{i+1}$。那么我们就可以按整棵树中序遍历，将树变成一段序列，再进行判断。

然后在应用性质2，对于任意一棵子树，其包括自己在内的所有子结点在前面所提到的序列就必然是一段连续的区间，那么判断就变成了序列上的问题，也就是在序列的一段连续区间里，满足不存在$a_i > a_{i+1}$，这个可以使用BIT进行动态的维护

具体的，可以对$a_i > a_{i+1}$进行统计，如果成立，那么就在$i$位置+1统计，则判定$[l,r]$区间是否可行也就变成了判定$[l,r-1]$区间内的统计次数是否为0，时间复杂度$O(log n)$

## 计算

通过观察，不难想到可以倍增跳1~u的路径，然后判断就行了，时间复杂度$O(log^2 n)$

具体实现可以看代码

## Code

```c++
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 2e5 + 10;
int n,Q;
int a[N];
int ls[N],rs[N];
int idx = 0;
int dfn[N],val[N];//节点的dfs序，dfs序对应的值
int fa[N][20];
int L[N],R[N];
void dfs(int u){
	L[u] = idx+1;
	if(ls[u]) dfs(ls[u]);
	dfn[u] = ++idx;
	val[idx] = a[u];
	if(rs[u]) dfs(rs[u]);
	R[u] = idx;
	return ;
}
struct BIT{
	int Tre[N];
	BIT(){
		memset(Tre,0,sizeof Tre);
	}
	int lowbit(int x){
		return x & (-x);
	}
	void add(int x,int k){
		while(x <= n){
			Tre[x] += k;
			x += lowbit(x);
		}
		return ;
	}
	int query(int x){
		int sum = 0;
		while(x){
			sum += Tre[x];
			x -= lowbit(x);
		}
		return sum ;
	}
}tre;
bool check(int u){
	return (tre.query(R[u]-1) - tre.query(L[u]-1)) == 0 ? 1 : 0;
}
int calc(int u){
	if(!check(u)) return 0;
	int sum = 1;
	for(int j=18;j>=0;j--){
		if(fa[u][j] && check(fa[u][j])){
			sum += (1 << j);
			u = fa[u][j];
		}
	}
	return sum;
}
int main()
{
	IOS;
	cin >> n >> Q;
	for(int i=1;i<=n;i++){
		cin >> ls[i] >> rs[i];
		if(ls[i]) fa[ls[i]][0] = i;
		if(rs[i]) fa[rs[i]][0] = i;
	}
	for(int i=1;i<=n;i++)
		cin >> a[i];
	dfs(1);
	for(int j=1;j<=18;j++){
		for(int i=1;i<=n;i++) 
			fa[i][j] = fa[fa[i][j-1]][j-1];
	}
	for(int i=1;i<n;i++){
		if(val[i] > val[i+1]){
			tre.add(i,1);
		}
	}
	int ans = 0;
	for(int i=1;i<=n;i++)
		ans += int(check(i));
	while(Q -- ){
		int x,y;
		cin >> x >> y;
		int u = dfn[x];
		ans -= calc(x);
		if(val[u-1] > val[u]) tre.add(u-1,-1);
		if(val[u] > val[u+1]) tre.add(u,-1);
		val[u] = y;
		if(val[u-1] > val[u]) tre.add(u-1,1);
		if(val[u] > val[u+1]) tre.add(u,1);
		ans += calc(x);
		cout << ans << "\n";
	}
	return 0;
}
```