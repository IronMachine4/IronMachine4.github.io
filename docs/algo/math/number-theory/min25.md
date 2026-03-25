# Min_25 筛

Min_25 筛是在低于线性时间的复杂度内求出积性函数 $f$ 前缀和的算法。

## 记号约定 $\def\floor#1{\left\lfloor#1\right\rfloor}\def\bracket#1{\left(#1\right)}\def\minp{\operatorname{minp}}$

- 下文若无特殊说明，名为 $p$ 的变量均属于质数集。
- $\mathbb{P}$：质数集。
- $p_i$：从小到大第 $i$ 个质数。特别地，$p_0=1$。
- $\minp(i)$：$i$ 的最小质因子。

## 适用范围

1. $f$ 是一个积性函数（对于 $\gcd(a,b)=1$ 有 $f(ab)=f(a)f(b)$）；
2. $f(p^c)$ 可以被表示为少量完全积性函数之和（比如多项式）。

## 算法思路

### 第一部分

根据积性函数的性质，当我们求 $f(n)$ 的值时，我们可以把它的最小质因子的那一项拆出来，递归求其剩余的部分，即 $f(n)=f(\minp(n)^c)f\bracket{\floor{\frac{n}{\minp(n)^c}}}$（$c$ 为最小质因子的次数）。根据“适用范围”中的条件，$f(\minp(n)^c)$ 是容易求的。

令 $S(n,j)$ 表示 $2\sim n$ 中最小质因子大于 $p_j$ 的数的 $f$ 值之和，即：

$$
S(n,j)=\sum_{i=2}^{n}[\minp(i)>p_j]f(i)
$$

那么最终的答案为 $S(n,0)+f(1)$。

我们枚举这些 $i$ 的最小质因子以及其次数，有：

$$
S(n,j)=\sum_{\substack{k>j \\ p_{k}^{2}\le n}}\sum_{\substack{c\ge 1 \\ p_{k}^{c}\le n}}f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}+\sum_{\substack{k>j \\ p_k\le n}}f(p_k)
$$

???+ note "解释"

    首先，这个式子被分成了两个部分：以最后一步的加号为界，前半部分计算 $\minp>p_j$ 的合数的 $f$ 值，后半部分计算 $>p_j$ 的质数的 $f$ 值。

    -   对于前半部分，我们枚举合数的最小质因子 $p_k$ 和最小质因子的系数 $c$。由于是合数，我们要求 $p_k^2\le n$。
        
        对于所有最小质因子及次数为 $p_k^c$ 的合数 $i$，我们将 $f(i)$ 拆成 $f(p_k^c)f\bracket{\frac{i}{p_k^c}}$ 的形式（因为两者肯定互质），则：

        $$
        \begin{aligned}
        &\sum_{i}f(i)\\
        =&\sum_{i}f(p_k^c)f\bracket{\frac{i}{p_k^c}}\\
        =&f(p_k^c)\sum_{i}f\bracket{\frac{i}{p_k^c}}
        \end{aligned}
        $$

        由于 $i$ 的最小质因子为 $p_k$，所以 $\minp\bracket{\frac{i}{p_k^c}}>p_k$；同时 $\frac{i}{p_k^c}$ 的最大值为 $\frac{n}{p_k^c}$，最小值为 $[c>1]$（$c=1$ 的时候不能取到 $1$，否则 $i$ 为质数），所以：

        $$
        \sum_{i}f(i)=f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}
        $$

    -   对于后半部分，枚举范围内的质数，把 $f$ 加起来。

我们不妨设 $q(n)$ 表示 $\sum_{p\le n}f(p)$，则改写原式后半部分为：

$$
S(n,j)=\sum_{\substack{k>j \\ p_{k}^{2}\le n}}\sum_{\substack{c\ge 1 \\ p_{k}^{c}\le n}}f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}+q(n)-\sum_{i=1}^{j}f(p_i)
$$

现在，只要能求 $q$ 的值，就能求 $S$ 的值了。

### 第二部分

求 $q$ 的过程，我们使用埃筛的思想。

由于 $n$ 很大，我们不能直接对质数下手，而是考虑把 $[1,n]$ 中的合数都筛掉，剩下的就是质数的答案。我们发现埃筛的过程，是从小到大枚举每一个质数，筛掉这个质数的所有倍数。我们可以利用这种思路求 $q$。

但这里仍然有一个问题：$f$ 并不是完全积性函数，当 $f(n)$ 被 $\minp(n)$ 筛到时，$f(\minp(n))f\bracket{\frac{n}{\minp(n)}}\neq f(n)$。

注意到我们在这一部分的最终目的是求出所有质数的 $f$ 值之和，其在合数处的取值我们是不关心的。由“适用范围”得，$f$ 在质数的次方处可以拆成若干个完全积性函数，我们令完全积性函数 $f'$ 依次等于这些函数（即分成若干子问题），对 $f'$ 求其在质数处的取值之和，就可以组合成 $f$ 的 $q$ 值了。

比如：如果 $f(p^c)=p^{2c}+p^{c}$，现在求 $q(10)$。我们可以令 $f'(p^c)$ 先等于 $p^{2c}$，再等于 $p^{c}$，分别求出其在 $10$ 以内的质数的取值之和为 $2^2+3^2+5^2+7^2=87$ 和 $2+3+5+7=17$，则 $q(n)=87+17=104$。

我们令 $g(n,j)$ 表示 $[2,n]$ 的数在第 $j$ 轮埃筛后，剩下的数字的 $f'$ 值之和，则 $q(n)=g(n,|\mathbb{P}|)$，即：

$$
g(n,j)=\sum_{i=2}^{n}[i\in\mathbb{P}\operatorname{or}\minp(i)>p_j]f'(i)
$$

尝试从第 $j-1$ 次埃筛推出第 $j$ 次埃筛，我们有递推式：

$$
g(n,j)=
\left\{
\begin{aligned}
&g(n,j-1)&,p_{j}^{2}>n\\
&g(n,j-1)-f'(p_j)\bracket{g\bracket{\floor{\frac{n}{p_j}},j-1}-\sum_{i=1}^{j-1}f'(p_i)}&,p_{j}^{2}\le n
\end{aligned}
\right.
$$

初值显然为 $g(n,0)=\sum_{i=1}^{n}f'(i)$。

???+ note "解释"

    递推式的大致思路是从第 $j-1$ 轮埃筛 $g(n,j-1)$，减掉第 $j$ 轮埃筛涉及到的数（最小质因子为 $p_j$ 的合数）的 $f'$ 值，得到 $g(n,j)$ 的值。

    首先，显然地，当 $p_j^2>n$，没有最小质因子等于 $p_j$ 的合数，$g(n,j)=g(n,j-1)$。

    当存在 $\minp=p_j$ 的合数时，我们要减去这些合数的 $f'$ 值之和。模仿 $S$ 的求解过程，我们将这些合数的 $f'$ 值提取公因数 $f'(p_j)$，这些合数除以 $p_j$ 剩下部分的 $\minp$ 应当大于等于 $p_j$，即公式中的 $g\bracket{\floor{\frac{n}{p_j}},j-1}$，但是 $g\bracket{\floor{\frac{n}{p_j}},j-1}$ 中还包含了一些质数，我们要减掉 $\sum_{i=1}^{j-1}f'(p_i)$ 才行。

观察 $S$ 和 $g$ 的第一维：每次递归往下递归都是除以一个数下取整。根据 $\floor{\frac{\floor{\frac{n}{a}}}{b}}=\floor{\frac{n}{ab}}$，第一维我们一定可以表现为 $\floor{\frac{n}{t}}$ 的形式，这意味着第一维只有 $O(\sqrt{n})$ 种取值，内存问题就解决了。

## 时间复杂度

时间复杂度的证明居然需要积分，太难了，有兴趣自己去找找看吧。这里直接说结论：$O\bracket{\frac{n^{\frac{3}{4}}}{\log n}}$（一说 $O(n^{1-\epsilon})$）。

感性理解一下，两个函数的第一维都只有 $O(\sqrt{n})$，第二维和 $\sqrt{n}$ 以内质数大小有关，反正乘起来大概不会超过 $O(n)$。

## 参考代码

代码细节我放在注释了，代码难度相比数据结构都算小清新了。

???+ note "[P5325 【模板】Min_25 筛](https://www.luogu.com.cn/problem/P5325)"

    ```cpp
    #include<bits/stdc++.h>
    #define N 2000005
    #define mod 1000000007
    #define inv2 500000004
    #define inv6 166666668
    using namespace std;
    //由于 f(p^c)=p^{2c}-p^c，所以若无特殊解释，下文带数字 1 的数组计算 f'=p^c 时的情况，带数字 2 的数组计算 f'=p^{2c} 的情况
    long long n,sqr;
    bool p[N];
    long long tot,pri[N],s1[N],s2[N];
    void sieve()
    {
        //筛 sqrt(n) 以内质数
        for(int i=2;i<N;i++)
        {
            if(!p[i]) pri[++tot]=i;
            for(int j=1;j<=tot&&i*pri[j]<N;j++)
            {
                p[i*pri[j]]=true;
                if(i%pri[j]==0) break;
            }
        }
        
        //这是用于第二部分 g 的递推式和第一部分 f 的递推式中求 f' 在前若干个质数的取值之和
        for(int i=1;i<=tot;i++) s1[i]=(s1[i-1]+pri[i])%mod;
        for(int i=1;i<=tot;i++) s2[i]=(s2[i-1]+pri[i]*pri[i]%mod)%mod;
        return;
    }
    long long m,id1[N],id2[N],w[N],g1[N],g2[N];
    //这是用于计算 g(n,0) 的初值
    inline long long f1(long long x){x%=mod;return x*(x+1)%mod*inv2%mod;}
    inline long long f2(long long x){x%=mod;return x*(x+1)%mod*(2*x%mod+1)%mod*inv6%mod;}//sum_{i=1}^{n} i^2 = n(n+1)(2n+1)/6，可用整数裂项证明
    
    //这是用于 g 数组第一维的压空间
    //根据上文，g 的第一维只有 n/1 ~ n/n 等取值，不同的只有 sqrt(n) 种
    //我们给不同的 n/1 ~ n/n 编号，在整除分块下可做
    //但下文代码中显然从 n/1 ~ n/n 映射到编号更频繁
    //我们需要以 n/1 ~ n/n 为数组的下标
    //但有的 n/t 太大了
    //考虑到 min(t,n/t) <= sqrt(n)
    //我们记两个数组：id1 和 id2
    //如果 n/t 不是很大，直接作为 id1 的下标进行存储
    //否则，应存到 id2 的 n/(n/t) 的位置上
    //查询同理，getid 根据查询的 x 的大小判断返回哪个数组存储的编号
    inline long long getid(long long x)
    {
        if(x<=sqr) return id1[x];
        else return id2[n/x];
    }

    //S 的递归计算函数
    long long S(long long n,long long k)
    {
        //这显然吧
        if(pri[k]>n) return 0;

        //S 函数递推式的后半部分，记得在使用 g 数组时要 getid
        long long res=((g2[getid(n)]-g1[getid(n)]+mod)%mod-(s2[k]-s1[k]+mod)%mod+mod)%mod;

        //枚举 p_k,c
        for(int i=k+1;i<=tot&&pri[i]*pri[i]<=n;i++)
        {
            for(long long c=1,pw=pri[i];pw<=n;c++,pw*=pri[i])
            {
                //抄公式
                res=(res+pw%mod*(pw%mod-1)%mod*(S(n/pw,i)+(c>1))%mod)%mod;
            }
        }
        return res;
    }
    int main()
    {
        sieve();
        scanf("%lld",&n);
        sqr=(long long)sqrt(n);

        //整除分块
        for(long long l=1,r;l<=n;l=r+1)
        {
            r=n/(n/l);
            w[++m]=n/l;//小巧思：注意到 w 实际上是从大到小的
            //顺手计算 g(n,0) 的初值
            g1[m]=(f1(w[m])-1+mod)%mod;
            g2[m]=(f2(w[m])-1+mod)%mod;
            if(w[m]<=sqr) id1[w[m]]=m;
            else id2[n/w[m]]=m;
        }

        //g 的递推计算
        //注意到 g 的第二维在计算 S 的时候不重要，所以我们滚掉第二维
        //发现 g(n,j) 的计算只依赖 $g(n/p_k,j-1)$，所以我们应从大到小枚举 g 的第一维来保证滚动数组的正确性
        //恰好 w 也是从大到小的，所以顺序遍历 w 数组即可
        for(int j=1;j<=tot;j++)//先滚第二维
        {
            for(int i=1;i<=m&&pri[j]*pri[j]<=w[i];i++)//再顺序遍历 w
            {
                g1[i]=(g1[i]-pri[j]*(g1[getid(w[i]/pri[j])]-s1[j-1]+mod)%mod+mod)%mod;
                g2[i]=(g2[i]-pri[j]*pri[j]%mod*(g2[getid(w[i]/pri[j])]-s2[j-1]+mod)%mod+mod)%mod;
            }
        }

        printf("%lld\n",(S(n,0)+1)%mod);
        return 0;
    }
    ```