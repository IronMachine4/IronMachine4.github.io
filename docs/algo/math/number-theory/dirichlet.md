# 狄利克雷卷积

狄利克雷卷积是一种函数之间的运算，我们记函数 $f$，$g$ 的卷积为 $f*g$，其定义如下：

$$
(f*g)(d)=\sum_{i\mid d}f(i)g\left(\frac{d}{i}\right)
$$

狄利克雷卷积有如下性质：

- 交换律：$f*g=g*f$；
- 结合律：$(f*g)*h=f*(g*h)$；
- 分配律：$(f+g)*h=f*h+g*h$；
- 单位元：$f*\varepsilon=f$。

狄利克雷卷积是数论中的重要运算。以下是常见数论函数的卷积关系：

- $\varepsilon=\mu*I$
- $\sigma=\text{id}*I$
- $\varphi=\mu*\text{id}$
- $\text{id}=I*\varphi$

???+ success "推数论式子 trick"

    当求和符号中出现类似 $d\mid i$ 等枚举因数的东西，可以考虑交换求和顺序，改为先枚举 $d$ 再枚举 $d$ 的倍数，效果一般不错。