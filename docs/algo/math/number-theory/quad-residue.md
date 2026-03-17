# 二次剩余

二次剩余用于解决模意义下的开平方问题。

对于整数 $a,p$ 满足 $\gcd(a,p)=1$，若存在 $x$ 使得

$$
x^2\equiv a\pmod{p}
$$

那么称 $a$ 为模 $p$ 的**二次剩余**。否则称 $a$ 为模 $p$ 的**二次非剩余**。

下文假设整数不为 $0$。（$0$ 的二次剩余为 $0$）

## Euler 判别法

由定义可知并非所有数都有二次剩余。为判定是否存在二次剩余，有定理：

???+ abstract "Euler 判别法"

    对于奇质数 $p$ 和整数 $a$ 满足 $\gcd(a,b)=1$，
    
    1. $a$ 是 $p$ 的二次剩余当且仅当 $a^{\frac{p-1}{2}}\equiv 1\pmod{p}$；
    2. $a$ 是 $p$ 的二次非剩余当且仅当 $a^{\frac{p-1}{2}}\equiv -1\pmod{p}$。

证明见 [OI-Wiki](https://oi-wiki.org/math/number-theory/quad-residue/#euler-%E5%88%A4%E5%88%AB%E6%B3%95)。

由定理可知，对于任意奇质数 $p$，模 $p$ 意义下二次剩余和非二次剩余的数量均为 $\frac{p-1}{2}$。

## Cippolla 算法

Cippolla 算法在 $O(\log p)$ 的时间复杂度内求出整数 $n$ 对奇质数 $p$ 的二次剩余。

设 $a$ 满足 $a^2-n$ 对 $p$ 二次非剩余，那么我们可以设一个类似虚数单位的数 $i$ 满足 $i^2\equiv a^2-n\pmod{p}$。此时我们需要若干引理：

???+ abstract "引理 1：费马小定理"
    对于任意整数 $a$ 和质数 $p$，都有：
    
    $$
    a^p\equiv a\pmod{p}
    $$
    证明见 [OI-Wiki](https://oi-wiki.org/math/number-theory/fermat/#%E8%B4%B9%E9%A9%AC%E5%B0%8F%E5%AE%9A%E7%90%86)。

???+ abstract "引理 2"
    对于任意整数 $a,b$ 和质数 $p$，有：

    $$
    (a+b)^p\equiv a^p+b^p\pmod{p}
    $$
    ??? note "证明"

        $$
        \begin{aligned}
        (a+b)^p&=\sum_{i=0}^{p}\binom{p}{i}a^{i}b^{p-i}\\
        &=\sum_{i=0}^{p}\frac{p!}{i!(p-i)!}a^{i}b^{p-i}\\
        &\equiv a^{p}b^{0}+a^{0}b^{p} &\pmod{p}\\
        &=a^p+b^p
        \end{aligned}
        $$

???+ abstract "引理 3"
    对于奇质数 $p$，有：
    $$
    i^{p}\equiv i(i^{2}) ^{\frac{p-1}{2}}\equiv i(a^2-n) ^{\frac{p-1}{2}}\equiv -i\pmod{p}
    $$

那么，

$$
\begin{aligned}
(a+i)^{p+1} &\equiv(a+i)^p(a+i) &\pmod{p}\\
&\equiv(a^p+i^p)(a+i) &\pmod{p}\\
&\equiv(a^p-i)(a+i) &\pmod{p}\\
&\equiv(a-i)(a+i) &\pmod{p}\\
&= a^2-i^2\\
&= a^2-a^2+n\\
&= n
\end{aligned}
$$

所以 $\pm (a+i)^{\frac{p+1}{2}}$ 是 $a$ 的二次剩余。

??? note "参考代码：[P5491 【模板】二次剩余](https://www.luogu.com.cn/problem/solution/P5491)"
    
    ```cpp
    #include<bits/stdc++.h>
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
    long long n,mod,sqri;
    struct Complex
    {
        long long real,imag;
        Complex():real(0),imag(0){}
        Complex(long long x,long long y):real(x),imag(y){}
        bool operator ==(const Complex& b)const {return real==b.real&&imag==b.imag;}
        Complex operator *(const Complex& b)const
        {
            return Complex(
                (real*b.real%mod+imag*b.imag%mod*sqri%mod)%mod,
                (imag*b.real%mod+real*b.imag%mod)%mod
            );
        }
    };
    long long qpow(long long a,long long b)
    {
        a%=mod;
        long long res=1;
        while(b)
        {
            if(b&1) res=res*a%mod;
            a=a*a%mod;
            b>>=1;
        }
        return res;
    }
    Complex qpow(Complex a,long long b)
    {
        Complex res(1,0);
        while(b)
        {
            if(b&1) res=res*a;
            a=a*a;
            b>>=1;
        }
        return res;
    }
    bool check(long long x) {return qpow(x,(mod-1)/2)==1;}
    mt19937 rnd((random_device())());
    void solve()
    {
        n=read(),mod=read();
        if(n==0) return void(puts("0"));
        if(!check(n)) return void(puts("Hola!"));
        long long a=rnd()%mod;
        while(!a||check((a*a%mod-n+mod)%mod)) a=rnd()%mod;
        sqri=(a*a%mod-n+mod)%mod;
        long long ans1=qpow(Complex(a,1),(mod+1)/2).real,ans2=mod-ans1;
        if(ans1>ans2) swap(ans1,ans2);
        printf("%lld %lld\n",ans1,ans2);
        return;
    }
    int main()
    {
        int t=read();
        while(t--) solve();
        return 0;
    }
    ```