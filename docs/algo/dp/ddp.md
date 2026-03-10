# 动态 DP

动态 DP 问题是猫锟在 WC2018 讲的黑科技，一般用来解决树上的带有点权（边权）修改操作的 DP 问题。

首先，设原树的 DP 状态为 $f_{u,i}$，我们需要对原树进行重链剖分，并处理出新的状态 $g_{u,i}$，表示不考虑 $u$ 的重儿子 $\text{son}_u$ 的 DP 值，并把原递推式改为

$$
\begin{bmatrix}
f_{u,0}\\
f_{u,1}\\
\dots\\
f_{u,i}
\end{bmatrix}
=M_{u}\otimes
\begin{bmatrix}
f_{\text{son}_u,0}\\
f_{\text{son}_u,1}\\
\dots\\
f_{\text{son}_u,i}
\end{bmatrix}
$$

的形式，把每个节点的状态压进一个矩阵，节点 $u$ 的状态 $f_u$ 可以由重儿子的状态 $f_{\text{son}_u}$ 乘上只和轻儿子有关的矩阵（即只与 $g_u$ 有关的矩阵 $M_u$）得到，那么 $f_u$ 的值就是 $M_u\otimes M_{\text{son}_u}\otimes\dots\otimes M_{\text{end}_u}\otimes\left[\begin{smallmatrix}0\\0\\\dots\\0\end{smallmatrix}\right]$（$\text{end}_u$ 表示 $u$ 所在的重链的结尾，即叶子节点），对点权/边权的修改只会影响到 $O(\log n)$ 个点的 $M$ 矩阵。可以由数据结构维护（比如对重链建线段树，或者全局平衡二叉树）。

注意：上文的矩阵乘法在绝大多数情况下，都是指 $(\max/\min,+)$ 的广义矩阵乘法。

???+ note "广义矩阵乘法"

    通常，矩阵乘法 $A\times B=C$ 都是指：
    
    $$
    C_{i,j}=\sum_{k}A_{i,k}B_{k,j}
    $$

    的矩阵乘法。我们不妨进行拓展：定义 $(\oplus,\otimes)$ 的矩阵乘法 $A\times B=C$ 为：

    $$
    C_{i,j}=\bigoplus_{k}A_{i,k}\otimes B_{k,j}
    $$

    矩阵乘法需要满足结合律但不满足交换律。交换律有没有都无所谓，但如何保证有结合律呢？

    直接给出结论：满足以下条件的广义矩阵乘法有结合律：

    1. 对于任意的 $x,y,z$，有 $x\otimes(y\oplus z)=(x\otimes y)\oplus(x\otimes z)$，$(x\oplus y)\otimes z=(x\otimes z)\oplus(y\otimes z)$（即 $\otimes$ 对 $\oplus$ 有左分配律和右分配律）;
    2. 对于任意的 $x,y$，有 $x\oplus y=y\oplus x$（即 $\oplus$ 满足结合律）。

    证明在[此](https://www.zhihu.com/question/643809025)。信竞中最常用的广义矩阵乘法是 $(\min/\max,+)$。

下面结合例题来讲：

???+ note "[P4719 【模板】动态 DP](https://www.luogu.com.cn/problem/P4719) 和[加强版](https://www.luogu.com.cn/problem/P4751)"

    给定一颗大小为 $n$ 的带点权树。修改 $q$ 次单点点权，每次修改后求最大带权独立集大小。

    ???+ note "题目解法"

        首先，不带修的 DP 是容易的：令 $f_{u,0/1}$ 表示 $u$ 被选/不被选时 $u$ 子树的独立集大小。有：

        $$
        \begin{aligned}
        f_{u,0}=&\sum_{v\in\text{son}_u}\max(f_{v,0},f_{v,1})\\
        f_{u,1}=&\sum_{v\in\text{son}}f_{v,0}+w_{u}
        \end{aligned}
        $$

        现在带修，考虑动态 DP。令 $g_{u,0}$ 表示不考虑 $u$ 重儿子时 $u$ 选/不选时 $u$ 子树的独立集大小。有：

        $$
        \begin{aligned}
        f_{u,0}=&g_{u,0}+\max(f_{\text{son}_u,0},f_{\text{son}_u,1})\\
        f_{u,1}=&g_{u,1}+f_{\text{son}_u,0}
        \end{aligned}
        $$

        写成矩阵形式就是

        $$
        \begin{bmatrix} f_{u,0}\\f_{u,1} \end{bmatrix}=
        \begin{bmatrix}
        g_{u,0}&g_{u,0}\\
        g_{u,1}&-\infty
        \end{bmatrix}
        \begin{bmatrix} f_{\text{son}_u,0}\\ f_{\text{son}_u,1} \end{bmatrix}
        $$

        令 $M_u=\left[\begin{smallmatrix}g_{u,0}&g_{u,0}\\g_{u,1}&-\infty\end{smallmatrix}\right]$，则 $f_u=M_u M_{\text{son}_u}M_{\text{son}_{\text{son}_u}}\cdots M_{\text{end}_u}\left[\begin{smallmatrix}0\\0\end{smallmatrix}\right]$。

        - 如果用线段树维护矩阵乘法，则 $u$ 的点权被修改，会使 $M_u$ 和 $M_{\text{fa}_{\text{top}_u}}$ 被改变。然后继续递归到 $\text{fa}_{\text{top}_u}$ 修改。需进行 $\log n$ 次单点修改，时间复杂度 $O(n\log^2n)$，可通过 $1\le n,q \le 10^5$ 的弱化版。

            ??? note "参考代码"

                ```cpp
                #include<bits/stdc++.h>
                #define N 100005
                #define inf 0x3f3f3f3f
                #define max(a,b) ((a)>(b)?a:b)
                #define min(a,b) ((a)<(b)?a:b)
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
                struct mat
                {
                    int a11,a12,a21,a22;
                    mat operator *(const mat& b)const
                    {
                        return mat{
                            max(a11+b.a11,a12+b.a21),
                            max(a11+b.a12,a12+b.a22),
                            max(a21+b.a11,a22+b.a21),
                            max(a21+b.a12,a22+b.a22)
                        };
                    }
                }t[N*4],B[N];

                int n,m,now,head[N],to[N*2],nxt[N*2],w[N];
                void add(int x,int y)
                {
                    to[++now]=y;
                    nxt[now]=head[x];
                    head[x]=now;
                }

                int sz[N],fa[N],son[N],f[N][2];
                void dfs1(int x,int father)
                {
                    sz[x]=1;
                    fa[x]=father;
                    f[x][1]=w[x];
                    for(int i=head[x];i;i=nxt[i])
                    {
                        int v=to[i];
                        if(v==fa[x]) continue;
                        dfs1(v,x);
                        if(sz[v]>sz[son[x]]) son[x]=v;
                        sz[x]+=sz[v];
                        f[x][0]+=max(f[v][0],f[v][1]);
                        f[x][1]+=f[v][0];
                    }
                    return;
                }

                int tot,id[N],pos[N],top[N],ed[N],g[N][2];
                int dfs2(int x,int t)
                {
                    id[x]=++tot;
                    pos[tot]=x;
                    top[x]=t;
                    if(son[x]) ed[x]=dfs2(son[x],t);
                    else ed[x]=x;
                    g[x][1]=w[x];
                    for(int i=head[x];i;i=nxt[i])
                    {
                        int v=to[i];
                        if(v==fa[x]||v==son[x]) continue;
                        dfs2(v,v);
                        g[x][0]+=max(f[v][0],f[v][1]);
                        g[x][1]+=f[v][0];
                    }
                    B[x].a11=B[x].a12=g[x][0];
                    B[x].a21=g[x][1],B[x].a22=-inf;
                    return ed[x];
                }

                inline void pushup(int o) {t[o]=t[o*2]*t[o*2+1];}
                void build(int o,int l,int r)
                {
                    if(l==r)
                    {
                        t[o]=B[pos[l]];
                        return;
                    }
                    int mid=(l+r)/2;
                    build(o*2,l,mid);
                    build(o*2+1,mid+1,r);
                    pushup(o);
                    return;
                }
                void modify(int o,int l,int r,int p)
                {
                    if(l==r)
                    {
                        t[o]=B[pos[l]];
                        return;
                    }
                    int mid=(l+r)/2;
                    if(p<=mid) modify(o*2,l,mid,p);
                    else modify(o*2+1,mid+1,r,p);
                    pushup(o);
                    return;
                }
                mat query(int o,int l,int r,int L,int R)
                {
                    if(L<=l&&r<=R) return t[o];
                    int mid=(l+r)/2;
                    mat res={0,-inf,-inf,0};
                    if(L<=mid) res=res*query(o*2,l,mid,L,R);
                    if(R>mid) res=res*query(o*2+1,mid+1,r,L,R);
                    return res;
                }

                void update(int x,int c)
                {
                    B[x].a21+=c-w[x];
                    w[x]=c;
                    while(x)
                    {
                        mat lst=query(1,1,n,id[top[x]],id[ed[x]]);
                        modify(1,1,n,id[x]);
                        mat now=query(1,1,n,id[top[x]],id[ed[x]]);
                        x=fa[top[x]];
                        B[x].a11+=max(now.a11,now.a21)-max(lst.a11,lst.a21);
                        B[x].a12=B[x].a11;
                        B[x].a21+=now.a11-lst.a11;
                    }
                    return;
                }

                int main()
                {
                    n=read(),m=read();
                    for(int i=1;i<=n;i++) w[i]=read();
                    for(int i=1;i<n;i++)
                    {
                        int u=read(),v=read();
                        add(u,v);
                        add(v,u);
                    }
                    dfs1(1,0);
                    dfs2(1,1);
                    build(1,1,n);
                    while(m--)
                    {
                        int p=read(),c=read();
                        update(p,c);
                        mat ans=query(1,1,n,1,id[ed[1]]);
                        printf("%d\n",max(ans.a11,ans.a21));
                    }
                    return 0;
                }
                ```
        - 如果用全局平衡二叉树，则修改 $u$ 的权值会使 $M_u$ 变化，我们在 $u$ 所在二叉树中一路往根跳并 pushup，即可维护好这颗二叉树的信息。然后跳 $(\text{rt}_u,\text{fa}_{\text{rt}_u})$ 这条轻边，修改 $M_{\text{fa}_{\text{rt}_u}}$，并递归处理。时间复杂度 $O(n\log n)$，能过 $1\le n\le 10^6$ 的强化版。

            ??? note "参考代码"

                ```cpp
                #include<bits/stdc++.h>
                #define N 1000005
                #define inf 0x3f3f3f3f
                #define max(a,b) ((a)>(b)?a:b)
                #define min(a,b) ((a)<(b)?a:b)
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
                struct mat
                {
                    int a11,a12,a21,a22;
                    mat():a12(-inf),a21(-inf){}
                    mat(int x,int y,int z,int w):a11(x),a12(y),a21(z),a22(w){}
                    mat operator *(const mat& b)const
                    {
                        return mat{
                            max(a11+b.a11,a12+b.a21),
                            max(a11+b.a12,a12+b.a22),
                            max(a21+b.a11,a22+b.a21),
                            max(a21+b.a12,a22+b.a22)
                        };
                    }
                }t[N],B[N];

                int n,m,now,head[N],to[N*2],nxt[N*2],w[N];
                void add(int x,int y)
                {
                    to[++now]=y;
                    nxt[now]=head[x];
                    head[x]=now;
                    return;
                }

                int fa[N],sz[N],son[N],f[N][2];
                void dfs1(int x,int father)
                {
                    fa[x]=father;
                    sz[x]=1;
                    f[x][1]=w[x];
                    for(int i=head[x];i;i=nxt[i])
                    {
                        int v=to[i];
                        if(v==fa[x]) continue;
                        dfs1(v,x);
                        f[x][0]+=max(f[v][0],f[v][1]);
                        f[x][1]+=f[v][0];
                        sz[x]+=sz[v];
                        if(sz[v]>sz[son[x]]) son[x]=v;
                    }
                    return;
                }

                int tot,id[N],pos[N],top[N],ed[N],g[N][2];
                void dfs2(int x,int tp)
                {
                    top[x]=tp;
                    id[x]=++tot;
                    pos[tot]=x;
                    ed[x]=x;
                    if(son[x]) dfs2(son[x],tp),ed[x]=ed[son[x]];
                    g[x][1]=w[x];
                    for(int i=head[x];i;i=nxt[i])
                    {
                        int v=to[i];
                        if(v==fa[x]||v==son[x]) continue;
                        dfs2(v,v);
                        g[x][0]+=max(f[v][0],f[v][1]);
                        g[x][1]+=f[v][0];
                    }
                    B[x].a11=B[x].a12=g[x][0];
                    B[x].a21=g[x][1],B[x].a22=-inf;
                    return;
                }

                int rt,ls[N],rs[N],up[N];
                void pushup(int o)
                {
                    t[o]=t[ls[o]]*B[o]*t[rs[o]];
                }
                int cbuild(int l,int r)
                {
                    int sum=0,tmp=0;
                    for(int i=l;i<=r;i++) sum+=sz[pos[i]]-sz[son[pos[i]]];
                    for(int i=l;i<=r;i++)
                    {
                        int x=pos[i];
                        tmp+=sz[x]-sz[son[x]];
                        if(tmp*2>=sum)
                        {
                            if(i-1>=l) ls[x]=cbuild(l,i-1),up[ls[x]]=x;
                            if(i+1<=r) rs[x]=cbuild(i+1,r),up[rs[x]]=x;
                            pushup(x);
                            return x;
                        }
                    }
                    return 0;
                }
                int build(int x)
                {
                    for(int i=id[x];i<=id[ed[x]];i++)
                    {
                        int u=pos[i];
                        for(int j=head[u];j;j=nxt[j])
                        {
                            int v=to[j];
                            if(v==fa[u]||v==son[u]) continue;
                            up[build(v)]=u;
                        }
                    }
                    return cbuild(id[x],id[ed[x]]);
                }

                void modify(int x,int c)
                {
                    B[x].a21+=c-w[x];
                    w[x]=c;
                    while(x)
                    {
                        int fa=up[x];
                        if(fa&&ls[fa]!=x&&rs[fa]!=x)
                        {
                            mat lst=t[x];
                            pushup(x);
                            mat now=t[x];
                            int flst[2]={max(lst.a11,lst.a12),max(lst.a21,lst.a22)};
                            int fnow[2]={max(now.a11,now.a12),max(now.a21,now.a22)};
                            B[fa].a11+=max(fnow[0],fnow[1])-max(flst[0],flst[1]);
                            B[fa].a12=B[fa].a11;
                            B[fa].a21+=fnow[0]-flst[0];
                        }
                        else pushup(x);
                        x=up[x];
                    }
                    return;
                }

                int lstans;
                int main()
                {
                    n=read(),m=read();
                    for(int i=1;i<=n;i++) w[i]=read();
                    for(int i=1;i<n;i++)
                    {
                        int u=read(),v=read();
                        add(u,v);
                        add(v,u);
                    }
                    dfs1(1,0);
                    dfs2(1,1);
                    rt=build(1);
                    while(m--)
                    {
                        int p=read()^lstans,c=read();
                        modify(p,c);
                        printf("%d\n",lstans=max(t[rt].a11,t[rt].a21));
                    }
                    return 0;
                }
                ```