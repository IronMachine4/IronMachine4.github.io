# 快速数论变换

**快速数论变换**是一种在 $O(n\log n)$ 的时间复杂度内求出两个多项式在**模意义**下的卷积。

## 模意义下的单位根

复数域的单位根在计算机中需要使用浮点数来存储，有比较严重的精度问题。这个时候就需要用模意义下的原根（是正整数）代替单位根。

令模 $p$ 意义下的原根为 $g$（$p$ 为质数），则模 $p$ 意义下的 $n$ 次单位根 $g_{n}=g^{\frac{p-1}{n}}$。

???+ abstract "原根"

    如果 $g$ 是模 $p$ 意义下的原根，其满足以下条件：

    1. $g^{\varphi(p)}\equiv 1\pmod{p}$；
    2. $g^0,g^1,g^{2},\dots,g^{\varphi(p)-1}$ 模 $p$ 不同余。

    通常情况下 $p$ 为质数，此时 $\varphi(p)=p-1$。可以证明质数下的原根一定存在。

    原根的求解可以暴力从小往大枚举或者随机，期望复杂度很小。

为什么称 $g_n=g^{\frac{p-1}{n}}$ 是模 $p$ 意义下的 $n$ 次单位根？我们可以分析它与普通单位根有哪些相同的性质。

1.  单位根的定义：
    - $\omega_{n}^{n}=1$
    - $g_{n}^n\equiv g^{\frac{p-1}{n}\cdot n}\equiv g^{p-1}\equiv 1\pmod{p}$
2.  单位根的性质
    1. $\omega_{rn}^{rk}=\omega_{n}^{k}$，$g_{rn}^{rk}\equiv g^{\frac{p-1}{rn}\cdot rk}\equiv g^{\frac{p-1}{n}\cdot k}\equiv g_{n}^{k}\pmod{p}$；
    2. $\omega_{n}^{k+n}=\omega_{n}^{k}$，$g_{n}^{k+n}=g_{n}^{k}g_{n}^{n}\equiv g_{n}^{k}\pmod{p}$；
    3. $\omega_{n}^{k+\frac{n}{2}}=-\omega_{n}^{k}$，$(g_{n}^{k+\frac{n}{2}})^2\equiv g_{n}^{2k+n}\equiv g_{n}^{2k}\pmod{p}\Leftrightarrow g_{n}^{k+\frac{n}{2}}\equiv g_{n}^{k}\pmod{p}$。

可见，模意义下的单位根和普通单位根有很多相似之处，在某种意义下，可以直接互相替换。

## 快速数论变换（NTT）

我们发现：在 FFT 的过程中，只使用了上节中提到的三条有关单位根的性质。所以，我们可以直接用模意义下的单位根替换 FFT 中的所有 $\omega$。

但仍存在一个问题：根据模意义下的单位根 $g_n=g^{\frac{p-1}{n}}$ 的定义，发现 $n\mid p-1$，而在 FFT 中 $n$ 永远为 $2$ 的某个次幂。所以 NTT 对模数是有要求的，必须在减完 $1$ 后是一个 $2$ 的较大次幂的倍数。常见可做模数如下：

$$
\begin{aligned}
p&=167772161=5\times 2^{25}+1,&g=3\\
p&=469762049=7\times 2^{26}+1,&g=3\\
p&=754974721=3^2\times 5\times 2^{24}+1,&g=11\\
p&=998244353=7\times 17\times 2^{23}+1,&g=3\\
p&=1004535809=479\times 2^{21}+1,&g=3
\end{aligned}
$$

下文，我将同时给出 FFT 和 NTT 的代码，可以对比查看。

## 参考代码

???+ note "[P3803 【模板】多项式乘法（FFT）](https://www.luogu.com.cn/problem/P3803)"

    === "FFT"

        ```cpp
        #include<bits/stdc++.h>
        #define N 4000005
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
        //自定义复数类
        struct Complex
        {
            double re,im;
            Complex(double a=0,double b=0):re(a),im(b){}
            Complex operator +(Complex b) {return Complex(re+b.re,im+b.im);}
            Complex operator -(Complex b) {return Complex(re-b.re,im-b.im);}
            Complex operator *(Complex b) {return Complex(re*b.re-im*b.im,im*b.re+re*b.im);}
        }tmp[N];
        int rev[N],bit;
        constexpr double PI=acos(-1.0);
        void FFT(Complex* f,int n,int op)
        {
            for(int i=0;i<n;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<(bit-1));
            for(int i=0;i<n;i++) if(i<rev[i]) swap(f[i],f[rev[i]]);
            for(int len=2;len<=n;len<<=1)
            {
                Complex pw(cos(2*PI/len),sin(2*PI*op/len));
                for(int l=0,mid;l<n;l+=len)
                {
                    Complex cur(1,0);
                    mid=l+len/2;
                    for(int k=l;k<mid;k++)
                    {
                        Complex x=f[k],y=f[k+len/2];
                        f[k]=x+cur*y;
                        f[k+len/2]=x-cur*y;
                        cur=cur*pw;
                    }
                }
            }
            return;
        }
        int n,m,lim;
        Complex a[N],b[N];
        int main()
        {
            n=read(),m=read();
            for(int i=0;i<=n;i++) a[i].re=read();
            for(int i=0;i<=m;i++) b[i].re=read();
            lim=1;
            while(lim<=n+m) lim*=2,bit++;
            FFT(a,lim,1);
            FFT(b,lim,1);
            for(int i=0;i<=lim;i++) a[i]=a[i]*b[i];
            FFT(a,lim,-1);
            for(int i=0;i<=n+m;i++) printf("%d ",int(a[i].re/lim+0.5));
            return 0;
        }
        ```
    
    === "NTT"

        ```cpp
        #include<bits/stdc++.h>
        #define N 4000005
        #define mod 998244353
        #define G 3
        #define invG 332748118
        #define swap(x,y) ((x)^=(y)^=(x)^=(y))
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
        int n,m,bit,lim,a[N],b[N],rev[N];
        void NTT(int* f,int n,bool op)
        {
            for(int i=0;i<n;i++) if(i<rev[i]) swap(f[i],f[rev[i]]);
            for(int len=2,pw,cur;len<=n;len<<=1)
            {
                pw=qpow((op?G:invG),(mod-1)/len);
                for(int l=0,mid,x,y;l<n;l+=len)
                {
                    cur=1;
                    mid=l+len/2;
                    for(int k=l;k<mid;k++)
                    {
                        x=f[k],y=f[k+len/2];
                        f[k]=(x+1ull*cur*y%mod)%mod;
                        f[k+len/2]=(x-1ull*cur*y%mod+mod)%mod;
                        cur=1ull*cur*pw%mod;
                    }
                }
            }
            return;
        }
        int main()
        {
            n=read(),m=read();
            for(int i=0;i<=n;i++) a[i]=read();
            for(int i=0;i<=m;i++) b[i]=read();
            lim=1,bit=0;
            while(lim<=n+m) lim*=2,bit++;
            for(int i=0;i<=lim;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<(bit-1));
            NTT(a,lim,true);
            NTT(b,lim,true);
            for(int i=0;i<=lim;i++) a[i]=1ull*a[i]*b[i]%mod;
            NTT(a,lim,false);
            int inv=qpow(lim,mod-2);
            for(int i=0;i<=n+m;i++) printf("%lld ",1ll*a[i]*inv%mod);
            return 0;
        }
        ```

非常不建议使用递归做 NTT，因为如果每次递归算一次快速幂，复杂度会变成 $O(n\log^2 n)$ 还不如 FFT，而且没多少人会为此写预处理。迭代版就没有这个问题。