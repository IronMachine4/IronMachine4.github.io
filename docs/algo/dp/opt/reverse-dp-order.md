# 翻转递推方向

## 算法思路

翻转递推方向常用于这样的 DP：

$$
f_{u}=\sum_{v}f_{v}\times w(u,v)
$$

其中 $w(u,v)$ 是只跟 $u$，$v$ 有关的函数，且对于一定的 $u$，对应的 $v$ 的集合一定。

此时把每种状态看作点，所有可能的 $(u,v)$ 看作边权 $w(u,v)$ 的边，就形成了一张 DAG，那么从 $f_{S}$ 到 $f_{T}$ 的 DP 就可以看作从点 $S$ 到点 $T$ 所有路径上边权的乘积和（即对于每条路径，将路径上的边权乘起来，最后再将每条路径加起来）。可以发现：如果交换 $f_{S}$ 和 $f_{T}$ 以及翻转所有有向边的方向，$f_{T}$（即 DP 的终点）的值仍然是不变的。

由此可以优化一些 DP：有的题目要求对每个 $1\le x\le n$ 都做一次 DP，但每次 DP 转移式都与 $x$ 无关，只有起点与 $x$ 有关导致多一个 $O(n)$ 的复杂度。这时可以考虑翻转递推方向，从终点往起点推，这样只用做一次 DP。

## 例题

### [P10013 [集训队互测 2023] Tree Topological Order Counting](https://www.luogu.com.cn/problem/P10013)

#### 题目大意

给定一颗外向树和数组 $b$。对于每个点 $u$，若在某个合法拓扑序中排名为 $i$，则它在该拓扑序中价值为 $b_{i}$。记 $w_{u}$ 为 $u$ 在所有合法拓扑序中的价值之和，求出所有的 $w_{u}$。$1\le n\le 5000$

#### 题目思路

既然要求每个点的答案，就枚举所有的 $1\le x\le n$ 求值。发现在树中只有 $x$ 的祖先的拓扑序与 $x$ 的拓扑序的大小关系是确定的：一定小于。所以考虑以 $x$ 为起点，在 $x$ 到根的链上 DP。记 $f_{u,i}$ 表示只考虑 $u$ 这颗子树，有 $i$ 个点拓扑序小于 $x$ 的拓扑序数量。有递推式：

$$
\def\sz{\operatorname{sz}}
f_{u,j+k}=\sum_{0\le j<\sz_{v}\wedge 1\le k\le\sz_{u}-\sz_{v}}f_{v,j}\binom{j+k-1}{j}\binom{\sz_{u}-j-k-1}{\sz_{v}-j-1}C_{u,v},u=\operatorname{fa}_v
$$

初值：$f_{x,0}$ 为 $x$ 子树的拓扑序数量

答案：$ans=\sum_{0\le i<n}f_{1,i}b_{i+1}$

解释：首先，在 $f_{v,j}$ 中有 $j$ 个拓扑序比 $x$ 小的有序节点，而 $f_{u,j+k}$ 中有 $(j+k)$ 个，去掉 $u$ 的拓扑序此时一定为最小后这 $j$ 个节点有 $(j+k-1)$ 个位置可以塞，即 $\binom{j+k-1}{j}$ 种插法，$\def\sz{\operatorname{sz}}\binom{\sz_{u}-j-k-1}{\sz_{v}-j-1}$ 同理。把子树 $v$ 插入子树 $u$ 后要给除 $v$ 外 $u$ 的子树排拓扑序，这里记作 $C_{u,v}$ 种。

如果直接这么做，相当于 $n$ 次树上背包，总复杂度 $O(n^3)$。但转移式和 $x$ 没有关系，这样做太过于浪费。考虑 DP 式可以看作 $f_{x,0}$ 到 $ans$ 的路径乘积和，所以可以翻转 DP 方向，从 $ans$（即初值为 $b_{i}$ 的所有 $f_{1,i}$）往 $f_{x,0}$ DP，最终 $x$ 的答案为 $f_{x,0}$ 乘上 $x$ 子树的拓扑序数量。这样就是对整棵树进行了一次树上背包，复杂度 $O(n^2)$。

接下来填上文的坑：记 $g_{u}$ 为 $u$ 子树的拓扑序数量，则 $g_{u}$ 可以这样由儿子得到：

$$
\def\sz{\operatorname{sz}}
g_{u}=(\sz_{u}-1)!\times\prod_{v\in \operatorname{son}_u}\frac{g_{v}}{\sz_{v}!}
$$

解释：首先，$u$ 一定为拓扑序第一位，其余位置 $(\operatorname{sz}_{u}-1)!$ 种排法；但这些剩余位置其实是 $u$ 的所有子树拓扑序并起来的，对于子树 $v$，$\operatorname{sz}_{v}!$ 种排列方式中只有 $g_{v}$ 种是合法的，所以要乘上一个 $\frac{g_{v}}{\operatorname{sz}_{v}!}$ 的系数。

那么，

$$
\def\sz{\operatorname{sz}}
\def\son{\operatorname{son}}
\begin{aligned}
C_{u,v}&=(\sz_u-\sz_v-1)!\times\prod_{x\in\son_{u}\wedge x\ne v}\frac{g_x}{\sz_{x}!}\\
&=\frac{(\sz_u-\sz_v-1)!\times\prod_{x\in\son_{u}\wedge x\ne v}\frac{g_x}{\sz_{x}!}\times(\sz_{u}-1)!\times\prod_{x\in \operatorname{son}_u}\frac{g_{x}}{\sz_{x}!}}{(\sz_{u}-1)!\times\prod_{x\in \operatorname{son}_u}\frac{g_{x}}{\sz_{x}!}}\\
&=\frac{(\sz_u-\sz_v-1)!g_u}{(\sz_u-1)!\frac{g_v}{\sz_v!}}\\
&=\frac{g_u(\sz_u-\sz_v-1)!\sz_v!}{g_v(\sz_{u}-1)!}
\end{aligned}
$$

#### 参考代码

```cpp
#include<bits/stdc++.h>
#define N 5005
#define mod 1000000007
using namespace std;
int n,now,head[N],to[N],nxt[N],b[N];
inline void add(int x,int y)
{
	to[++now]=y;
	nxt[now]=head[x];
	head[x]=now;
}
long long fac[N],ifac[N];
long long qpow(long long a,long long b)
{
	long long res=1;
	while(b)
	{
		if(b&1) res=res*a%mod;
		a=a*a%mod;
		b>>=1;
	}
	return res;
}
inline long long inv(long long x){return qpow(x,mod-2);}
void init()
{
	fac[0]=ifac[1]=1;
	for(int i=1;i<N;i++) fac[i]=fac[i-1]*i%mod;
	ifac[N-1]=inv(fac[N-1]);
	for(int i=N-2;i>=0;i--) ifac[i]=ifac[i+1]*(i+1)%mod;
}
inline long long C(long long n,long long m)
{
	if(n<0||m<0||n<m) return 0;
	return fac[n]*ifac[m]%mod*ifac[n-m]%mod;
}
int sz[N];
long long g[N],f[N][N];
void dfs1(int x)
{
	sz[x]=g[x]=1;
	for(int i=head[x];i;i=nxt[i])
	{
		int v=to[i];
		dfs1(v);
		sz[x]+=sz[v];
		g[x]=g[x]*g[v]%mod*ifac[sz[v]]%mod;
	}
	g[x]=g[x]*fac[sz[x]-1]%mod;
}
void dfs2(int x)
{
	for(int i=head[x];i;i=nxt[i])
	{
		int v=to[i];
		long long cnt=g[x]*fac[sz[v]]%mod*fac[sz[x]-sz[v]-1]%mod*inv(g[v]*fac[sz[x]-1]%mod)%mod;
		for(int j=0;j<sz[v];j++)
		{
			for(int k=1;k<=sz[x]-sz[v];k++)
			{
				f[v][j]=(f[v][j]+f[x][j+k]*C(j+k-1,j)%mod*C(sz[x]-j-k-1,sz[v]-j-1)%mod*cnt%mod)%mod;
			}
		}
		dfs2(v);
	}
}
int main()
{
	init();
	scanf("%d",&n);
	for(int i=2;i<=n;i++)
	{
		int x;
		scanf("%d",&x);
		add(x,i);
	}
	for(int i=0;i<n;i++) scanf("%d",&b[i]);
	for(int i=0;i<n;i++) f[1][i]=b[i];
	dfs1(1),dfs2(1);
	for(int i=1;i<=n;i++) printf("%lld ",f[i][0]*g[i]%mod);
	return 0;
}
```