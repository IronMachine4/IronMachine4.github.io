# 拉格朗日插值法

对于 $x$ 坐标互不相同的 $n+1$ 个点 $(x_0,y_0),(x_1,y_1),\dots,(x_n,y_n)$，可以唯一确定一个多项式 $f(x)=\sum_{i=0}^{n-1}a_ix^i$ 满足 $\forall 0\le i\le n,f(x_i)=y_i$。朴素的拉格朗日插值法可以在 $O(n^2)$ 的时间复杂度求出这个多项式在另一个 $x$ 下的值。

设若干多项式 $f_0(x),f_1(x),\dots,f_n(x)$ 满足：

$$
f_i(x_j)=
\left\{
\begin{aligned}
&0&,i\neq j\\
&y_j&,i=j
\end{aligned}
\right.
$$

则 $f(x)=\sum_{i=0}^{n}f_i(x)$。

考虑构造 $f_i(x)$。我们可以设：

$$
f_i(x)=a\prod_{j\neq i}(x-x_j)
$$

这样就满足了 $f_i$ 在 $x_j$（$j\neq i$）处的取值为 $0$。

为了使 $f_i(x_i)=y_i$，

$$
a=\frac{f_i(x_i)}{\prod_{j\neq i}(x_i-x_j)}=\frac{y_i}{\prod_{j\neq i}(x_i-x_j)}
$$

代回原式得到：

$$
f_i(x)=y_i\prod_{j\neq i}\frac{x-x_j}{x_i-x_j}
$$

则：

$$
f(x)=\sum_{i=0}^{n}y_i\prod_{j\neq i}\frac{x-x_j}{x_i-x_j}
$$

这样我们就有了 $O(n^2)$ 的拉格朗日插值法。

如果运算是在模意义下的，要注意计算逆元的时间复杂度消耗。

???+ note "参考代码：[P4781 【模板】拉格朗日插值](https://www.luogu.com.cn/problem/P4781)"

    ```cpp
    #include<bits/stdc++.h>
    #define mod 998244353
    #define N 2005
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
    long long n,k,x[N],y[N],ans;
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
    long long inv(long long x) {return qpow(x,mod-2);}
    int main()
    {
        n=read()-1,k=read();
        for(int i=0;i<=n;i++) x[i]=read(),y[i]=read();
        for(int i=0;i<=n;i++)
        {
            long long res=y[i];
            for(int j=0;j<=n;j++) if(i!=j)
            {
                res=res*(k-x[j]+mod)%mod*inv((x[i]-x[j]+mod)%mod)%mod;
            }
            ans=(ans+res)%mod;
        }
        printf("%lld\n",ans);
        return 0;
    }
    ```

## 练习

???+ note "[P10832 [COTS 2023] 传 Mapa](https://www.luogu.com.cn/problem/P10832)"

    通信题。有 $n$ 个值域在 $[1,10^9]$ 的键值对，你需要用长度不超过 $L$ 的 01 串编码和解码这些键值对。$1\le n\le 100$，$L=3000$。

    ??? note "题解"

        首先 $[1,10^9]$ 之间的数用二进制表示需要 $30$ 位，长度限制实际上告诉我们用 $n$ 个数压缩原来的 $2n$ 个数。

        键值对的形式让我们想到函数，我们将键值对看作函数上的点，就可以用拉插刻画这个函数。

        多项式有两种表达方式，一种是系数表示法，另一种是点值表示法。显然系数表示法过于暴力且麻烦，我们可以钦定 $f$ 在 $0$ 到 $n-1$ 上取值，把 $f(0)$ 到 $f(n-1)$ 的值记录下来，这样就省去了点值表示法记录横坐标的开销。

        时间复杂度 $O(n^3)$。

    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 105
        #define B 31
        #define mod 1000000007
        using namespace std;
        long long n,q,x[N],y[N];
        string s;
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
        long long inv(long long x){return qpow(x,mod-2);}
        long long f(long long k)
        {
            long long ans=0;
            for(int i=0;i<=n;i++)
            {
                long long res=y[i];
                for(int j=0;j<=n;j++) if(i!=j)
                {
                    res=res*((k-x[j]+mod)%mod)%mod*inv((x[i]-x[j]+mod)%mod)%mod;
                }
                ans=(ans+res)%mod;
            }
            return ans;
        }
        string DtoS(long long x)
        {
            string res;
            for(int i=0;i<30;i++) res.push_back(char((x&1)+'0')),x>>=1;
            reverse(res.begin(),res.end());
            return res;
        }
        long long StoD(string s)
        {
            long long res=0;
            for(int i=0;i<s.size();i++)
            {
                res=(res<<1)|(s[i]-'0');
            }
            return res;
        }
        void solve1()
        {
            cin>>n;n--;
            for(int i=0;i<=n;i++) cin>>x[i]>>y[i];
            for(int i=0;i<=n;i++) s+=DtoS(f(i));
            cout<<s.size()<<'\n'<<s<<endl;
            return;
        }
        void solve2()
        {
            int len;
            cin>>n>>q>>len>>s;n--;
            for(int i=0,l=0,r=29;i<=n;i++,l+=30,r+=30)
            {
                x[i]=i;
                y[i]=StoD(s.substr(l,r-l+1));
            }
            while(q--)
            {
                int k;
                cin>>k;
                cout<<f(k)<<'\n';
            }
            return;
        }
        int main()
        {
            int t;
            cin>>t;
            if(t==1) solve1();
            else solve2();
            return 0;
        }
        ```