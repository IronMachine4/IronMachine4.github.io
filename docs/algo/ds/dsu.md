# 并查集

并查集是一种维护集合合并的数据结构，支持高效地：

- 合并两个集合
- 查询元素属于哪个集合

其基本原理为给每个集合钦定一个“代表元”，给集合内的每个元素钦定一个父元素，使整个集合变成一个根为代表元的树形结构（代表元的的父亲为自己）。查询元素所在集合就是从该元素跳父亲直到代表元，即父亲等于自身，合并两个集合只需将其中一个集合的代表元的父亲设为另一个集合代表元即可。

朴素的并查集复杂度与集合的树高有关，最坏为 $O(n)$，所以我们要使用**按秩合并**和**路径压缩**两种优化方式优化复杂度。

- **路径压缩**：每次查询元素所在集合的代表元的时候，把路径上的所有点的父亲直接设为代表元。正确性显然。只使用路径压缩的并查集复杂度最坏为 $O(\log n)$。路径压缩是“破坏性”比较大的算法，在特殊题目（可持久化并查集，线段树分治等）不能使用。
```cpp
int find(int x)
{
    if(x==fa[x]) return x;
    return fa[x]=find(fa[x]);
}
```

- **按秩合并**：每次合并两个集合的时候，总是把元素少/树高矮的集合代表元做另一个集合代表元的儿子。考虑把小树塞大树下面会导致小树中每个元素到代表元的距离 +1，但是树的大小却变成了原来的 $2$ 倍，也就是说，任意一个元素只会经历 $O(\log n)$ 次距离增加，所以按秩合并的树高为 $O(\log n)$ 的，单次操作复杂度为 $O(\log n)$。
```cpp
void merge(int x,int y)
{
    x=find(x),y=find(y);
    if(sz[x]>sz[y]) swap(x,y);
    fa[x]=y;
    sz[y]+=sz[x];
}
```

如果两者都使用了，可以证明单次时间复杂度平均为 $O(\alpha(n))$ 的，其中 $\alpha(n)$ 为增长极慢的函数。不卡常的话直接将并查集视为 $O(1)$ 即可。

??? note "[CF571D Campus](https://codeforces.com/problemset/problem/571/D)"

    给定一个长度为 $n$，初值为 $0$ 的数组，以及 A 类，B 类两种下标集合，初始每类各有 $n$ 个集合，分别包含下标 $1\sim n$。维护 $m$ 次以下操作：

    - 合并同类的两个集合
    - 将某个 A 类集合中的下标对应数组上的值加该集合大小
    - 将某个 B 类集合中的下标对应数组上的值清零
    - 查询数组某个位置上的值。

    $1\le n,m \le 5\times 10^5$。

    ??? note "题目解法"

        首先，看到集合合并，自然想到并查集，修改操作直接在树根打标记，影响整棵子树。但直接这么做是错误的。比如，在 $u$ 树上你打了一个清零标记，现在 $u$ 和 $v$ 合并，$u$ 做 $v$ 的父亲，这样 $u$ 节点上的标记就影响到了这个 $v$ 子树，但显然 $v$ 不当被影响。

        修改时在树根打标记肯定是对的了，想一想如何纠正这个错误。会发现这与“修改”和“合并”两个操作的先后顺序有关。因此，在合并和打标记时，我们都在相应位置记录操作的时间。对于在 $\operatorname{clr}_u$ 时对 $u$ 的清零操作和 $u$ 在 $t_u$ 时当上 $v$ 的父亲两件事，只有 $\operatorname{clr}_u>t_u$，$\operatorname{clr}_u$ 才对子树 $v$ 有效。

        回到题目。对 A,B 类集合分别建并查集。为了求某个位置的值，我们需要知道这个位置最后一次清零在什么时候，在这之后加操作的总影响。最后一次清零好求，只需在并查集 B 上找到该位置到根路径上，有效清零操作的时刻最大值。加操作总影响稍微难求，我们需在 A 并查集上每个节点建一个 vector，进行加操作的修改时，在树根的 vector 内插入修改的时间和加值的大小，并前缀和。询问时就对于询问节点到根路径上每个 vector，二分求出修改时间大于最后清零时间的加操作之和（上文的前缀和这时候用）。

        可以看出，并查集**不能**路径压缩。只按秩合并的并查集树高为 $O(\log n)$，再算上二分，总复杂度 $O(m\log^2 n)$。实际上没法卡满，可轻松通过。

    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 500005
        #define MP make_pair
        #define PB push_back
        #define fir first
        #define sec second
        #define BK back()
        #define LB(a,b) (lower_bound((a).begin(),(a).end(),(b))-(a).begin())
        #define SUF(a,b) ((a).back().second-(a)[b-1].second)
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
        inline char qchar()
        {
            char ch=getchar();
            while(!isgraph(ch)) ch=getchar();
            return ch;
        }
        int n,m,fa[N],fb[N],sza[N],szb[N],ta[N],tb[N],clr[N];
        vector<pair<int,long long>> add[N];
        int finda(int x)
        {
            if(x==fa[x]) return x;
            return finda(fa[x]);
        }
        int findb(int x)
        {
            if(x==fb[x]) return x;
            return findb(fb[x]);
        }
        void mergea(int x,int y,int t)
        {
            x=finda(x),y=finda(y);
            if(sza[x]>sza[y]) swap(x,y);
            ta[x]=t,fa[x]=y,sza[y]+=sza[x];
            return;
        }
        void mergeb(int x,int y,int t)
        {
            x=findb(x),y=findb(y);
            if(szb[x]>szb[y]) swap(x,y);
            tb[x]=t,fb[x]=y,szb[y]+=szb[x];
            return;
        }
        void Add(int x,int t)
        {
            x=finda(x);
            add[x].PB(MP(t,add[x].BK.sec+sza[x]));
            return;
        }
        void Clr(int x,int t)
        {
            x=findb(x);
            clr[x]=t;
            return;
        }
        long long query(int x)
        {
            int fx=x,t0=clr[x];
            while(fx!=fb[fx])
            {
                if(tb[fx]<clr[fb[fx]]) t0=max(t0,clr[fb[fx]]);
                fx=fb[fx];
            }
            long long p=LB(add[x],MP(t0,0ll)),ans=SUF(add[x],p);
            fx=x;
            while(fx!=fa[fx])
            {
                p=LB(add[fa[fx]],MP(max(t0,ta[fx]),0ll));
                ans+=SUF(add[fa[fx]],p);
                fx=fa[fx];
            }
            return ans;
        }
        int main()
        {
            n=read(),m=read();
            for(int i=1;i<=n;i++)
            {
                fa[i]=fb[i]=i;
                sza[i]=szb[i]=1;
                add[i].PB(MP(-1,0));
            }
            char ch;
            int x,y;
            for(int i=1;i<=m;i++)
            {
                ch=qchar();
                if(ch=='U') x=read(),y=read(),mergea(x,y,i);
                else if(ch=='M') x=read(),y=read(),mergeb(x,y,i);
                else if(ch=='A') x=read(),Add(x,i);
                else if(ch=='Z') x=read(),Clr(x,i);
                else x=read(),printf("%lld\n",query(x));
            }
            return 0;
        }
        ```