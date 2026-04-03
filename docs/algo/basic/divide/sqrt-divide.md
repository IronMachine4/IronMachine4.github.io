# 根号分治

根号分治是一种优化算法。当题目有多种复杂度瓶颈不同的暴力算法时，我们尝试对数据大小进行分治，当数据大小大于某个阈值时使用其中一种暴力算法，小于时使用另一个暴力算法，最终使得复杂度起到一个均摊的作用。由于阈值的取值通常在根号处最优，所以称为根号分治。

## 鸽巢原理分治

鸽巢原理分治，类似根号分治，是通过对数据的分治，不同数据采取不同做法的优化算法。

鸽巢原理，简单来说，就是将 $n$ 个元素分成 $k$ 组，必然存在一组的元素数大于等于 $\lceil\frac{n}{k}\rceil$，一组的元素数小于等于 $\lfloor\frac{n}{k}\rfloor$。

根据这个性质，当题目的数据可以被看作 $k$ 种不同的元素时，必然有一种元素的数量小于等于 $\lfloor\frac{n}{k}\rfloor$ 个。这样我们对每种元素，设计一种复杂度和这种元素的个数有关的算法，然后对于具体的数据，取数量最少的那种元素对应的算法，整体复杂度中的 $n$ 就可以被优化为 $\frac{n}{k}$。

???+ note "[P7670 [JOI 2018 Final] 毒蛇越狱 / Snake Escaping](https://www.luogu.com.cn/problem/P7670)"

    对 $0\sim 2^n-1$ 的每个二进制数赋一个权值。$q$ 次询问，每次询问给定一个长度为 $n$ 的由 $\texttt{0},\texttt{1},\texttt{?}$ 组成的字符串，分别表示第 $i$ 位必须为 $0$ 或 $1$ 或任意，求所有满足这一限制的二进制数的权值之和。$n\le 20$，$q\le 10^6$。

    ??? note "题解"
        
        发现字符串 $\texttt{0},\texttt{1},\texttt{?}$ 中数量最少的不会超过 $\frac{n}{3}$ 个。考虑为三种字符各设计一种算法。

        当 $\texttt{?}$ 的数量最少时，显然直接暴力枚举每个 $\texttt{?}$ 填 $0$ 还是 $1$ 即可。单次询问复杂度 $O(2^{\frac{n}{3}})$。

        当 $\texttt{1}$ 的数量最少时，我们考虑如果询问的字符串中不含有 $\texttt{1}$ 怎么做。此时，我们将所有 $\texttt{?}$ 钦定为 $1$，变成一个二进制串，则答案可以看作这个二进制串所有子集的权值之和，使用高维前缀和即可在 $O(n2^n)$ 的时间复杂度内完成预处理。

        现在考虑加入 $\texttt{1}$。模仿 $\texttt{?}$ 最少时的操作，枚举每个 $\texttt{1}$ 的位置替换为 $\texttt{0}$ 或 $\texttt{?}$，进行容斥。若一共有 $m$ 个 $\texttt{1}$，其中有 $k$ 个 $\texttt{1}$ 被替换成 $\texttt{?}$，则在这种情况下，在任意一个被筛选出的二进制数中至多出现 $k$ 个 $1$，而我们的最终目标是“对应的 $n$ 个位置恰好全部都是 $1$”，则容斥系数为 $(-1)^{m-k}$。预处理 $O(n2^n)$，单次询问 $O(2^{\frac{n}{3}})$。

        当 $\texttt{0}$ 的数量最少时，同理，不再赘述。

        综上，时间复杂度为 $O(n2^n+q2^{\frac{n}{3}})$，足以通过。我们用鸽巢原理分治将复杂度指数上的 $n$ 优化为了 $\frac{n}{3}$。

    ??? note "参考代码"

        ```cpp
        #include<bits/stdc++.h>
        #define N 21
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
        inline void qgets(char* s)
        {
            char ch=getchar();
            while(!isgraph(ch)) ch=getchar();
            while(isgraph(ch)) *s=ch,s++,ch=getchar();
            return;
        }
        char a[1<<N],Q[N];
        int n,q,pre[1<<N],suf[1<<N];
        int dfs0to1(int dep,int S,int cnt1)
        {
            if(dep==n) return ((cnt1&1)?-1:1)*suf[S];
            if(Q[dep]=='1') return dfs0to1(dep+1,(S<<1)|1,cnt1);
            else if(Q[dep]=='?') return dfs0to1(dep+1,S<<1,cnt1);
            else return dfs0to1(dep+1,(S<<1)|1,cnt1+1)+dfs0to1(dep+1,S<<1,cnt1);
        }
        int dfs1to0(int dep,int S,int cnt0)
        {
            if(dep==n) return ((cnt0&1)?-1:1)*pre[S];
            if(Q[dep]=='0') return dfs1to0(dep+1,S<<1,cnt0);
            else if(Q[dep]=='?') return dfs1to0(dep+1,(S<<1)|1,cnt0);
            else return dfs1to0(dep+1,S<<1,cnt0+1)+dfs1to0(dep+1,(S<<1)|1,cnt0);
        }
        int dfs(int dep,int S)
        {
            if(dep==n) return a[S];
            if(Q[dep]=='0') return dfs(dep+1,S<<1);
            else if(Q[dep]=='1') return dfs(dep+1,(S<<1)|1);
            else return dfs(dep+1,S<<1)+dfs(dep+1,(S<<1)|1);
        }
        int main()
        {
            n=read(),q=read(),qgets(a);
            for(int i=0;i<(1<<n);i++) a[i]-='0';
            for(int i=0;i<(1<<n);i++) suf[i]=pre[i]=a[i];
            for(int i=1;i<(1<<n);i<<=1)
            {
                for(int j=0;j<(1<<n);j+=(i<<1))
                {
                    for(int k=0;k<i;k++)
                    {
                        pre[i+j+k]+=pre[j+k];
                    }
                }
            }
            for(int i=1;i<(1<<n);i<<=1)
            {
                for(int j=0;j<(1<<n);j+=(i<<1))
                {
                    for(int k=0;k<i;k++)
                    {
                        suf[j+k]+=suf[i+j+k];
                    }
                }
            }
            while(q--)
            {
                qgets(Q);
                int cnt=0,cnt0=0,cnt1=0,ans;
                for(int i=0;i<n;i++)
                {
                    if(Q[i]=='0') cnt0++;
                    else if(Q[i]=='1') cnt1++;
                    else cnt++;
                }
                if(cnt<=cnt1&&cnt<=cnt0) ans=dfs(0,0);
                else if(cnt1<=cnt&&cnt1<=cnt0) ans=dfs1to0(0,0,0);
                else ans=dfs0to1(0,0,0);
                printf("%d\n",ans);
            }
            return 0;
        }
        ```