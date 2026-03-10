# 链表

## 算法思路

链表是一种数据结构，由若干个节点组成。每个节点上存储数据以及指向其他节点的指针(1)。根据每个节点上指针的数量方向可大致分类为：
{.annotate}

1. 在链表的代码实现上，考虑到 C++ 的指针用不好就会出锅，所以通常把所有节点塞进足够大的数组中，用数组下标来代替指针。

- 单向链表：每个节点指向后一个节点；
- 双向链表：每个节点指向前面和后面的节点；
- 循环链表：单向链表基础上把最后一个节点指向头节点；
- 十字链表：矩阵中每个节点指向上下左右节点等。

链表相对于顺序表的优势在于删除插入节点更高效：顺序表为 $O(n)$，链表只需 $O(1)$ 地修改两个节点之间的指针。劣势在于无法随机访问，因为链表在内存上不连续，随机访问的复杂度为 $O(n)$。

链表在 OI 中最重要的作用为邻接表（链式前向星）存图：把一个节点连出的边塞进一个链表里，加边时直接将这条边作为表头，指针指向原表头即可。

=== "加边"

    ```cpp
    void add(int x,int y)
    {
        to[++now]=y;
        nxt[now]=head[x];
        head[x]=now;
    }
    ```

=== "遍历"

    ```cpp
    for(int i=head[u];i;i=nxt[i])
    {
        int v=to[i];
        //do something ...
    }
    ```

## 例题

### [P10061 [SNOI2024] 矩阵](https://www.luogu.com.cn/problem/P10061)

#### 题目大意

给定一个 $n\times n$ 的方阵，做 $q$ 次子方阵逆时针旋转 90 度或子矩阵加操作，求最终的矩阵。$1\le n,q \le 3000$，$8 \text{s}$。

#### 题目解法

首先这题的朴素（真的朴素吗）的想法是对每一行每一列维护一个 fhq treap，子矩阵加是简单的，逆时针旋转就是把子方阵每一行从对应行的 treap 里分裂出来，再和对应列合并，列变行同理。复杂度 $O(n^2\log n)$，理论上能过，但实测 $20\text s\sim 30\text s$，常数大，不可通过。

考虑这道题的难点：子矩阵旋转。会发现子矩阵内元素之间的位置关系是不变的，只有子矩阵与外围元素之间的位置关系有变化。维护这样的相邻位置关系，启发我们使用**十字链表**，这样只需 $O(n)$ 步找到矩阵边界，修改边界上 $O(n)$ 个值了。

继续考虑。在普通二维数组中子矩阵加的维护方式是差分，这启发我们用类似的方法维护这道题的子矩阵加。对于每个元素，我们维护四个方向的相邻元素与自身的差，这个元素的真实值就是从 $(0,0)$ 到这个元素的路径上的权值和。这样子矩阵加也只需修改 $O(n)$ 个值了。并且，对于子矩阵旋转，其实我们还需维护每个元素的“旋转值”，计算这个元素真正的上方是哪里（子矩阵旋转时元素的上下左右的相邻关系也被旋转了），“旋转值”的维护实际上也是子矩阵加，也可以用一样的方法解决。

#### 参考代码

```cpp
#include<bits/stdc++.h>
#define N 3005
#define U 0
#define R 1
#define D 2
#define L 3
#define mod1 998244353
#define mod2 1000000007
#define id(x,y) ((x)*(n+2)+(y))
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
int n,q;
struct node
{
    static node data[N*N];
    long long val;
    int d;
    struct {long long val;int d,to;} nxt[4];
    int go(int dir)
    {
        dir=(dir+d)&3;
        int v=nxt[dir].to;
        data[v].val=val+nxt[dir].val;
        data[v].d=(d+nxt[dir].d)&3;
        return v;
    };
    auto& operator [](int dir)
    {
        return nxt[(dir+d)&3];
    }
};
node node::data[N*N];
node* data=node::data;
int at(int x,int y)
{
    int iter=id(0,0);
    while(x--) iter=data[iter].go(D);
    while(y--) iter=data[iter].go(R);
    return iter;
}
void rotate(int x1,int y1,int x2,int y2)
{
    int a=at(x1,y1),b=at(x2,y1),c=at(x2,y2),d=at(x1,y2),aa,bb,cc,dd;
    for(int i=1;i<=x2-x1+1;i++)
    {
        aa=data[a].go(L),
        bb=data[b].go(D),
        cc=data[c].go(R),
        dd=data[d].go(U),
        data[aa][R]={data[d].val-data[aa].val,(data[d].d-data[aa].d+4+1)&3,d},
        data[bb][U]={data[a].val-data[bb].val,(data[a].d-data[bb].d+4+1)&3,a},
        data[cc][L]={data[b].val-data[cc].val,(data[b].d-data[cc].d+4+1)&3,b},
        data[dd][D]={data[c].val-data[dd].val,(data[c].d-data[dd].d+4+1)&3,c},
        data[a][L]={data[bb].val-data[a].val,(data[bb].d-data[a].d+4+3)&3,bb},
        data[b][D]={data[cc].val-data[b].val,(data[cc].d-data[b].d+4+3)&3,cc},
        data[c][R]={data[dd].val-data[c].val,(data[dd].d-data[c].d+4+3)&3,dd},
        data[d][U]={data[aa].val-data[d].val,(data[aa].d-data[d].d+4+3)&3,aa},
        a=data[a].go(D),
        b=data[b].go(R),
        c=data[c].go(U),
        d=data[d].go(L);
    }
    return;
}
void add(int x1,int y1,int x2,int y2,long long c)
{
    int a=at(x1,y1),b=at(x2,y1),aa,bb;
    for(int i=1;i<=y2-y1+1;i++)
    {
        aa=data[a].go(U),
        bb=data[b].go(D),
        data[a][U].val-=c,data[aa][D].val+=c,
        data[b][D].val-=c,data[bb][U].val+=c,
        a=data[a].go(R),
        b=data[b].go(R);
    }
    a=at(x1,y1),b=at(x1,y2);
    for(int i=1;i<=x2-x1+1;i++)
    {
        aa=data[a].go(L),
        bb=data[b].go(R),
        data[a][L].val-=c,data[aa][R].val+=c,
        data[b][R].val-=c,data[bb][L].val+=c,
        a=data[a].go(D),
        b=data[b].go(D);
    }
    return;
}
long long a[N][N];
void show()
{
    int row=at(1,1),col;
    for(int i=1;i<=n;i++)
    {
        col=row;
        for(int j=1;j<=n;j++)
        {
            a[i][j]=data[col].val;
            col=data[col].go(R);
        }
        row=data[row].go(D);
    }
    return;
}
int main()
{
    n=read(),q=read();
    for(int i=1;i<=n;i++)
    {
        a[i][1]=(i+1);
        for(int j=2;j<=n;j++) a[i][j]=a[i][j-1]*(i+1)%mod1;
    }
    for(int i=0;i<=n+1;i++)
    {
        for(int j=0;j<=n;j++)
        {
            data[id(i,j)][R]={a[i][j+1]-a[i][j],0,id(i,j+1)};
            data[id(i,j+1)][L]={a[i][j]-a[i][j+1],0,id(i,j)};
        }
    }
    for(int i=0;i<=n;i++)
    {
        for(int j=0;j<=n+1;j++)
        {
            data[id(i,j)][D]={a[i+1][j]-a[i][j],0,id(i+1,j)};
            data[id(i+1,j)][U]={a[i][j]-a[i+1][j],0,id(i,j)};
        }
    }
    while(q--)
    {
        int op=read(),x1=read(),y1=read(),x2=read(),y2=read();
        if(op==1) rotate(x1,y1,x2,y2);
        else add(x1,y1,x2,y2,read());
    }
    show();
    long long mul=1,ans=0;
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=n;j++)
        {
            mul=mul*12345%mod2;
            ans=(ans+a[i][j]%mod2*mul%mod2)%mod2;
        }
    }
    
    printf("%lld\n",ans);
    return 0;
}
```
