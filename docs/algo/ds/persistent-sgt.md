# 可持久化线段树

前置知识：动态开点线段树。

## 算法思路

可持久化线段树别名主席树。

线段树的一条重要性质，就是每次操作至多修改 $O(\log n)$ 个节点。所以在记录历史版本的时候，我们不用把整棵树复制一遍，只需要复制被修改过的节点即可。这样的时间空间复杂度就是 $O(n\log n)$ 的了。

![主席树图解](https://oi-wiki.org/ds/images/persistent-seg.png "主席树图解 图源 OI-Wiki")

## 例题

### [P3834 【模板】可持久化线段树 2](https://www.luogu.com.cn/problem/P3834)

#### 题目大意

给定静态数组，长度为 $n$，$q$ 次询问，每次求一个区间 $[l,r]$ 内的第 $k$ 小。

$1\le l\le r\le n\le 2\times 10^5$，$1\le q\le2\times 10^5$，$1\le k\le r-l+1$。

#### 题目解法

下文默认你已经离散化了。

首先，如果要求整个数组的第 $k$ 小，一个选择就是权值线段树（记录值域内每个数的出现次数的线段树）。如果左儿子比 $k$ 大，答案就是在左儿子，否则 $k$ 减去左儿子大小，答案在右儿子。复杂度单次 $O(\log n)$。

现在加强问题：对于每个前缀 $[1,r]$，求前 $k$ 小。结合可持久化线段树，这也是简单的。如果我们建好了前缀 $[1,r]$ 的权值线段树，那么拓展到前缀 $[1,r+1]$ 就是一次操作，所以每个前缀都可以看作权值线段树的一个版本，用可持久化维护它。

回到原问题：求任意区间 $[l,r]$ 的前 $k$ 小。利用前缀和的思想，我们可以从 $[1,l-1]$ 和 $[1,r]$ 两个版本的信息推出 $[l,r]$ 的信息：只需将两个版本对应节点的信息相减，就是 $[l,r]$ 建出来的树对应节点的信息了。

复杂度 $O(n\log n)$。

#### 参考代码

```cpp
#include<bits/stdc++.h>
#define N 200005
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
void write(int x,char c=0)
{
    if(x<0)
        putchar_unlocked('-'),x=-x;
    if(x>9)
        write(x/10);
    putchar_unlocked(x%10+'0');
    if(c) putchar_unlocked(c);
    return;
}
int n,m,a[N],cnt,tmp[N],val[N];
int tot,rt[N],ls[N<<6],rs[N<<6],t[N<<6];
int copy(int o)
{
    ++tot;
    ls[tot]=ls[o];
    rs[tot]=rs[o];
    t[tot]=t[o];
    return tot;
}
int build(int l,int r)
{
    ++tot;
    if(l==r) return tot;
    int mid=(l+r)/2;
    ls[tot]=build(l,mid);
    rs[tot]=build(mid+1,r);
    return tot;
}
int modify(int o,int l,int r,int p)
{
    o=copy(o);
    if(l==r)
    {
        t[o]++;
        return o;
    }
    int mid=(l+r)/2;
    if(p<=mid) ls[o]=modify(ls[o],l,mid,p);
    else rs[o]=modify(rs[o],mid+1,r,p);
    t[o]=t[ls[o]]+t[rs[o]];
    return o;
}
int query(int o1,int o2,int l,int r,int k)
{
    if(l==r) return l;
    int mid=(l+r)/2,x=t[ls[o2]]-t[ls[o1]];
    if(k<=x) return query(ls[o1],ls[o2],l,mid,k);
    else return query(rs[o1],rs[o2],mid+1,r,k-x);
}
int main()
{
    n=read(),m=read();
    for(int i=1;i<=n;i++) tmp[i]=a[i]=read();
    sort(tmp+1,tmp+n+1);
    cnt=unique(tmp+1,tmp+n+1)-(tmp+1);
    for(int i=1;i<=n;i++)
    {
        int x=lower_bound(tmp+1,tmp+cnt+1,a[i])-tmp;
        val[x]=a[i];
        a[i]=x;
    }
    rt[0]=build(1,cnt);
    for(int i=1;i<=n;i++) rt[i]=modify(rt[i-1],1,cnt,a[i]);
    while(m--)
    {
        int l=read(),r=read(),k=read();
        write(val[query(rt[l-1],rt[r],1,cnt,k)],'\n');
    }
    return 0;
}
```