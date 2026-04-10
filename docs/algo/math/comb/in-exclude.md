# 容斥原理

容斥原理是一种计数技巧，用于解决计数中出现的集合之间交和并等操作。

## 容斥原理

容斥原理的公式长这样：

$$
\begin{aligned}
f(S)=\sum_{T\subseteq S}g(T)\Leftrightarrow g(S)=\sum_{T\subseteq S}(-1)^{|S|-|T|}f(T)\\
f(S)=\sum_{S\subseteq T}g(T)\Leftrightarrow g(S)=\sum_{S\subseteq T}(-1)^{|T|-|S|}f(T)
\end{aligned}
$$

通常用于“至多/至少”与“恰好”之间的转换。有的计数题想要我们求出“恰好 $n$ 个的方案”，而我们求 “至多/至少 $n$ 个的方案” 通常会更加容易，此时可以尝试容斥原理。

容斥原理非常常见，计数题应积极往这个角度思考。

???+ note "[P6662 [POI 2019/2020 R1] Przedszkole / 幼儿园](https://www.luogu.com.cn/problem/P6662)"

    给定 $n$ 个点 $m$ 条边的图，$q$ 次询问，每次给出一个正整数 $k$，将图上的点染成 $k$ 种颜色中的一种，要求相邻的两个点不同色。问方案数。

    $q\le 1000$，$k\le 10^9$。

    Subtask 1: $n\le 15$；  
    Subtask 2: $m\le 24,n\le 10^5$；  
    Subtask 3: $n\le 10^5$，图为若干个环。

    ??? note "题解"

        首先，这道题三个 subtask 是独立的。

        **Subtask 1:**

        注意到 $n$ 较小，考虑从 $n$ 的角度进行状压 DP。

        令 $f_{i,S}$ 表示已经使用了 $i$ 种颜色，$S$ 中的点已经被染色的方案数。考虑现在染第 $i+1$ 种颜色。明显地，对于一种颜色所占的点一定是独立集。令 $c(S)$ 表示 $S$ 是否为独立集，有递推式：

        $$
        f_{i,S}=\sum_{T\subseteq S}c(T)f_{i-1,S-T}
        $$

        直接做即可。发现：$n$ 个点最多染 $n$ 种颜色，所以 DP 是与询问无关的。用 $k$ 种颜色染色的方案数为 $\sum_{i=1}^{n}\binom{k}{i}f_{i,2^n-1}$。

        时间复杂度 $O(n3^{n}+qn)$。

        **Subtask 2:**

        注意到 $m$ 较小，考虑从边的角度做这件事情。

        边的端点颜色不同是难做的。考虑正难则反，我们钦定若干个联通的点颜色相同，并对其进行容斥，就能得到所有相邻点颜色不同的方案数了。

        令 $f(S)$ 表示至少 $S$ 中的点不符合条件的染色方案数，$g(S)$ 表示恰好只有 $S$ 中的点不符合条件的染色方案数。有：

        $$
        f(S)=\sum_{S\subseteq T}g(T)
        $$

        容斥一下得：

        $$
        g(\varnothing)=\sum_{S\subseteq V}(-1)^{|S|}f(S)
        $$

        而 $f(S)$ 是好求的，设只保留了 $S$ 中的点之间的边，剩下 $cnt$ 个连通块，则 $f(S)=k^{cnt}$。

        观察到：如果一个点是孤点，其一定不能统计入 $S$，因为孤点一定合法。那么，$S$ 的取值只有 $2^m$ 种：以边集的任意一个子集涉及到的点为 $S$。

        具体实现上，我们可以直接对边集 DFS，并用可撤销并查集维护连通块。为了避免每询问一次就 DFS 一次拉低复杂度，我们直接预处理所有连通块个数为 $i$ 的所有 $S$ 的容斥系数之和。

        时间复杂度 $O(2^m\log n+qn)$。

        **Subtask 3:**

        在环上的染色是简单的。令 $f_i$ 为长度为 $i$ 的环的染色方案。假设破环为链，在链上染色方案数为 $k(k-1)^{i-1}$，但是我们还要减去首尾相同的链染色方案，发现首尾相同要求第 $1$ 和 $i-1$ 的节点的染色不同，这个方案数是 $f_{i-1}$ 的。故：

        $$
        f_{i}=k(k-1)^{i-1}-f_{i-1}
        $$

        递推转通项为：

        $$
        f_{i}=(k-1)^{n}+(-1)^{n}(k-1)
        $$

        由于本质不同的环个数只有 $\sqrt{n}$ 个，时间复杂度为 $O(q\sqrt{n})$（实际上直接 $O(nq)$ 也能过）。

    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 100005
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
        int n,m,q,now,head[N],to[N*2],nxt[N*2];
        void add(int x,int y)
        {
            to[++now]=y;
            nxt[now]=head[x];
            head[x]=now;
            return;
        }
        namespace s1
        {
            #undef N
            #define N 16
            int e[N];
            bool flg[1<<N];
            long long f[1<<N][N],inv[N];
            void solve()
            {
                inv[1]=1;
                for(int i=2;i<=n;i++) inv[i]=inv[mod%i]*(mod-mod/i)%mod;
                for(int u=1;u<=n;u++)
                {
                    for(int i=head[u];i;i=nxt[i])
                    {
                        int v=to[i];
                        e[u]|=(1<<(v-1));
                    }
                }
                for(int s=1;s<(1<<n);s++)
                {
                    flg[s]=true;
                    for(int i=1;i<=n;i++)
                    {
                        if(((s>>(i-1))&1)&&(s&e[i]))
                        {
                            flg[s]=false;
                            break;
                        } 
                    }
                }
                f[0][0]=1;
                for(int i=1;i<=n;i++)
                {
                    for(int s=0;s<(1<<n);s++)
                    {
                        for(int t=s;t;t=s&(t-1)) if(flg[t])
                        {
                            f[s][i]=(f[s][i]+f[s-t][i-1])%mod;
                        }
                    }
                }
                for(int i=1;i<=q;i++)
                {
                    int k=read();
                    long long ans=0;
                    for(long long i=1,C=k;i<=min(n,k);i++,C=C*(k-i+1)%mod*inv[i]%mod)
                    {
                        ans=(ans+C*f[(1<<n)-1][i]%mod)%mod;
                    }
                    printf("%lld\n",ans);
                }
                return;
            }
        }
        namespace s2
        {
            #undef N
            #define N 100005
            #define M 30
            int fa[N],sz[N],s[N],top;
            void swap(int& x,int& y) {x^=y^=x^=y;}
            int find(int x)
            {
                if(x==fa[x]) return x;
                return find(fa[x]);
            }
            void merge(int x,int y)
            {
                x=find(x),y=find(y);
                if(x==y) return;
                if(sz[x]>sz[y]) swap(x,y);
                s[++top]=x,fa[x]=y,sz[y]+=sz[x];
                return;
            }
            void back()
            {
                if(!top) return;
                int x=s[top--],y=fa[x];
                sz[y]-=sz[x],fa[x]=x;
                return;
            }
            int U[N],V[N],c[N];
            void dfs(int dep,int cnt,int tot)
            {
                if(dep==m+1)
                {
                    c[cnt]=(c[cnt]+(tot&1?mod-1:1))%mod;
                    return;
                }
                if(find(U[dep])==find(V[dep])) return;
                dfs(dep+1,cnt,tot);
                merge(U[dep],V[dep]);
                dfs(dep+1,cnt-1,tot+1);
                back();
            }
            void solve()
            {
                for(int i=1;i<=n;i++) fa[i]=i,sz[i]=1;
                dfs(1,n,0);
                while(q--)
                {
                    int k=read();
                    long long ans=0;
                    for(long long i=1,pw=k;i<=n;i++,pw=pw*k%mod)
                    {
                        ans=(ans+pw*c[i]%mod+mod)%mod;
                    }
                    printf("%lld\n",ans);
                }
                return;
            }
        }
        namespace s3
        {
            #undef N
            #define N 100005
            long long f[N],cnt[N],fa[N],sz[N];
            int find(int x)
            {
                if(x==fa[x]) return x;
                return fa[x]=find(fa[x]);
            }
            void merge(int x,int y)
            {
                x=find(x),y=find(y);
                if(x==y) return;
                if(sz[x]>sz[y]) s2::swap(x,y);
                fa[x]=y,sz[y]+=sz[x];
                return;
            }
            void solve()
            {
                for(int i=1;i<=n;i++) fa[i]=i,sz[i]=1;
                for(int i=1;i<=m;i++) merge(s2::U[i],s2::V[i]);
                for(int i=1;i<=n;i++) if(fa[i]==i) cnt[sz[i]]++;
                while(q--)
                {
                    long long k=read();
                    for(long long i=1,pw=(k-1);i<=n;i++,pw=pw*(k-1)%mod) f[i]=(pw+(i&1?mod-1:1)*(k-1)%mod)%mod;
                    f[1]=k;
                    long long ans=1;
                    for(int i=1;i<=n;i++) for(int j=1;j<=cnt[i];j++) ans=ans*f[i]%mod;
                    printf("%lld\n",ans);
                }
            }
        }
        int main()
        {
            n=read(),m=read(),q=read();
            for(int i=1;i<=m;i++)
            {
                int u=read(),v=read();
                add(u,v);
                add(v,u);
                s2::U[i]=u;
                s2::V[i]=v;
            }
            if(n<=15) s1::solve();
            else if(m<=24) s2::solve();
            else s3::solve();
            return 0;
        }
        ```

???+ note "[P3349 [ZJOI2016] 小星星](https://www.luogu.com.cn/problem/P3349)"

    给定 $n$ 个点的图和 $n$ 个点的树，节点均有标号。求图中有多少生成树与给定树同构。两棵生成树不同当且仅当图中有一个节点在两棵生成树中对应给定树上的节点不同。$n\le 17$。

    ??? note "题解"

        显然搞这张图是没有前途的。考虑树形 DP。

        令 $f_{i,S}$ 为 $i$ 的子树中的点对应图中的点集 $S$ 的方案数。但这样显然是不够的，因为从儿子推父亲无法确定在图上的相连关系。考虑再加一维，$f_{i,j,S}$ 为树上 $i$ 对应图中 $j$，$i$ 子树中的点对应图中点集 $S$ 的方案数，这样就可以做了。

        但是，如果这样做，儿子向父亲推的时候还要考虑 $S$ 有没有交，这是子集卷积，时间复杂度 $O(n^42^{n})$ 无法通过。

        考虑放宽限制：不考虑树上的点有没有对应到图中的同一个点，不做任何限制，只要 $u$ 在树上有 $i$ 个相邻的点，在图上的“生成树”也有 $i$ 个相邻的点就可以。这是简单的，可以省略 $S$ 那一维（因为此时没有意义），时间复杂度 $O(n^3)$。

        考虑如何从这样的 DP 得到原问题的答案。注意到：我们要求树上的节点到图上的节点的映射能构成一棵合法的生成树，等价于放宽限制的“生成树”使用了图中的 $n$ 个点。而放宽限制的 DP 可以求出至多允许使用图的点集的一个子集中的点，“生成树”的方案数，所以可以容斥。
        
        时间复杂度 $O(n^32^n)$。
    
    ??? note "参考代码"

        ```cpp
        //输出 long long 的时候用 %lld 了吗 ~~~
        //交之前改 freopen 了吗 ~~~
        //改完代码及时交了吗 ~~~
        //算了内存不会 MLE 了吗 ~~~
        //T1 卡住看 T2 了吗 ~~~
        //树边存储开两倍了吗 ~~~
        //测时间的时候关掉 sanitizer 了吗 ~~~
        #include<bits/stdc++.h>
        #define N 18
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
        int n,m,now,head[N],to[N*2],nxt[N*2];
        bool e[N][N];
        void add(int x,int y)
        {
            to[++now]=y;
            nxt[now]=head[x];
            head[x]=now;
            return;
        }
        bool vis[N];
        long long f[N][N];
        void dfs(int x,int fa,int S)
        {
            for(int i=head[x];i;i=nxt[i])
            {
                int v=to[i];
                if(v==fa) continue;
                dfs(v,x,S);
            }
            for(int i=1;i<=n;i++)
            {
                if((S>>(i-1))&1) f[x][i]=1;
                else f[x][i]=0;
            }
            for(int i=1;i<=n;i++) if((S>>(i-1))&1)
            {
                for(int j=head[x];j;j=nxt[j])
                {
                    int v=to[j];
                    if(v==fa) continue;
                    long long res=0;
                    for(int k=1;k<=n;k++) if((S>>(k-1))&1) if(e[i][k])
                    {
                        res+=f[v][k];
                    }
                    f[x][i]*=res;
                }
            }
            return;
        }
        long long ans;
        int main()
        {
            n=read(),m=read();
            for(int i=1;i<=m;i++)
            {
                int u=read(),v=read();
                e[u][v]=e[v][u]=true;
            }
            for(int i=1;i<n;i++)
            {
                int u=read(),v=read();
                add(u,v);
                add(v,u);
            }
            for(int S=0;S<(1<<n);S++)
            {
                dfs(1,0,S);
                long long sum=0;
                for(int i=1;i<=n;i++) sum+=f[1][i];
                if((n-__builtin_popcount(S))&1) ans-=sum;
                else ans+=sum; 
            }
            printf("%lld\n",ans);
            return 0;
        }
        ```

## Min-max 容斥

设我们有一个序列 $x$，长度为 $n$，再令 $S=\{1,2,\dots,n\}$ 那我们有：

$$
\def\op{\operatorname}
\begin{gather}
\max_{i\in S}x_{i}=\sum_{T\subseteq S}(-1)^{|T|-1}\min_{j\in T}x_{j}\\
\min_{i\in S}x_{i}=\sum_{T\subseteq S}(-1)^{|T|-1}\max_{j\in T}x_{j}\\
\underset{i\in S}{\op{kthmax}}x_{i}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}\min_{j\in T}x_j\\
\underset{i\in S}{\op{kthmin}}x_{i}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}\max_{j\in T}x_j
\end{gather}
$$

最关键的是，如果 $x_i$ 为随机变量，那么以上公式对期望也是成立的：

$$
\def\op{\operatorname}
\def\bracket#1{\left(#1\right)}
\begin{gather}
E\bracket{\max_{i\in S}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-1}E\bracket{\min_{j\in T}x_{j}}\\
E\bracket{\min_{i\in S}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-1}E\bracket{\max_{j\in T}x_{j}}\\
E\bracket{\underset{i\in S}{\op{kthmax}}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}E\bracket{\min_{j\in T}x_j}\\
E\bracket{\underset{i\in S}{\op{kthmin}}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}E\bracket{\max_{j\in T}x_j}
\end{gather}
$$

???+ note "P4707 重返现世"

    $n$ 个元素，每个元素有权值 $p_i$，令 $m=\sum_{i=1}^{n}p_i$，则每过一个单位时间有 $\frac{p_i}{m}$ 的概率获得第 $i$ 个元素。求收集任意 $k$ 种不同元素的期望用时。$1\le n,m,k\le 1000$，$n-k\le 10$。

    ??? note "题解"

        注意到对于每种元素我们只关心其首次出现的时间。令 $x_i$ 为首次收集到第 $i$ 种元素的用时（为一个随机变量），则我们求的是 $E(\op{kthmin}_{i=1}^{n} x_i$)。注意到数据范围中 $n-k$ 的值较小，而第 $k$ 小相当于第 $n-k+1$ 大，我们用 $n-k+1$ 取代原本的 $k$，则我们求的是 $E(\op{kthmax}_{i=1}^{n} x_i)$

        使用 min-max 容斥公式：（$S=[1,n]\cap\mathbb{Z}$，即全集）

        $$
        E\bracket{\underset{i\in S}{\op{kthmax}}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}E\bracket{\min_{j\in T}x_j}
        $$

        思考 $E\bracket{\min_{j\in T}x_j}$ 的期望意义：首次获得 $T$ 中任意元素的期望时间。由于在每个时刻获得 $T$ 中元素的概率为 $\frac{\sum_{i\in T}p_i}{m}$，则期望值为 $\frac{m}{\sum_{i\in T}{p_i}}$。

        代入公式有：

        $$
        E\bracket{\underset{i\in S}{\op{kthmax}}x_{i}}=\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}\frac{m}{\sum_{i\in T}{p_i}}
        $$

        注意到 $n$ 较大，枚举子集是不可接受的。发现 $\sum_{i\in T}p_i$ 像选取元素的“重量”之和，考虑使用类似背包 DP 的方式，令 $f(i,j)$ 表示做到前 $i$ 个元素，对于所有 $T$ 中元素 $p$ 值之和为 $j$ 的 $T$ 的系数（$\sum_{T\subseteq S}(-1)^{|T|-k}\binom{|T|-1}{k-1}$）之和。

        当第 $i$ 个元素不加入子集时，有 $f(i,j)\leftarrow f(i-1,j)$，但当第 $i$ 个元素选择加入时，$|T|$ 会变化。注意到组合数有递推公式 $\binom{n}{m}=\binom{n-1}{m}+\binom{n-1}{m-1}$，考虑将 $k$ 作为一维状态加入 DP。则：

        $$
        \def\red#1{\textcolor{red}{#1}}
        \begin{aligned}
        f(i,j,k)&=\sum_{T\subseteq [1,i],\sum_{x\in T}p_x=j}(-1)^{|T|-k}\binom{|T|-1}{k-1}\\
        &=\sum_{T\subseteq [1,i],\sum_{x\in T}p_x=j,\red{i\in T}}(-1)^{|T|-k}\binom{|T|-1}{k-1}+f(i-1,j,k)\\
        &=\sum_{T\subseteq [1,i],\sum_{x\in T}p_x=j,\red{i\in T}}-(-1)^{(|T|-1)-k}\bracket{\binom{(|T|-1)-1}{k-1}+\binom{(|T|-1)-1}{k-2}}+f(i-1,j,k)\\
        &=\sum_{T\subseteq [1,i],\sum_{x\in T}p_x=j,\red{i\in T}}-(-1)^{(|T|-1)-k}\binom{(|T|-1)-1}{k-1}+\sum_{T\subseteq [1,i],\sum_{x\in T}p_x=j,\red{i\in T}}(-1)^{(|T|-1)-(k-1)}\binom{(|T|-1)-1}{(k-1)-1}+f(i-1,j,k)\\
        &=\sum_{T\subseteq [1,\red{i-1}],\sum_{x\in T}p_x=j\red{-p_i}}-(-1)^{|T|-k}\binom{|T|-1}{k-1}+\sum_{T\subseteq [1,\red{i-1}],\sum_{x\in T}p_x=j\red{-p_i}}(-1)^{|T|-(k-1)}\binom{|T|-1}{(k-1)-1}+f(i-1,j,k)\\
        &=\boxed{-f(i-1,j-p_i,k)+f(i-1,j-p_i,k-1)+f(i-1,j,k)}
        \end{aligned}
        $$

        初值 $f(0,0,0)=1$，时间复杂度 $O(nmk)$。

    ??? note "参考代码"

        ```cpp
        //输出 long long 的时候用 %lld 了吗 ~~~
        //交之前改 freopen 了吗 ~~~
        //改完代码及时交了吗 ~~~
        //算了内存不会 MLE 了吗 ~~~
        //T1 卡住看 T2 了吗 ~~~
        //树边存储开两倍了吗 ~~~
        //测时间的时候关掉 sanitizer 了吗 ~~~
        #include<bits/stdc++.h>
        #define N 1005
        #define M 10005
        #define K 15
        #define mod 998244353
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
        int n,m,k;
        long long p[N],f[M][K],inv[M],ans;
        int main()
        {
            n=read(),k=n-read()+1,m=read();
            for(int i=1;i<=n;i++) p[i]=read();
            inv[0]=inv[1]=1;
            for(int i=2;i<=m;i++) inv[i]=inv[mod%i]*(mod-mod/i)%mod;
            f[0][0]=1;
            for(int i=1;i<=n;i++)
            {
                for(int j=m;j>=p[i];j--)
                {
                    for(int t=k;t>=1;t--)
                    {
                        f[j][t]=(f[j][t]+f[j-p[i]][t-1]-f[j-p[i]][t]+mod)%mod;
                    }
                }
            }
            for(int i=m;i>=0;i--) ans=(ans+f[i][k]*m%mod*inv[i]%mod)%mod;
            printf("%lld\n",ans);
            return 0;
        }
        ```