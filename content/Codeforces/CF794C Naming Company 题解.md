---
tags:
  - 贪心
  - "#题解"
---
# 题目大意
A,B两人分别有$n$个小写字母组成的字符串。一开始有一个$n$个`?`组成的字符串。每次一个人将任意一个`?`替换为自己的一个字母，然后把该字母扔掉。两人轮流行动，A先，直到`?`全部被替换。A希望最后的字符串字典序尽可能小，B希望字典序尽量大。

假设A，B都以最优策略行动，最后的字符串是什么？
# 解题思路
## 一个错误的一眼思路
第一眼看到题，很容易想到，A选择最小的几个字符，B选择最大的几个字符，二人轮流往前放置即可

但按照这个思路，最终实现的代码会在第6个点WA掉

数据如下：

Input:

cbxz

aaaa

Ouput:

abac

而按照上述思路，输出的结果却是baca

## 调整思路
首先思考上述思路出现错误的原因，是因为A的最小字符已经超过了B的最大字符，导致了双方出现了一种类似于“互帮互助”的关系，也就相当于A在把大的往前放，B在把小的往前放

因此，在这种情况下，我们需要通过合适的选择，来减少自己对对方的帮助

具体的贪心如下：

对于A：当自身的字符都比B大的时候，选择自身最大的字符插到最后面(因为字典序前面的字符对结果的影响最大，因此需要将大的字符插到字符串的后面，更不利于对方)

对于B：当自身的字符都比A小的时候，选择自身最小的字符插到最后面
## 思路总结
最终的贪心很明显了

如果A中还有字符小于B的，那么把其中最小的字符往前放，否则就把选出的最大字符往后放，B反之，同理

>[!info] 证明
>假设不拿A中最小的放，而是拿次小的，显然不优
>
>而A如果直接不要最前面的位置，那么这个位置就会被B的最大占掉，更加的不优

至于A,B字符的选择：A肯定要选最小的n/2个，B肯定要选择自己最大的n/2个，不然也显然是不优的
# Code
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
#define IOS ios::sync_with_stdio(false);cin.tie(0),cout.tie(0)
#define File(s) freopen(s".in","r",stdin);freopen(s".out","w",stdout)
#define pii pair<int,int>
#define fi first
#define se second
namespace WinterXorSnow{
	const int N = 3e5 + 10;
	char a[N],b[N];
	char ans[N << 1];
	void work(){
		cin >> a >> b;
		int n = strlen(a);
		sort(a,a+n);
		sort(b,b+n,greater<char>());
		int ans_l,ans_r;
		ans_l = 1,ans_r = n;
		int cnt = 0;
		int a_l,a_r,b_l,b_r;
		a_l = b_l = 0,a_r = b_r = n/2 - 1;
		if(n & 1) a_r ++ ;
		while(cnt < n){
			cnt ++ ;
			if(a[a_l] < b[b_l]){//如果当前a中仍然还有比b要小的字符
				ans[ans_l] = a[a_l];
				ans_l ++ ;
				a_l ++ ;
			}
			else{//a当前的所有字符都大于b，要让b帮a打工
				ans[ans_r] = a[a_r];//把a中最大的放在后面，减少自己对b的帮助
				a_r -- ;
				ans_r -- ;
			}
			if(cnt >= n) break;
			cnt ++ ;
			if(b[b_l] > a[a_l]){
				ans[ans_l] = b[b_l];
				ans_l ++ ;
				b_l ++ ;
			}
			else{
				ans[ans_r] = b[b_r];
				b_r -- ;
				ans_r -- ;
			}
		}
		for(int i=1;i<=n;i++) cout << ans[i];
	}
}
int main()
{
	int T;
	T = 1;
	// cin >> T;
	while(T -- ) WinterXorSnow::work();
	return 0;
}
```