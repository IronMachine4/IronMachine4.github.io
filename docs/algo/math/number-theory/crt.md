# 中国剩余定理

中国剩余定理用于解决如下的线性同余方程组：

$$
\left\{
\begin{aligned}
x&\equiv a_1\pmod{p_1}\\
x&\equiv a_2\pmod{p_2}\\
&\dots\\
x&\equiv a_n\pmod{p_n}
\end{aligned}
\right.
$$

## 中国剩余定理

以下过程只在 $p$ **两两互质**时成立。

1. 求出 $M=\prod_{i=1}^{n}p_i$；
2. 令 $q_{i}=\frac{M}{p_i}$，$q_i^{-1}$ 是 $q_i$ 在模 $p_i$ 意义下的乘法逆元，则再令 $c_i=q_i q_{i}^{-1}$（**$c$ 不应对 $p$ 取模**）；
3. 方程在模 $M$ 意义下的唯一解为：$x=\sum_{i=1}^{n}a_i c_i \pmod{M}$。

??? note "证明"

    当 $i\neq j$ 时，有 $p_i\mid q_j$，所以 $p_i\mid c_j$。又有 $c_i=q_{i}(q_{i}^{-1}\bmod p_i)$，所以：

    $$
    \begin{aligned}
    x&\equiv\sum_{i=1}^{n}a_i c_i &\pmod{p_i}\\
    &\equiv a_i c_i &\pmod{p_i}\\
    &\equiv a_i q_{i}(q_{i}^{-1}\bmod p_i) &\pmod{p_i}\\
    &\equiv a_i &\pmod{p_i}
    \end{aligned}
    $$

    故通过上文得到的 $x$ 满足所有方程的约束条件，即 $x$ 为原方程组的解。

??? note "参考代码：[P1495 【模板】中国剩余定理（CRT）/ 曹冲养猪](https://www.luogu.com.cn/problem/P1495)"

    ```cpp
    #include<iostream>
    using namespace std;
    long long m[20],M[20],b[20],n,t[20],MM=1;
    long long mul(long long a,long long b)
    {
        long long ans=0;
        while(b!=0)
        {
            if(b&1) ans+=a,ans%=MM;
            a<<=1;
            a%=MM;
            b>>=1;
        }
        return ans;
    }
    void exgcd(long long a,long long b,long long &x,long long &y)
    {
        if(b==0)
        {
            x=1;
            y=0;
            return;
        }
        exgcd(b,a%b,y,x);
        y-=a/b*x;
    }
    long long niyuan(long long a,long long mod)
    {
        long long ans,tmp;
        exgcd(a,mod,ans,tmp);
        return (ans%mod+mod)%mod;
    }
    long long ans;
    int main()
    {
        cin>>n;
        for(int i=1;i<=n;i++)
        {
            cin>>m[i]>>b[i];
            MM*=m[i];
        }
        for(int i=1;i<=n;i++)
        {
            M[i]=MM/m[i];
            t[i]=niyuan(M[i],m[i]);
        }
        for(int i=1;i<=n;i++)
        {
            ans+=mul(b[i],mul(t[i],M[i]));
            ans%=MM;
        }
        cout<<ans<<'\n';
        return 0;
    }
    ```

## 拓展中国剩余定理（excrt）

上文方法只在 $p$ 两两互质的时候成立。考虑如果 $p$ 不满足这个条件怎么做。

我们尝试合并两个同余方程：

$$
\left\{
\begin{aligned}
x&\equiv a_1\pmod{p_1}\\
x&\equiv a_2\pmod{p_2}
\end{aligned}
\right.
$$

可以改写为不定方程：

$$
\left\{
\begin{aligned}
x&=p_{1}n+a_1\\
x&=p_{2}m+a_2
\end{aligned}
\right.
$$

即

$$
\begin{aligned}
p_{1}n+a_1&=p_{2}m+a_2\\
p_{1}n-p_{2}m&=a_2-a_1
\end{aligned}
$$

这是一个关于 $n,m$ 的不定方程。如果不定方程无解，则原同余方程组一定无解。否则我们使用 exgcd 解出一组特解，有：

$$
x\equiv p_1 n+a_1\pmod{\text{lcm}(p_1,p_2)}
$$

这样就把两个同余方程合并成了一个。对于多个方程式，按如上方法两两合并即可。

???+ note "[例题：P4774 [NOI2018] 屠龙勇士](https://www.luogu.com.cn/problem/P4774)"

    $n$ 条龙，初始 $m$ 把剑。第 $i$ 条龙有 $a_i$ 点血量，并且一次回血可恢复 $p_i$ 点。第 $i$ 把剑一次攻击扣除 $t_i$ 点血，杀死一条龙还会赠送一把剑。从 $1$ 到 $n$ 挑战每一条龙，每次选择攻击力不大于这条龙的攻击力最大的剑（没有就是攻击力最小的剑），并攻击这条龙 $x$ 次（血量可以为负）。之后这条龙尝试回血若干次，如果某次回血后血量为 $0$ 视为这条龙死亡，如果无论如何都到不了 $0$ 则视为挑战失败。求最小的 $x$ 使得每条龙都会死亡，或报告无解。$1\le n,m\le 10^5$。

    ??? note "题解"

        这道题的题面比较复杂。首先，对每条龙选择“攻击力不大于这条龙的攻击力最大的剑”是简单的，multiset 就可以解决。记对第 $i$ 条龙使用了攻击力为 $b_i$ 的剑，则题目就是让我们求以下同余方程组的最小正整数解：

        $$
        \left\{
        \begin{aligned}
        b_1 x&\equiv a_1\pmod{p_1}\\
        b_2 x&\equiv a_2\pmod{p_2}\\
        \dots\\
        b_n x&\equiv a_n\pmod{p_n}\\
        \end{aligned}
        \right.
        $$

        这与 excrt 的形式几乎一样，不同在于 $x$ 多了一个系数。考虑消掉系数。

        把同余方程改写为不定方程：

        $$
        b_i x+p_i y=a_i
        $$

        此时用 exgcd 求出方程 $b_i x+p_i y=\gcd(b_i,p_i)$ 的一组特解 $(x_0,y_0)$，则通解为

        $$
        x=\frac{a_i}{\gcd(b_i,p_i)}x_0+\frac{p_i}{\gcd(b_i,p_i)}t,t\in\mathbb{Z}
        $$

        即

        $$
        x\equiv\frac{a_i}{\gcd(b_i,p_i)}x_0\pmod{\frac{p_i}{\gcd(b_i,p_i)}}
        $$

        这样就把系数消掉了。后续过程就是 excrt 了。

        注意：打死龙有一个条件其实是砍 $x$ 刀后血量得不大于 $0$，所以最终的解要大于 $\max_{i=1}^{n}\lfloor\frac{a_i}{b_i}\rfloor$。
    
    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 100005
        #define int128 __int128
        using namespace std;
        char buf[1<<23],*p1=buf,*p2=buf;
        #define getchar() (p1==p2&&(p2=(p1=buf)+fread(buf,1,1<<21,stdin),p1==p2)?EOF:*p1++)
        inline int128 read()
        {
            int128 x=0,f=1;
            char ch=getchar();
            while(!isdigit(ch))
            {
                if(ch=='-') f=-1;
                ch=getchar();
            }
            while(isdigit(ch)) x=x*10+(ch^48),ch=getchar();
            return x*f;
        }
        inline void write(int128 x)
        {
            x<0?x=-x,putchar_unlocked('-'):0;
            static short Stack[50],top(0);
            do Stack[++top]=x%10,x/=10; while(x);
            while(top) putchar_unlocked(Stack[top--]|48);
        }
        int128 n,m,a[N],p[N],atk[N],prize[N],b[N],mx;
        multiset<int128> s;
        int128 exgcd(int128 a,int128 b,int128& x,int128& y)
        {
            if(!b) return x=1,y=0,a;
            int128 d=exgcd(b,a%b,x,y);
            (y^=x^=y^=x)-=a/b*x;
            return d;
        }
        int128 solve(int t)
        {
            mx=0;s.clear();
            n=read(),m=read();
            for(int i=1;i<=n;i++) a[i]=read();
            for(int i=1;i<=n;i++) p[i]=read();
            for(int i=1;i<=n;i++) prize[i]=read();
            for(int i=1;i<=m;i++) atk[i]=read();
            for(int i=1;i<=m;i++) s.insert(atk[i]);
            for(int i=1;i<=n;i++)
            {
                auto it=s.upper_bound(a[i]);
                if(it!=s.begin()) --it;
                b[i]=*it;
                mx=max(mx,(a[i]-1)/b[i]+1);
                s.erase(it);
                s.insert(prize[i]);
            }
            for(int i=1;i<=n;i++)
            {
                b[i]%=p[i],a[i]%=p[i];
                if(!b[i]&&!a[i]) {a[i]=0,p[i]=1;continue;}
                if(!b[i]&&a[i]) return -1;
                int128 x,y,d=exgcd(b[i],p[i],x,y);
                if(a[i]%d) return -1;
                p[i]=p[i]/d;
                a[i]=a[i]/d*((x%p[i]+p[i])%p[i])%p[i];
            }
            int128 A=0,M=1;
            for(int i=1;i<=n;i++)
            {
                int128 x,y,d=exgcd(M,p[i],x,y);
                if((a[i]-A)%d) return -1;
                int128 t=M;
                M=M/d*p[i];
                A=(A+t/d*((a[i]-A)%M+M)%M*(x%M+M)%M)%M;
            }
            return A>=mx?A:A+M*((mx-A+M-1)/M);
        }
        int main()
        {
            int t=read();
            while(t--) write(solve(t)),putchar_unlocked('\n');
            return 0;
        }
        ```