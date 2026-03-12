# 序列 DP

序列 DP 就是在一维序列上进行的动态规划。其状态设计通常为 $f_{i,\cdot}$ 表示序列 $[1,i]$ 子问题的答案。

以下为常见的 DP 模型：

## 本质不同子序列计数

对于本质不同子序列，我们通常在所有值相同的子序列中钦定**最靠前**的那个来计数，其余的直接视为不合法的子序列。比如序列 $a=[1,2,2,3,3]$ 中，下标集合 $\{1,2,4\},\{1,2,5\},\{1,3,4\},\{1,3,5\}$ 都对应子序列 $[1,2,3]$，但只有下标集合 $\{1,2,4\}$ 会被统计，因为它是最靠前的。

考虑怎么判断一个子序列是不是同种子序列中最靠前的。设序列 $a$ 中一个子序列的下标集合为 ${b_1,b_2,\dots,b_m}$，再令 $b_0=0$，则这个子序列是最靠前的当且仅当 $\forall 1\le i\le m,b_{i-1}<j<b_i$，$a_j\neq a_{b_i}$，即对于子序列中的每个数 $x$ ，它与它左侧的那个数之间不能有等于 $x$ 的数，否则可以用那个数替换 $x$ 使得子序列更靠前。

???+ note "[P14230 不连续子串 / subseq](https://www.luogu.com.cn/problem/P14230)"

    给定长度为 $n$ 的序列 $a$，求 $a$ **非空本质不同**子序列的**非空本质不同**子序列的数量之和。$1\le a_i<n\le 8000$

    ??? note "题目解法"

        定义一些记号：

        - $C(b)$ 表示 $b$ 的本质不同子序列的数量（包含空序列，下同）；
        - $M(b)$ 表示 $b$ 的本质不同子序列的本质不同子序列的数量；
        - $S(b)$ 表示 $b$ 的所有本质不同子序列构成的集合
        - $c_i=C(a_{1\sim i})$，$m_i=M(a_{1\sim i})$，$s_i=S(a_{1\sim i})$；
        - $dp(b,x)$ 表示 $b$ 的所有本质不同子序列中，结尾的右侧还有等于 $x$ 的数的子序列数量。
        - $f(s,x)=\sum_{b\in S(s)}dp(b,x)$；
        - $end(b)$ 表示 $b$ 最后（最右）的元素；
        - $lst_i$ 表示 $a_i$ 上一次出现的位置。

        题目的答案为 $\sum_{b\in S(a)}(C(b)-1)=(\sum_{b\in S(a)}C(b))-C(a)=m_n-c_n$。

        考虑 $m,c$ 怎么转移，我们需要对每个 $a_i$ 求出以 $a_i$ 结尾的所有本质不同子序列的数量（记作 $rc_i$）和 $C$ 值之和（记作 $rm_i$），那么

        $$
        \begin{aligned}
        m_i&=m_{i-1}+rm_i\\
        c_i&=c_{i-1}+rc_i
        \end{aligned}
        $$
        
        考虑 $a_i$ 可以接在哪些子序列 $b$ 的后面。由上文提到的判断子序列是不是最靠前的方法，$[end(b),i-1]$ 之间不能有等于 $a_i$ 的数，即 $lst_i\le end(b)<i$。

        现在我们知道了子序列 $b$ 后面能接 $a_i$ 的条件，$rc_i$ 的计算就是简单的了：
        
        $$
        \begin{aligned}
        rc_i=&|\{b+a_i\mid lst_i\le end(b)<i,b\in s_{i-1}\}|\\
        =&|\{b\mid end(b)\le i-1,b\in s_{i-1}\}|-|\{b\mid end(b)\le lst_i-1,b\in s_{i-1}\}|\\
        =&|s_{i-1}|-|s_{lst_i-1}|\\
        =&c_{i-1}-c_{lst_i-1}
        \end{aligned}
        $$

        现在计算 $rm_i$。考虑如何计算 $C(b+a_i)$，我们分为两种：

        - 不包含 $a_i$：子序列数量为 $C(b)$
        - 包含 $a_i$：子序列数量为 $C(b)-dp(b,a_i)$，因为 $b$ 中的一些子序列的结尾和 $a_i$ 之间有等于 $a_i$ 的数，这些序列不能被统计。

        所以我们得到 $C$ 的递推式：

        $$C(b+a_i)=2C(b)-dp(b,a_i)$$

        然后得到：

        $$
        \begin{aligned}
        rm_i&=\sum_{b\in s_{i-1}\wedge lst_i\le end(b)<i}C(b+a_i)\\
        &=2\sum_{b\in s_{i-1}\wedge lst_i\le end(b)<i}C(b)-\sum_{b\in s_{i-1}\wedge lst_i\le end(b)<i}dp(b,a_i)\\
        &=2\left(\sum_{b\in s_{i-1}}C(b)-\sum_{b\in s_{lst_i-1}}C(b)\right)-\left(\sum_{b\in s_{i-1}}dp(b,a_i)-\sum_{b\in s_{lst_i-1}}dp(b,a_i)\right)\\
        &=2(m_{i-1}-m_{lst_i-1})-(f(i-1,a_i)-f(lst_i-1,a_i))\\
        \end{aligned}
        $$

        $rm_i$ 的计算依赖 $f$，所以我们再想一想 $f(i,x)$ 的转移。分为两类：

        - $x=a_i$：

        $$
        \begin{aligned}
        f(i,x)&=\sum_{b\in s_i}dp(b,x)\\
        &=\sum_{b\in s_{i-1}}dp(b,x)+\sum_{b\in s_i\wedge end(b)=i}dp(b,x)\\
        &=f(i-1,x)+\sum_{b\in s_{i-1}\wedge lst_i\le end(b)<i}C(b)\\
        &=f(i-1,x)+\sum_{b\in s_{i-1}}C(b)-\sum_{b\in s_{lst_i-1}}C(b)\\
        &=f(i-1,x)+m_{i-1}-m_{lst_i-1}
        \end{aligned}
        $$

        - $x\neq a_i$：

        $$
        \begin{aligned}
        f(i,x)&=\sum_{b\in s_i}dp(b,x)\\
        &=\sum_{b\in s_{i-1}}dp(b,x)+\sum_{b\in s_i\wedge end(b)=i}dp(b,x)\\
        &=f(i-1,x)+\sum_{b\in s_{i-1}\wedge lst_i\le end(b)<i}dp(b,x)\\
        &=f(i-1,x)+\sum_{b\in s_{i-1}}dp(b,x)-\sum_{b\in s_{lst_i-1}}dp(b,x)\\
        &=2f(i-1,x)-f(lst_i-1,x)
        \end{aligned}
        $$

        综上这道题就做完了。时空复杂度 $O(n^2)$。

    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 8005
        #define mod 1000000007
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
        int n,a[N];
        int lst[N],M[N],C[N],f[N][N];
        int main()
        {
            n=read();
            for(int i=1;i<=n;i++) a[i]=read();
            M[0]=C[0]=1;
            for(int i=1;i<=n;i++)
            {
                int x=a[i],L=lst[x];
                int rc=(C[i-1]-(L>0?C[L-1]:0)+mod)%mod;
                int rm=(2ll*(M[i-1]-(L>0?M[L-1]:0)+mod)%mod-(f[i-1][x]-(L>0?f[L-1][x]:0)+mod)%mod+mod)%mod;
                M[i]=(M[i-1]+rm)%mod;
                C[i]=(C[i-1]+rc)%mod;
                for(int v=1;v<=n;v++) if(v!=x) f[i][v]=(2ll*f[i-1][v]-(L>0?f[L-1][v]:0)+mod)%mod;
                f[i][x]=(f[i-1][x]+(M[i-1]-(L>0?M[L-1]:0)+mod)%mod)%mod;
                lst[x]=i;
            }
            printf("%d\n",(M[n]-C[n]+mod)%mod);
            return 0;
        }
        ```