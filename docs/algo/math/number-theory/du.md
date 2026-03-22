# 杜教筛

杜教筛是一种在低于线性时间的复杂度内计算数论函数 $f$ 的前缀和的算法。

## 适用范围

如果对于数论函数 $f$，能找到数论函数 $g$，使得 $g$ 和 $f*g$ 的前缀和能快速求出，则 $f$ 适用于杜教筛。

## 算法思路

对于数论函数 $f$，定义其前缀和为 $S(n)=\sum_{i=1}^{n}f(i)$。则对于数论函数 $g$，有：

$$
\def\floor#1{\left\lfloor#1\right\rfloor}
\def\bracket#1{\left(#1\right)}
\begin{aligned}
&\sum_{i=1}^{n}(f*g)(i)\\
=&\sum_{i=1}^{n}\sum_{d\mid i}g(d)f\bracket{\frac{i}{d}}\\
=&\sum_{i=1}^{n}\sum_{j=1}^{\floor{\frac{n}{i}}}g(i)f(j)\\
=&\sum_{i=1}^{n}g(i)\sum_{j=1}^{\floor{\frac{n}{i}}}f(j)\\
=&\sum_{i=1}^{n}g(i)S\bracket{\floor{\frac{n}{i}}}\\
=&g(1)S(n)+\sum_{i=2}^{n}g(i)S\bracket{\floor{\frac{n}{i}}}
\end{aligned}\\
$$

即：

$$
S(n)=\frac{\sum_{i=1}^{n}(f*g)(i)-\sum_{i=2}^{n}g(i)S\bracket{\floor{\frac{n}{i}}}}{g(1)}
$$

这是杜教筛**最重要的等式**。

那么，对 $S(\floor{\frac{n}{i}})$ 数论分块，如果我们能够快速地求出 $f*g$ 和 $g$ 的前缀和，就能快速求出 $f$ 的前缀和了。

## 时间复杂度

首先，由 $\floor{\frac{\floor{\frac{n}{a}}}{b}}=\floor{\frac{n}{ab}}$ 得，对 $S(n)$ 有贡献的 $S(x)$ 的 $x$ 的取值为 $\floor{\frac{n}{1}},\floor{\frac{n}{2}},\dots,1$ 共 $O(\sqrt{n})$ 种，所以我们可以通过**记忆化**优化复杂度。

若 $g$ 和 $f*g$ 的前缀和可以被 $O(1)$ 计算，那么算法的复杂度为 $O(n^{\frac{3}{4}})$。

当然，如果 $f$ 是一个积性函数，我们还可以线性筛筛出前 $M$ 个 $f$ 的值。可以证明，当 $M=n^{\frac{2}{3}}$ 时算法的复杂度最小为 $O(n^{\frac{2}{3}})$。


## 参考代码

杜教筛的代码比较模版。

```cpp
//getf(n),getg(n),getfg(n) 分别对应 f,g,f*g 的前缀和
long long getf(long long n)
{
    if(n<=M) return pref[n];//线性筛预处理
    if(mp[n]) return mp[n];//记忆化
    long long res=getfg(n);
    for(long long l=2,r;l<=n;l=r+1)
    {
        r=n/(n/l);
        res-=(getg(r)-getg(l-1))*getf(n/l);
    }
    return res;//不除以 g(1) 是因为大部分数论函数在 1 的值为 1。
}
```

由于杜教筛有“找合适的 $g$”的步骤，所以一般用于求常见数论函数的前缀和。以下给出 $\mu$ 和 $\varphi$ 的前缀和求法：

=== "莫比乌斯函数"

    由 $\mu*I=\varepsilon$ 可得：

    ```cpp
    long long getS_mu(long long n)
    {
        if(n<=M) return pref[n];
        if(mp[n]) return mp[n];
        long long res=1;
        for(long long l=2,r;l<=n;l=r+1)
        {
            r=n/(n/l);
            res-=(r-l+1)*getS_mu(n/l);
        }
        return res;
    }
    ```

=== "欧拉函数"

    由 $\varphi*I=\text{id}$ 可得：

    ```cpp
    long long getS_phi(long long n)
    {
        if(n<=M) return pref[n];
        if(mp[n]) return mp[n];
        long long res=n*(n+1)/2;
        for(long long l=2,r;l<=n;l=r+1)
        {
            r=n/(n/l);
            res-=(r-l+1)*getS_phi(n/l);
        }
        return res;
    }
    ```

??? note "参考代码：[P4213 【模板】杜教筛](https://www.luogu.com.cn/problem/P4213)"

    ```cpp
    #include<bits/stdc++.h>
    #define N 1000005
    using namespace std;
    char buf[1<<23],*p1=buf,*p2=buf;
    #define getchar() (p1==p2&&(p2=(p1=buf)+fread(buf,1,1<<21,stdin),p1==p2)?EOF:*p1++)
    inline int read()
    {
        int x=0,f=1;
        char ch=getchar();
        while(!isdigit(ch))
        {
            if(ch=='-') f=-1;
            ch=getchar();
        }
        while(isdigit(ch)) x=x*10+(ch^48),ch=getchar();
        return x*f;
    }
    bool p[N];
    long long tot,pri[N],phi[N],mu[N];
    unordered_map<long long,long long> sphi,smu;
    void sieve()
    {
        mu[1]=1;
        phi[1]=1;
        for(int i=2;i<N;i++)
        {
            if(!p[i])
            {
                pri[++tot]=i;
                phi[i]=i-1;
                mu[i]=-1;
            }
            for(int j=1;j<=tot&&i*pri[j]<N;j++)
            {
                p[i*pri[j]]=true;
                if(i%pri[j]==0)
                {
                    mu[i*pri[j]]=0;
                    phi[i*pri[j]]=phi[i]*pri[j];
                    break;
                }
                mu[i*pri[j]]=-mu[i];
                phi[i*pri[j]]=phi[i]*phi[pri[j]];
            }
        }
        for(int i=1;i<N;i++) mu[i]+=mu[i-1],phi[i]+=phi[i-1];
        return;
    }
    long long getSPhi(long long n)//phi*I=id
    {
        if(n<N) return phi[n];
        if(sphi[n]) return sphi[n];
        long long res=n*(n+1)/2;
        for(long long l=2,r;l<=n;l=r+1)
        {
            r=n/(n/l);
            res-=(r-l+1)*getSPhi(n/l);
        }
        return sphi[n]=res;
    }
    long long getSMu(long long n)//mu*I=eps
    {
        if(n<N) return mu[n];
        if(smu[n]) return smu[n];
        long long res=1;
        for(long long l=2,r;l<=n;l=r+1)
        {
            r=n/(n/l);
            res-=(r-l+1)*getSMu(n/l);
        }
        return smu[n]=res;
    }
    void solve()
    {
        long long n=read();
        printf("%lld %lld\n",getSPhi(n),getSMu(n));
    }
    int main()
    {
        sieve();
        int t=read();
        while(t--) solve();
        return 0;
    }
    ```