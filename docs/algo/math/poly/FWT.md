# 快速沃尔什变换

快速沃尔什变换，是一种在 $O(n\log n)$ 的时间复杂度下解决位运算卷积的算法。

## 算法思路$\def\FWT{\operatorname{FWT}}$

首先，我们要先定义什么是位运算卷积。设一种位运算为 $\circ$，则多项式 $A$ 和 $B$ 的 $\circ$ 卷积 $C$ 被定义为：

$$
C_{k}=\sum_{i\circ j=k}A_i B_j
$$

参考 FFT/NTT 的思路。在做多项式的加法卷积时，我们先把多项式变成点值表示法，然后进行点乘，最后再从点值表示法逆变换成目标多项式。而在 FWT，我们可以对于多项式 $A$ 定义一个数组 $\FWT(A)$，使得：

1. $A$ 和 $\FWT(A)$ 一一对应；
2. 如果 $a$ 和 $b$ 位运算卷积得到 $c$，则 $\forall i,\FWT(C)_i=\FWT(A)_i\FWT(B)_i$。

对于第一条限制，我们设 $A_j$ 对 $\FWT(A)_i$ 的贡献系数为 $c(i,j)$，即：

$$
\FWT(A)_i=\sum_{j=0}^{n-1}c(i,j)A_j
$$

容易看出这是一个线性变换，其逆变换是简单的。

根据 $\FWT(C)_i=\FWT(A)_i\FWT(B)_i$，有：

$$
\begin{aligned}
\FWT(C)_i&=\FWT(A)_i\FWT(B)_i\\
\sum_{p=0}^{n-1}c(i,p)C_p&=\sum_{j=0}^{n-1}\sum_{k=0}^{n-1}c(i,j)c(i,k)A_jB_k
\end{aligned}
$$

又根据 $C_k=\sum_{i\circ j=k}A_iB_j$，有：

$$
\begin{aligned}
\sum_{p=0}^{n-1}c(i,p)\sum_{t_1\circ t_2=p}A_{t_1}B_{t_2}&=\sum_{j=0}^{n-1}\sum_{k=0}^{n-1}c(i,j)c(i,k)A_jB_k\\
\sum_{j=0}^{n-1}\sum_{k=0}^{n-1}c(i,j\circ k)A_jB_k&=\sum_{j=0}^{n-1}\sum_{k=0}^{n-1}c(i,j)c(i,k)A_jB_k
\end{aligned}
$$

对比左右两边可以得到：只要满足 $c(i,j\circ k)=c(i,j)c(i,k)$，就可以满足 $\FWT$ 数组的条件。

注意到位运算中每一位的计算具有独立性。因此，我们可以令 $c(i,j)$ 等于每一位的贡献系数的乘积。即令 $i_t$ 为 $i$ 二进制下的第 $t$ 位，则 $c(i,j)=\prod_{t}c(i_t,j_t)$。容易证明这是正确的。这样，我们就只用求出 $c(0,0),c(0,1),c(1,0),c(1,1)$ 的值了。

现在考虑 $\FWT$ 数组具体的计算。模仿 FFT/NTT 的分治思想，把区间内最高位为 $0$ 和 $1$ 的分为两边递归处理，即：（令 $i_0,i'$ 分别表示 $i$ 的最高位和去掉最高位剩下的部分）

$$
\begin{aligned}
\FWT(A)_i&=\sum_{j=0}^{n-1}c(i,j)A_j\\
&=\sum_{j=0}^{\frac{n}{2}-1}c(i,j)A_j+\sum_{j=\frac{n}{2}}^{n-1}c(i,j)A_j\\
&=\sum_{j=0}^{\frac{n}{2}-1}c(i_0,j_0)c(i',j')A_j+\sum_{j=\frac{n}{2}}^{n-1}c(i_0,j_0)c(i',j')A_j\\
&=c(i_0,0)\sum_{j=0}^{\frac{n}{2}-1}c(i',j')A_j+c(i_0,1)\sum_{j=\frac{n}{2}}^{n-1}c(i',j')A_j\\
\end{aligned}
$$

可以看出，$\sum_{j=0}^{\frac{n}{2}-1}c(i',j')A_j$ 和 $\sum_{j=\frac{n}{2}}^{n-1}c(i',j')A_j$ 可以看做 $A$ 从中间分成左右两边，分别做长度为 $\frac{n}{2}$ 的 $\FWT$，记 $A$ 左半部分为 $A_0$，右半部分为 $A_1$，则：

$$
\begin{aligned}
\FWT(A)_i&=c(0,0)\FWT(A_0)_i+c(0,1)\FWT(A_1)_i\\
\FWT(A)_{i+\frac{n}{2}}&=c(1,0)\FWT(A_0)_i+c(1,1)\FWT(A_1)_i\\
\end{aligned}
$$

其中 $i\in [1,\frac{n}{2})$。

这和 FFT/NTT 中的分治几乎一模一样，复杂度 $O(n\log n)$。

至于逆变换，我们上文提到过，FWT 的本质是线性变换。因此，把矩阵 $\left[\begin{smallmatrix}c(0,0)&c(0,1)\\ c(1,0)&c(1,1)\end{smallmatrix}\right]$ 求一个逆，就是 IFWT 的转移系数了。

接下来，我们给出一些常见位运算的 FWT 的转移系数矩阵：

- 按位 or：$\left[\begin{smallmatrix}1&0 \\ 1&1\end{smallmatrix}\right]$，逆矩阵 $\left[\begin{smallmatrix}1&0\\ -1&0\end{smallmatrix}\right]$；
- 按位 and：$\left[\begin{smallmatrix}1&1 \\ 0&1\end{smallmatrix}\right]$，逆矩阵 $\left[\begin{smallmatrix}1&-1\\ 0&1\end{smallmatrix}\right]$；
- 按位 xor：$\left[\begin{smallmatrix}1&1 \\ 1&-1\end{smallmatrix}\right]$，逆矩阵 $\left[\begin{smallmatrix}0.5&0.5\\ 0.5&-0.5\end{smallmatrix}\right]$。

## 参考代码

???+ note "[P4717 【模板】快速莫比乌斯 / 沃尔什变换 (FMT / FWT)](https://www.luogu.com.cn/problem/P4717)"

    ```cpp
    #include<bits/stdc++.h>
    #define N 131075
    #define mod 998244353
    #define inv2 499122177
    using namespace std;
    char buf[1<<23],*p1=buf,*p2=buf;
    #define getchar() (p1==p2&&(p2=(p1=buf)+fread(buf,1,1<<21,stdin),p1==p2)?EOF:*p1++)
    inline long long read()
    {
        long long x=0,f=1;
        char ch=getchar();
        while(!isdigit(ch))
        {
            if(ch=='-') f=-1;
            ch=getchar();
        }
        while(isdigit(ch)) x=x*10+(ch^48),ch=getchar();
        return x*f;
    }
    long long n,a[N],b[N],f[N],g[N];
    long long cor[2][2]={{1,0},{1,1}};
    long long icor[2][2]={{1,0},{mod-1,1}};
    long long cand[2][2]={{1,1},{0,1}};
    long long icand[2][2]={{1,mod-1},{0,1}};
    long long cxor[2][2]={{1,1},{1,mod-1}};
    long long icxor[2][2]={{inv2,inv2},{inv2,mod-inv2}};
    void fwt(long long* f,long long c[2][2])
    {
        for(int len=2;len<=(1<<n);len<<=1)
        {
            for(int i=0;i<(1<<n);i+=len)
            {
                for(int j=i;j<i+len/2;j++)
                {
                    long long x=f[j],y=f[j+len/2];
                    f[j]=(c[0][0]*x%mod+c[0][1]*y%mod)%mod;
                    f[j+len/2]=(c[1][0]*x%mod+c[1][1]*y%mod)%mod;
                }
            }
        }
    }
    void bitmul(long long* f,long long* g,long long c[2][2],long long ic[2][2])
    {
        fwt(f,c);
        fwt(g,c);
        for(int i=0;i<(1<<n);i++) f[i]=f[i]*g[i]%mod;
        fwt(f,ic);
    }
    int main()
    {
        n=read();
        for(int i=0;i<(1<<n);i++) a[i]=read()%mod;
        for(int i=0;i<(1<<n);i++) b[i]=read()%mod;
        memcpy(f,a,(1<<n)*sizeof(long long));
        memcpy(g,b,(1<<n)*sizeof(long long));
        bitmul(f,g,cor,icor);
        for(int i=0;i<(1<<n);i++) printf("%lld ",f[i]);
        putchar('\n');
        memcpy(f,a,(1<<n)*sizeof(long long));
        memcpy(g,b,(1<<n)*sizeof(long long));
        bitmul(f,g,cand,icand);
        for(int i=0;i<(1<<n);i++) printf("%lld ",f[i]);
        putchar('\n');
        memcpy(f,a,(1<<n)*sizeof(long long));
        memcpy(g,b,(1<<n)*sizeof(long long));
        bitmul(f,g,cxor,icxor);
        for(int i=0;i<(1<<n);i++) printf("%lld ",f[i]);
        putchar('\n');
        
        return 0;
    }
    ```