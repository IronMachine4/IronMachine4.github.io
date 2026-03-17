# 拓展欧几里得算法

拓展欧几里得算法（缩写 exgcd），常用于求一次不定方程 $ax+by=c$ 的**整数**解。

## 裴蜀定理

裴蜀定理（贝祖定理），给出了判定一次不定方程**有整数解**的充要条件。

???+ abstract "裴蜀定理"
    设 $a,b$ 为不全为 $0$ 的整数，那么：

    1. 对于任意整数 $x,y$，都有 $\gcd(a,b)\mid ax+by$；
    2. 存在一组整数 $x,y$，使得 $ax+by=\gcd(x,y)$。

第一条是显然的。而第二条的证明可以由下文的 exgcd 过程给出。

通过裴蜀定理，我们得知：

???+ abstract "二元一次不定方程整数解的判定"
    关于 $x,y$ 的方程 $ax+by=c$ 有整数解，当且仅当 $\gcd(a,b)\mid c$。

## 拓展欧几里得算法

拓展欧几里得算法，是在使用欧几里得算法（辗转相除法）求整数 $\gcd(a,b)$ 的同时求出 $ax+by=\gcd(a,b)$ 的一组特解。

考虑辗转相除法时，我们有等式：

$$
\gcd(a,b)=\gcd(b,a\bmod b)
$$

那么，我们可以列两个方程：

$$
\left\{
\begin{aligned}
&ax_1+by_1=\gcd(a,b)\\
&bx_2+(a\bmod b)y_2=\gcd(b,a\bmod b)
\end{aligned}
\right.
$$

假设我们知道了 $(x_2,y_2)$ 的一组特解，我们用 $(x_2,y_2)$ 来表示 $(x_1,y_1)$。因为 $\gcd(a,b)=\gcd(b,a\bmod b)$，则：

$$
\begin{aligned}
ax_1+by_1&=bx_2+(a\bmod b)y_2\\
ax_1+by_1&=bx_2+(a-b\lfloor\frac{a}{b}\rfloor)y_2\\
ax_1+by_1&=bx_2+ay_2-b\lfloor\frac{a}{b}\rfloor y_2\\
ax_1+by_1&=ay_2+b(x_2-\lfloor\frac{a}{b}\rfloor y_2)
\end{aligned}
$$

那么：

$$
\left\{
\begin{aligned}
&x_1=y_2\\
&y_1=x_2-\lfloor\frac{a}{b}\rfloor y_2
\end{aligned}
\right.
$$

是 $ax_1=by_1=\gcd(a,b)$ 的一组特解。

发现欧几里得算法到最后一步时 $b=0$，此时 $ax+by=\gcd(a,b)$ 的特解显然为 $x=1,y=0$。我们只要在递归回溯时求出当前方程的特解，就能解出初始方程的特解了。时间复杂度 $O(\log V)$。

??? note "参考代码"
    
    ```cpp
    long long exgcd(long long a,long long b,long long& x,long long& y)//调用 exgcd(a,b,x,y)，返回 gcd(a,b)，x,y 存储 ax+by=gcd(a,b) 的一组解。
    {
        if(!b) return x=1,y=0,a;
        long long d=exgcd(b,a%b,x,y);
        (y^=x^=y^=x)-=a/b*x;
        return d;
    }
    ```

## 二元一次不定方程整数通解

知道了 $ax+by=\gcd(a,b)$ 的一组解 $(x^*,y^*)$，则对于有整数解的方程 $ax+by=c$，其特解 $(x_0,y_0)$ 为 $(\frac{c}{\gcd(a,b)}x^*,\frac{c}{\gcd(a,b)}y^*)$。

现在考虑通解。把 $ax+by=c$ 看作直线 $y=-\frac{a}{b}x+\frac{c}{a}$，则 $ax+by=c$ 的通解就是直线上的整点。因为 $(x_0,y_0)$ 和 $(x_0+b,y_0-a)$ 均为线上整点，则 $(x_0+\frac{b}{d},y_0-\frac{a}{d})$ 在直线上。显然，令 $\frac{a}{d},\frac{b}{d}$ 为整数的最大 $d=\gcd(a,b)$。所以原方程的通解为：

$$
\left\{
\begin{aligned}
x&=\frac{c}{\gcd(a,b)}x^*+\frac{b}{\gcd(a,b)}t\\
y&=\frac{c}{\gcd(a,b)}y^*+\frac{a}{\gcd(a,b)}t
\end{aligned}
\right.
,t\in\mathbb{Z}
$$

## 应用

exgcd 可以用于求乘法逆元。对于互质的整数 $a,p$，若 $ax\equiv 1\pmod{p}$，则 $x$ 是 $a$ 模 $p$ 意义下的乘法逆元，记作 $a^{-1}$。求出了乘法逆元，我们就可以对任何有理数取模了，只需分子乘上分母的乘法逆元即可。

对于 $ax\equiv 1\pmod{p}$，我们可以改写成不定方程 $ax+py=1$，这样我们就可以求出 $x$ 的通解，答案为 $x$ 的最小正整数解。

??? note "[参考代码：P3811 【模板】模意义下的乘法逆元](https://www.luogu.com.cn/problem/P3811) 80 pts"

    ```cpp
    #include<bits/stdc++.h>
    #define N 3000005
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
    inline void write(int x)
    {
        x<0?x=-x,putchar_unlocked('-'):0;
        static short Stack[50],top(0);
        do Stack[++top]=x%10,x/=10; while(x);
        while(top) putchar_unlocked(Stack[top--]|48);
    }
    int n,mod;
    int exgcd(int a,int b,int& x,int& y)
    {
        if(!b) return x=1,y=0,a;
        long long d=exgcd(b,a%b,x,y);
        (y^=x^=y^=x)-=a/b*x;
        return d;
    }
    int inv(int a)
    {
        int x,y,d=exgcd(a,mod,x,y);
        return (x%mod+mod)%mod;
    }
    int main()
    {
        n=read(),mod=read();
        for(int i=1;i<=n;i++) write(inv(i)),putchar_unlocked('\n');
        return 0;
    }
    ```