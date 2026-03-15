# 最小斯坦纳树

给定无向图中的指定点集，则将点集中的所有点连通的子图称作**斯坦纳树**，边权/点权和最小的斯坦纳树称作**最小斯坦纳树**(1)。
{.annotate}

1. 显然最小斯坦纳树一定为树，否则可以删除环上的任意一条边使答案更优。

如图，红色点为给定点集，红边为斯坦纳树边。图中的斯坦纳树是最小的。（图源洛谷）

![alt text](https://cdn.luogu.com.cn/upload/image_hosting/rdu06bwj.png)

普通图的最小斯坦纳树是 NP 问题，没有多项式算法。

考虑状压 DP。令 $f_{S,i}$ 表示把 $S$ 中的点连通，斯坦纳树的树根为 $i$ 的最小答案。转移分为两类：

- $i$ 的度数 $=1$：对于边 $(i,j)$，$f_{S,i}\leftarrow f_{S,j}+w(i,j)$；
- $i$ 的度数 $\ge 2$：则 $i$ 的子树把 $S$ 分为至少两个部分，$f_{S,i}\leftarrow f_{T,i}+f_{S-T,i}$。

从小到大枚举 $S$。对于第二种转移，直接枚举子集即可。对于第一种转移，$f_{S,i}\leftarrow f_{S,j}+w(i,j)$ 就是最短路中的三角形不等式，跑一遍 SPFA 或者 Dijkstra 即可。

??? note "参考代码：[P6192 【模板】最小斯坦纳树](https://www.luogu.com.cn/problem/P6192)"

    ```cpp
    #include<bits/stdc++.h>
    #define N 105
    #define K 11
    #define M 505
    #define inf 0x3f3f3f3f
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
    int n,m,k,now,head[N],to[M*2],nxt[M*2],w[M*2],f[N][1<<K],p[N];
    bool vis[N];
    queue<int> q;
    void add(int x,int y,int z)
    {
        to[++now]=y;
        nxt[now]=head[x];
        head[x]=now;
        w[now]=z;
        return;
    }
    void spfa(int S)
    {
        while(q.size())
        {
            int x=q.front();
            q.pop();
            vis[x]=false;
            for(int i=head[x];i;i=nxt[i])
            {
                int v=to[i];
                if(f[v][S]>f[x][S]+w[i])
                {
                    f[v][S]=f[x][S]+w[i];
                    if(!vis[v]) q.push(v),vis[v]=true;
                }
            }
        }
    }
    int main()
    {
        n=read(),m=read(),k=read();
        for(int i=1;i<=m;i++)
        {
            int u=read(),v=read(),w=read();
            add(u,v,w);
            add(v,u,w);
        }
        memset(f,0x3f,sizeof(f));
        for(int i=1;i<=k;i++)
        {
            p[i]=read();
            f[p[i]][1<<(i-1)]=0;
        }
        for(int S=0;S<(1<<k);S++)
        {
            for(int i=1;i<=n;i++)
            {
                for(int T=S&(S-1);T;T=S&(T-1))
                {
                    f[i][S]=min(f[i][S],f[i][T]+f[i][S-T]);
                }
                if(f[i][S]<inf) q.push(i),vis[i]=true;
            }
            spfa(S);
        }
        printf("%d\n",f[p[1]][(1<<k)-1]);
        return 0;
    }
    ```