#DP 
区间DP。难题。

[题目链接](https://www.luogu.com.cn/problem/P10197)。

## Trick
考虑拆 max。$\max(a,b) = \frac{1}{2} (a + b + |a-b|)$。因此，问题转化为求 $\sum\limits_{i=1}^{n-1} \frac{1}{2}(a_i+a_{i+1}+|a_{i}-a_{i+1}|)$ 的最小值。不难发现这等价与求后面部分的最小值。

## 性质
### (一)
实际上的问题如下：有 $n-k$ 个数字填入由 $k$ 个固定位置划分成的序列，要求上式最小。考虑将数字从小到大排序，那么对于每一段，一定是填入一段连续的数字是最优的。因此我们只用考虑最小值和最大值就好了。然后一段的贡献也就是 $|L - mn| + mx - mn + |R - mx|$，其中钦定 $L < R$。

这里需要讲下最左边的段和最右边的段怎么处理。考虑在最左边和最右边添加两个极大的固定点，其值为 $C$。真实的答案为 $\sum\limits_{i=1}^{n} a_i - \frac{1}{2}(a_1+a_n) + \sum\limits_{i=1}^{n-1}\frac{1}{2}|a_i - a_{i+1}|$。通过添加虚拟点，答案变成了$\sum\limits_{i=1}^{n} a_i + C + \sum\limits_{i=1}^{n-1}\frac{1}{2}|a_i - a_{i+1}| + \frac{1}{2}(C-a_1+C-a_n)$。注意到上下两个答案相差的刚好为 $2C$。因此并不需要分讨左右的情况。

### (二)
这是一个更强的性质，是本题的核心。

对于序列上的两个所选的区间，满足一定不会相交，也就是只会出现包含或相离的情况。

考虑使用调整法，假设我们选择了 $mn_i < mn_j < mx_i < mx_j$。那么我们就可以通过一系列的操作，使得 $mx_i < mn_j$，而这个操作缩小了 $mx_i$，扩大了 $mn_j$，可以证明这样不会让答案变得更大。

## DP
由于是区间问题，考虑区间 $DP$，观察到 $k$ 的数目不多，因此状态设计如下：

定义 $f[l][r][S]$ 表示通过选择序列(注：序列指的是可以交换的数字的集合)上 $[l,r]$ 的一段，使得集合 $S$ 内的区间得到解决的最小花费。

则有以下转移：

$f[l][r][S] = \min(f[l+1][r][S],f[l][r-1][S])$，表示留给包围的段进行选择。

$f[l][r][S] = \min(f[l][r][S],f[l][l+slen[T]-1][T]+f[l+slen[T]][R][S \verb|\| T]$。表示两段进行合并。这里不用考虑包的情况，因为在后面会先一步考虑。

$f[l][r][S]=\min(f[l][r][S],f[l+1][r-1][S\verb|\|{x}]+|R-mx|+mx-mn+|L-mn|)$，表示包含的情况。

## Code
```cpp
#include<bits/stdc++.h>
using namespace std;
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define LL long long
#define fi first
#define se second
const int N = 305,K = 10,M = (1 << 7) | 5;
int n,k;
LL a[N],b[N];
bool vis[N];
LL ans = 0;
int tot;
LL L[N],R[N],len[N],slen[M];
LL f[N][N][M];
vector<LL> vec;
int m;
int main(){
    IOS;
    cin >> n >> k;
    a[0] = a[n+1] = int(1e6);
    ans = int(2e6);
    for(int i=1;i<=n;i++){
        cin >> a[i];
        ans += a[i] * 2;
    }
    b[0] = 0,b[k+1] = n + 1;
    for(int i=1;i<=k;i++){
        cin >> b[i];
        vis[b[i]] = 1;
    }
    for(int i=1;i<=n;i++)
        if(!vis[i])
           vec.push_back(a[i]);
    vec.push_back(0);
    sort(vec.begin(),vec.end());
    for(int i=0;i<=k;i++){
        if(b[i] == b[i+1] - 1){
            ans +=  abs(a[b[i+1]] - a[b[i]]);
            continue;
        }
        L[++tot] = a[b[i]];
        R[tot] = a[b[i+1]];
        len[tot] = b[i+1] - b[i] - 1;
        if(L[tot] > R[tot]) swap(L[tot],R[tot]);
    }
    for(int i=1;i<(1<<tot);i++)
        for(int j=0;j<tot;j++)
            if((i >> j) & 1) slen[i] += len[j+1];
    m = vec.size();
    m -- ;
    for(int i=0;i<=m+1;i++)
        for(int j=0;j<=m+1;j++)
            for(int k=0;k<(1<<tot);k++)
                f[i][j][k] = ((k == 0) ? 0 : int(1e9));
    for(int i=1;i<=m;i++)
        for(int l=1;l<=m;l++){
            int r = l + i - 1;
            if(r > m) break;
            for(int S=1;S<(1<<tot);S++){
                f[l][r][S] = min(f[l+1][r][S],f[l][r-1][S]);
                if(slen[S] > i) continue;
                if(slen[S] == i){
                    for(int k=0;k<tot;k++){
                        if((S >> k) & 1){
                            if(i == 1) f[l][r][S] = min(f[l][r][S],abs(R[k+1] - vec[l]) + abs(L[k+1] - vec[l]));
                            else f[l][r][S] = min(f[l][r][S],f[l+1][r-1][S^(1<<k)] + abs(R[k+1] - vec[r]) + abs(L[k+1] - vec[l]) + vec[r] - vec[l]);
                        }
                    }
                }
                for(int T = (S - 1) & S;T;T = (T - 1) & S){
                    f[l][r][S] = min(f[l][r][S],f[l][l+slen[T]-1][T] + f[l+slen[T]][r][S^T]);
                }
            }
        }
    ans += f[1][m][(1<<tot)-1];
    ans /= 2;
    ans -= (LL)(2e6);
    cout << ans;
    return 0;
}
```
