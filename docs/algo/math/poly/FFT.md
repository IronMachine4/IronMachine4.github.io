# 快速傅里叶变换

快速傅里叶变换，是一种在 $O(n\log n)$ 的时间复杂度内求出两个多项式的卷积的算法。

## 单位根$\def\bracket#1{\left(#1\right)}$

### 单位根的定义

我们称方程 $x^k=1$ 的解为 **$k$ 次单位根**。

将 $k$ 次单位根画在复平面上，会发现这些复数对应的向量把单位元等分成了 $k$ 份。如图，这是 $4$ 次单位根，即 $x^4=1$ 的所有解（$x_1=1,x_2=i,x_3=-1,x_4=-i$）：

![](/images/4 次单位根.png)

这是 $3$ 次单位根，对应 $x^3=1$ 的三个解

![](/images/3 次单位根.png)

我们将非 $1$ 的辐角最小的 $n$ 次单位根记作 $\omega_n$。根据一点点复数和三角函数的知识，我们知道 $\omega_n^k$ 就是从 $1$ 逆时针旋转得到的第 $k$ 个单位根，如上图所示。

如何计算单位根的实部和虚部？我们有公式：

$$
\omega_{n}^{k}=cos\left(\frac{2k\pi}{n}\right)+i\sin\left(\frac{2k\pi}{n}\right)
$$

根据三角函数易证。

### 单位根的性质

单位根的性质有很多，但本文只会用到以下性质：

1.  $\omega_{rn}^{rk}=\omega_{n}^{k}$；

    ??? abstract "证明"
       
        $$
        \omega_{rn}^{rk}=cos\bracket{\frac{2rk\pi}{rn}}+i\sin\bracket{\frac{2rk\pi}{rn}}=cos\bracket{\frac{2k\pi}{n}}+i\sin\bracket{\frac{2k\pi}{n}}=\omega_{n}^{k}
        $$

2.  $\omega_{n}^{k+\frac{n}{2}}=-\omega_{n}^{k}$

    ??? abstract "证明"

        将指数加上 $\frac{n}{2}$ 相当于把单位根旋转了半圈，那自然是 $-\omega_{n}^{k}$。

3.  $\omega_{n}^{k+n}=\omega_{n}^{k}$

    ??? abstract "证明"

        同上，在指数加 $n$ 相当于单位根旋转一整圈，值不变。

## 快速傅里叶变换（FFT）

观察到：对于多项式 $f$，我们有两种表达方式：一种是最常见的系数表示法，对于 $f(x)=\sum_{i=0}^{n}a_{i}x^i$，系数表示法为 $(a_0,a_1,a_2,\dots,a_n)$。但以这种表示法去做多项式乘法显然没有前途。

还有一种表示法：点值表示法。对于 $n$ 次多项式，我们只要知道其在 $n$ 个点处的取值，就能唯一确定这个多项式。即 $f$ 的点值表示法为 $((x_1,f(x_1)),(x_2,f(x_2)),\dots,(x_n,f(x_n)))$。而且点值表示法有一个好处：对于已知点值表示法（且所选的点相同）的多项式 $f$ 和 $g$，他们的乘积的点值表示法是好求的，为 $((x_1,f(x_1)g(x_1)),(x_2,f(x_2)g(x_2)),\dots,(x_n,f(x_n)g(x_n)))$。

但仍然有问题：如果随便选 $n$ 个代入值的话，求点值表示法的复杂度仍然是 $O(n^2)$ 的。所以，我们尝试将 $\omega_{n}^{0}\sim \omega_{n}^{n-1}$ 代入。

假设我们要求 $f=a_0x^0+a_1x^1+\dots+a_{n-1}x^{n-1}$ （下文设 $n=8$）在 $n$ 次单位根处的点值表示法。我们先按次数的奇偶，将 $f$ 分为两部分：

$$
\begin{aligned}
f(x)&=a_0x^0+a_1x^1+a_2x^2+a_3x^3+a_4x^4+a_5x^5+a_6x^6+a_7x^7\\
&=(a_0x^0+a_2x^2+a_4x^4+a_6x^6)+(a_1x^1+a_3x^3+a_5x^5+a_7x^7)\\
\end{aligned}
$$

我们设两个次数为 $\frac{n}{2}$ 的新多项式：

$$
\begin{aligned}
g(x)&=\sum_{i=0}^{\frac{n}{2}}a_{2i}x^i=a_0x^0+a_2x^1+a_4x^2+a_6x^3\\
h(x)&=\sum_{i=0}^{\frac{n}{2}}a_{2i+1}x^i=a_1x^0+a_3x^1+a_5x^2+a_7x^3
\end{aligned}
$$

那么我们有

$$
f(x)=g(x^2)+xh(x^2)
$$

考虑依次将 $\omega_{n}^{k}$ 代入 $x$，有：

$$
\begin{aligned}
f(\omega_{n}^{k})&=g(\omega_{n}^{2k})+\omega_{n}^{k}h(\omega_{n}^{2k})\\
&=g(\omega_{\frac{n}{2}}^{k})+\omega_{n}^{k}h(\omega_{\frac{n}{2}}^{k})\\
\end{aligned}
$$

根据单位根的性质，我们还有：

$$
\begin{aligned}
f(\omega_{n}^{k+\frac{n}{2}})&=g(\omega_{\frac{n}{2}}^{k+\frac{n}{2}})+\omega_{n}^{k+\frac{n}{2}}h(\omega_{\frac{n}{2}}^{k+\frac{n}{2}})\\
&=g(\omega_{\frac{n}{2}}^{k})-\omega_{n}^{k}h(\omega_{\frac{n}{2}}^{k})\\
\end{aligned}
$$

所以，我们只要能求出 $g$ 和 $h$ 在 $\omega_{\frac{n}{2}}^{0}\sim \omega_{\frac{n}{2}}^{\frac{n}{2}-1}$ 处的取值即可。这与原问题同构，所以我们可以递归处理。

为了每次递归时能恰好把序列分为长度相等的两部分，我们的 $n$ 必须是 $2$ 的某个次幂。如果不是，我们要强制在高位补零。

复杂度 $O(n\log n)$。

## 快速傅里叶逆变换（IFFT）

现在我们得到了求点值表示法的方法，我们还需要将得到的点值表示法转化为系数表示法。

考虑原多项式 $f(x)=\sum_{i=0}^{n-1}a_ix^i$ 和点值表示法 $y_i=f(\omega_{n}^{i})$，我们构造一个这样的多项式

$$
A(x)=\sum_{i=0}^{n-1}y_ix^i
$$

设 $A$ 在 $b_i=\omega_{n}^{-i}$ 处的点值表示法为 $\{A(b_0),A(b_1),\dots,A(b_{n-1})\}$。

我们做一些变换：

$$
\begin{aligned}
A(b_k)&=\sum_{i=0}^{n-1}y_i(b_k)^i\\
&=\sum_{i=0}^{n-1}f(\omega_{n}^{i})\omega_{n}^{-ik}\\
&=\sum_{i=0}^{n-1}\omega_{n}^{-ik}\sum_{j=0}^{n-1}a_j(\omega_{n}^{i})^{j}\\
&=\sum_{i=0}^{n-1}\sum_{j=0}^{n-1}a_j\omega_{n}^{i(j-k)}\\
&=\sum_{j=0}^{n-1}a_j\sum_{i=0}^{n-1}(\omega_{n}^{j-k})^i
\end{aligned}
$$

令 $S(\omega_{n}^{k})=\sum_{i=0}^{n-1}(\omega_{n}^{k})^i$，则根据等比数列求和公式：

- 当 $k=0$ 时 $S(\omega_{n}^{k})=n$
- 当 $k\neq 0$ 时
  
    $$
    S(\omega_{n}^{k})=\frac{\omega_{n}^{nk}-1}{\omega_{n}^{k}-1}=\frac{\omega_{1}^{k}-1}{\omega_{n}^{k}-1}=0
    $$

所以：

$$
\begin{aligned}
A(b_k)=na_k
\end{aligned}
$$

综上，我们把 FFT 中的所有单位根取倒数，对 ${y_0,y_1,\dots,y_{n-1}}$ 跑 FFT，然后除以 $n$ 就可以得到 $f(x)$ 的系数表示。

## 递归改迭代

FFT 的递归常数较大。我们考虑改成迭代形式。

我们尝试把递归的顺序倒过来，从叶子节点开始往上推。每次递归时，我们有一个把偶数质数扔前面，奇数质数扔后面的操作。设最开始的多项式系数为 $a_0\sim a_{n-1}$，则每个叶子结点的系数是多少呢？

$$
\begin{aligned}
&\{a_0,a_1,a_2,a_3,a_4,a_5,a_6,a_7\}\\
&\{a_0,a_2,a_4,a_6\}\{a_1,a_3,a_5,a_7\}\\
&\{a_0,a_4\}\{a_2,a_6\}\{a_1,a_5\}\{a_3,a_7\}\\
&\{a_0\}\{a_4\}\{a_2\}\{a_6\}\{a_1\}\{a_5\}\{a_3\}\{a_7\}
\end{aligned}
$$

观察到，从左到右第 $i$ 个叶子结点的多项式系数，等于原多项式的第 “$i$ 在 $\log_2(n)$ 位二进制下的左右翻转”个系数。

设 $i$ 的翻转为 $\text{rev}_i$，则我们有：

```cpp
rev[i]=(rev[i>>1]>>1)|((i&1)<<(bit-1));//2^bit=n
```

知道了叶子节点的系数，我们就可以以一个比较小的常数从下往上递推了。

## 参考代码

???+ note "[P3803 【模板】多项式乘法（FFT）](https://www.luogu.com.cn/problem/P3803)"

    === "递归版"
    
        ```cpp
        #include<bits/stdc++.h>
        #define N 4000005
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
        //自定义复数类
        struct Complex
        {
            double re,im;
            Complex(double a=0,double b=0):re(a),im(b){}
            Complex operator +(Complex b) {return Complex(re+b.re,im+b.im);}
            Complex operator -(Complex b) {return Complex(re-b.re,im-b.im);}
            Complex operator *(Complex b) {return Complex(re*b.re-im*b.im,im*b.re+re*b.im);}
        }tmp[N];
        constexpr double PI=acos(-1.0);
        //FFT 递归实现
        void FFT(Complex* f,int n,int op)
        {
            if(n==1) return;
            for(int i=0;i<n;i++) tmp[i]=f[i];
            for(int i=0;i<n;i++)
            {
                if(i&1) f[n/2+i/2]=tmp[i];
                else f[i/2]=tmp[i];
            }
            Complex *g=f,*h=f+n/2;
            FFT(g,n/2,op);
            FFT(h,n/2,op);
            Complex cur(1,0),omega(cos(2*PI/n),sin(2*PI*op/n));
            for(int i=0;i<n/2;i++)
            {
                Complex b=cur*h[i];
                tmp[i]=g[i]+b;
                tmp[n/2+i]=g[i]-b;
                cur=cur*omega;
            }
            for(int i=0;i<n;i++) f[i]=tmp[i];
            return;
        }
        int n,m,lim;
        Complex a[N],b[N];
        int main()
        {
            n=read(),m=read();
            for(int i=0;i<=n;i++) a[i].re=read();
            for(int i=0;i<=m;i++) b[i].re=read();
            lim=1;
            while(lim<=n+m) lim*=2;
            FFT(a,lim,1);
            FFT(b,lim,1);
            for(int i=0;i<=lim;i++) a[i]=a[i]*b[i];
            FFT(a,lim,-1);
            for(int i=0;i<=n+m;i++) printf("%d ",int(a[i].re/lim+0.5));
            return 0;
        }
        ```
    
    === "迭代版"

        ```cpp
        #include<bits/stdc++.h>
        #define N 4000005
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
        //自定义复数类
        struct Complex
        {
            double re,im;
            Complex(double a=0,double b=0):re(a),im(b){}
            Complex operator +(Complex b) {return Complex(re+b.re,im+b.im);}
            Complex operator -(Complex b) {return Complex(re-b.re,im-b.im);}
            Complex operator *(Complex b) {return Complex(re*b.re-im*b.im,im*b.re+re*b.im);}
        }tmp[N];
        int rev[N],bit;
        constexpr double PI=acos(-1.0);
        void FFT(Complex* f,int n,int op)
        {
            for(int i=0;i<n;i++) rev[i]=(rev[i>>1]>>1)|((i&1)<<(bit-1));
            for(int i=0;i<n;i++) if(i<rev[i]) swap(f[i],f[rev[i]]);
            for(int len=2;len<=n;len<<=1)
            {
                Complex pw(cos(2*PI/len),sin(2*PI*op/len));
                for(int l=0,mid;l<n;l+=len)
                {
                    Complex cur(1,0);
                    mid=l+len/2;
                    for(int k=l;k<mid;k++)
                    {
                        Complex x=f[k],y=f[k+len/2];
                        f[k]=x+cur*y;
                        f[k+len/2]=x-cur*y;
                        cur=cur*pw;
                    }
                }
            }
            return;
        }
        int n,m,lim;
        Complex a[N],b[N];
        int main()
        {
            n=read(),m=read();
            for(int i=0;i<=n;i++) a[i].re=read();
            for(int i=0;i<=m;i++) b[i].re=read();
            lim=1;
            while(lim<=n+m) lim*=2,bit++;
            FFT(a,lim,1);
            FFT(b,lim,1);
            for(int i=0;i<=lim;i++) a[i]=a[i]*b[i];
            FFT(a,lim,-1);
            for(int i=0;i<=n+m;i++) printf("%d ",int(a[i].re/lim+0.5));
            return 0;
        }
        ```