## 前言
可以视作*状压DP*的升级，常常与**插头DP**联合使用。被巨佬 `Loop1st` 称为 `普及组算法`。

其通常出现在下面的情况中：
- 问题可以转化成网格图
- 点会受到周围点的约束并且约束比较强
- $n,m$ 中有一个或几个较小，可以接受
- 能用 DP

实际上学的是 [这篇博客](https://www.luogu.com.cn/article/f4gtskp9)。

## 例一、放置多米诺骨牌
[蒙德里安的梦想](https://www.luogu.com.cn/problem/P10975)。

**插头DP + 轮廓线DP 的经典题目**。

### 解法

如果考虑普通的状压，转移起来会变得非常痛苦，因此我们考虑不需要操作的算法——轮廓线DP。

```text

    -----
----*
```

当我们考虑 `*` 的时候，只用考虑上面，左边的位置就好了，至于下面，右边，都是后面的点需要考虑的。

但是这样子不好转移，因为我们尝试在状态上面记录一些辅助转移的东西。然后就该用插头DP。

```text

    -----
---R*
```

这样子我们这个位置就是固定的接在左边的位置上了。

同理，我们通过也能通过上面的状态推广到下面。

然后就做完了。

### 实现
轮廓线DP的实现有点与众不同，不妨对着代码讲。

```cpp
memset(f,0,sizeof f);
int nw = 0;
f[0][0] = 1;
for(int i=1;i<=h;i++){
	nw ^= 1;
	memset(f[nw],0,sizeof f[nw]);
	for(int j=0;j<(1<<w);j++)
		f[nw][j<<1] = f[nw^1][j];
	for(int j=1;j<=w;j++){
		nw ^= 1;
		memset(f[nw],0,sizeof f[nw]);
		for(int st=0;st<(1<<(w+1));st++){
			int x = get(st,j);
			int y = get(st,j-1);
			if(x == 1 && y == 1)
				continue;
			if(x == 1)
				f[nw][st-cal(j)] += f[nw^1][st];
			else if(y == 1)
				f[nw][st-cal(j-1)] += f[nw^1][st];
			else{
				f[nw][st+cal(j)] += f[nw^1][st];
				f[nw][st+cal(j-1)] += f[nw^1][st];
			}
		}
	}
}
cout << f[nw][0] << "\n";
```

初始化：`f[0][0] = 1;`，这是因为初始状态下，没有一个插头，自然是`1`。

```cpp
for(int j=0;j<(1<<w);j++)
	f[nw][j<<1] = f[nw^1][j];
```

这里的写法，与后面的内容有关系，是为了为向下的插头作准备。

```cpp
for(int st=0;st<(1<<(w+1));st++){
	int x = get(st,j);
	int y = get(st,j-1);
	if(x == 1 && y == 1)
		continue;
	if(x == 1)
		f[nw][st-cal(j)] += f[nw^1][st];
	else if(y == 1)
		f[nw][st-cal(j-1)] += f[nw^1][st];
	else{
		f[nw][st+cal(j)] += f[nw^1][st];
		f[nw][st+cal(j-1)] += f[nw^1][st];
	}
}
```

`get(st,x)`，是在取 `st` 的第 `x` 个二进制位。

为什么插头向下需要修改 `j-1` 的位置，这就是特别的实现之处了。通过这种写法，可以满足当前位置如果使用向下的插头，不会影响后面的列，只有向右的插头才会影响。

因此前面的位移的作用也就很明显了，是为了让 `j-1` 回到应该的 `j` 的位置。

从 `1` 开始的 `j` 同样也是独特的点，这个 `Deepseek` 给出了 `0-based` 的做法，但是实现的复杂程度更高，因此从 `1` 开始的更为通用。

> [!note] 
> 
> ```cpp
> #include<bits/stdc++.h>
> using namespace std;
> #define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
> #define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
> #define LL long long
> #define fi first
> #define se second
> const int N = (1 << 12) | 5;
> int h,w;
> LL f[2][N];
> int get(int x,int k){
> 	return (x >> k) & 1;
> }
> int cal(int k){
> 	return 1 << k;
> }
> int main(){
> 	IOS;
> 	while(cin >> h >> w){
> 		if(h == 0 && w == 0) break;
> 		if(h == 0 || w == 0){
> 			cout << 0 << "\n";
> 			continue;
> 		}
> 		memset(f,0,sizeof f);
> 		int nw = 0;
> 		f[0][0] = 1;
> 		for(int i=1;i<=h;i++){
> 			nw ^= 1;
> 			memset(f[nw],0,sizeof f[nw]);
> 			for(int j=0;j<(1<<w);j++)
> 				f[nw][j<<1] = f[nw^1][j];
> 			for(int j=1;j<=w;j++){
> 				nw ^= 1;
> 				memset(f[nw],0,sizeof f[nw]);
> 				for(int st=0;st<(1<<(w+1));st++){
> 					int x = get(st,j);
> 					int y = get(st,j-1);
> 					if(x == 1 && y == 1)
> 						continue;
> 					if(x == 1)
> 						f[nw][st-cal(j)] += f[nw^1][st];
> 					else if(y == 1)
> 						f[nw][st-cal(j-1)] += f[nw^1][st];
> 					else{
> 						f[nw][st+cal(j)] += f[nw^1][st];
> 						f[nw][st+cal(j-1)] += f[nw^1][st];
> 					}
> 				}
> 			}
> 		}
> 		cout << f[nw][0] << "\n";
> 
> 	}
> 	return 0;
> }
> ```

