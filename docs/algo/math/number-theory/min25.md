# Min_25 筛

Min_25 筛是在低于线性时间的复杂度内求出积性函数 $f$ 前缀和的算法。

## 记号约定 $\def\floor#1{\left\lfloor#1\right\rfloor}\def\bracket#1{\left(#1\right)}\def\minp{\operatorname{minp}}$

- 下文若无特殊说明，名为 $p$ 的变量均属于质数集。
- $\mathbb{P}$：质数集。
- $p_i$：从小到大第 $i$ 个质数。特别地，$p_0=1$。
- $\minp(i)$：$i$ 的最小质因子。

## 适用范围

1. $f$ 是一个积性函数；
2. $f(p^c)$ 可以被表示为少量完全积性函数之和（比如多项式）。

## 算法思路

### 第一部分

当我们求 $f(n)$ 的值时，我们可以把它的最小质因子的那一项拆出来，递归求其剩余的部分，即 $f(n)=f(\minp(n)^c)f\bracket{\floor{\frac{n}{\minp(n)^c}}}$（$c$ 为最小质因子的次数）。根据“适用范围”中的条件，$f(\minp(n)^c)$ 是容易求的。

令 $S(n,j)$ 表示 $2\sim n$ 中最小质因子大于 $p_j$ 的数的 $f$ 值之和，即：

$$
S(n,j)=\sum_{i=2}^{n}[\minp(i)>p_j]f(i)
$$

那么最终的答案为 $S(n,0)+f(1)$。

我们枚举这些 $i$ 的最小质因子以及其次数，有：

$$
S(n,j)=\sum_{\substack{k>j \\ p_{k}^{2}\le n}}\sum_{\substack{c\ge 1 \\ p_{k}^{c}\le n}}f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}+\sum_{\substack{k>j \\ p_k\le n}}f(p_k)
$$

???+ note "解释"

    首先，这个式子被分成了两个部分：以最后一步的加号为界，前半部分计算 $\minp>p_j$ 的合数的 $f$ 值，后半部分计算 $>p_j$ 的质数的 $f$ 值。

    -   对于前半部分，我们枚举合数的最小质因子 $p_k$ 和最小质因子的系数 $c$。由于是合数，我们要求 $p_k^2\le n$。
        
        对于所有最小质因子及次数为 $p_k^c$ 的合数 $i$，我们将 $f(i)$ 拆成 $f(p_k^c)f\bracket{\frac{i}{p_k^c}}$ 的形式（因为两者肯定互质），则：

        $$
        \begin{aligned}
        &\sum_{i}f(i)\\
        =&\sum_{i}f(p_k^c)f\bracket{\frac{i}{p_k^c}}\\
        =&f(p_k^c)\sum_{i}f\bracket{\frac{i}{p_k^c}}
        \end{aligned}
        $$

        由于 $i$ 的最小质因子为 $p_k$，所以 $\minp\bracket{\frac{i}{p_k^c}}>p_k$；同时 $\frac{i}{p_k^c}$ 的最大值为 $\frac{n}{p_k^c}$，最小值为 $[c>1]$（$c=1$ 的时候不能取到 $1$，否则 $i$ 为质数），所以：

        $$
        \sum_{i}f(i)=f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}
        $$

    -   对于后半部分，枚举范围内的质数，把 $f$ 加起来。

我们不妨设 $q(n)$ 表示 $\sum_{p\le n}f(p)$，则改写原式后半部分为：

$$
S(n,j)=\sum_{\substack{k>j \\ p_{k}^{2}\le n}}\sum_{\substack{c\ge 1 \\ p_{k}^{c}\le n}}f(p_{k}^{c})\bracket{S\bracket{\floor{\frac{n}{p_{k}^{c}}},k}+[c>1]}+q(n)-\sum_{i=1}^{j}f(p_i)
$$

现在，只要能求 $q$ 的值，就能求 $S$ 的值了。

### 第二部分

求 $q$ 的过程，我们使用埃筛的思想。

由于 $n$ 很大，我们不能直接对质数下手，而是考虑把 $[1,n]$ 中的合数都筛掉，剩下的就是质数的答案。我们发现埃筛的过程，是从小到大枚举每一个质数，筛掉这个质数的所有倍数。我们可以利用这种思路求 $q$。

但这里仍然有一个问题：$f$ 并不是完全积性函数，当 $f(n)$ 被 $\minp(n)$ 筛到时，$f(\minp(n))f\bracket{\frac{n}{\minp(n)}}\neq f(n)$。

注意到我们在这一部分的最终目的是求出所有质数的 $f$ 值之和，其在合数处的取值我们是不关心的。由“适用范围”得，$f$ 在质数的次方处可以拆成若干个完全积性函数，我们令完全积性函数 $f'$ 依次等于这些函数，对 $f'$

我们令 $g_t(n,j)$ 表示 $[2,n]$ 的数在第 $j$ 轮埃筛后，剩下的数字的 $f'_{t}$ 值之和，即：

$$
g_t(n,j)=\sum_{i=2}^{n}[i\in\mathbb{P}\operatorname{or}\minp(i)>p_j]f'_{t}(i)
$$

尝试从第 $j-1$ 次埃筛推出第 $j$ 次埃筛，我们有递推式：

$$
g_t(n,j)=
\left\{
\begin{aligned}
&g_t(n,j-1)&,p_{j}^{2}>n\\
&g_t(n,j-1)-f'_{t}(p_j)\bracket{g_t\bracket{\floor{\frac{n}{p_j}},j}-\sum_{i=1}^{j-1}f'_{t}(p_i)}&,p_{j}^{2}\le n
\end{aligned}
\right.
$$

???+ note "解释"

    递推式的大致思路是从第 $j-1$ 轮埃筛，减掉第 $j$ 轮埃筛涉及到的数（最小质因子为 $p_j$ 的合数）的 $f'_{t}$ 值，得到 $g_t(n,j)$ 的值。

    首先，显然地，当 $p_j^2>n$，没有最小质因子等于 $p_j$ 的合数，$g_t(n,j)=g_t(n,j-1)$。

    当存在 $\minp=p_j$ 的合数时，模仿 $S$ 的求解过程，我们将这些合数的 $f'_t$ 值提取公因数 $f'_t(p_j)$，剩下的部分为 $g_t\bracket{\floor{\frac{n}{p_j}},j}-\sum_{i=1}^{j-1}f'_{t}(p_i)$，