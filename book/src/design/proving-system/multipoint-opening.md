# 多点打开证明 

假设 $A, B, C, D$ 分别是多项式 $a(X), b(X), c(X), d(X)$ 的多项式承诺值。比如说，我们要打开多项式 $a, b$ 在点$x$的取值，与此同时，打开$c, d$ 在两个点$x, \omega x$ 的取值。（此处的 $\omega$ 是乘法子群的本原单位根）。

比较容易想到的方法是，为每一个点构造一个多项式$Q$（每一个打开点就会对应电路中的一个相对转动（rotation））。但这不是一个有效的方法；比如，$c(X)$ 会出现在多个多项式中。

相反地，我们按打开点的集合，将多项式承诺值组织在一起：
$$
\begin{array}{cccc}
&\{x\}&     &\{x, \omega x\}& \\
 &A&            &C& \\
 &B&            &D&
\end{array}
$$

对于每一个组，我们将这些多项式合并成一个多项式集合，然后构造一个多项式$Q$， 并在该点打开所有相对偏移。

## 优化步骤

多点打开的优化算法的输入为：

- 验证者采样出一个随机点$x$，我们将计算多项式$a(X), b(X), c(X), d(X)$在该点的值；
- 证明者计算出那些我们想要的多项式点值：$a(x), b(x), c(x), d(x), c(\omega x), d(\omega x)$。

这些数据是 [vanishing证明](vanishing.md) 的输出。

多点打开优化算法的步骤如下：

1. 采样一个随机点$x_1$，用来让 $a, b, c, d$ 线性无关；
2. 对于每一个点集，将对应的多项式集合聚合成一个多项式，得到新的多项式集合 `q_polys`。

    $$
    \begin{array}{rccl}
    q_1(X) &=& a(X) &+& x_1 b(X) \\
    q_2(X) &=& c(X) &+& x_1 d(X)
    \end{array}
    $$
同理对于每一个点集，也计算出新的多项式点值集合 `q_eval_sets`。

```math
            [
                [a(x) + x_1 b(x)],
                [
                    c(x) + x_1 d(x),
                    c(\omega x) + x_1 d(\omega x)
                ]
            ]
```

3. 根据`q_eval_sets`，构造插值多项式集合 `r_polys`:
    $$
    \begin{array}{cccc}
    r_1(X) s.t.&&& \\
        &r_1(x) &=& a(x) + x_1 b(x) \\
    r_2(X) s.t.&&& \\
        &r_2(x) &=& c(x) + x_1 d(x) \\
        &r_2(\omega x) &=& c(\omega x) + x_1 d(\omega x) \\
    \end{array}
    $$
4. 构造 `f_polys` 来检查 `q_polys` 的正确性：

    $$
    \begin{array}{rcl}
    f_1(X) &=& \frac{ q_1(X) - r_1(X)}{X - x} \\
    f_2(X) &=& \frac{ q_2(X) - r_2(X)}{(X - x)(X - \omega x)} \\
    \end{array}
    $$

   如果 $q_1(x) = r_1(x)$，则 $f_1(X)$ 应当是一个多项式；
   如果 $q_2(x) = r_2(x)$并且 $q_2(\omega x) = r_2(\omega x)$，则 $f_2(X)$ 应当是一个多项式。

5. 采样一个随机数 $x_2$，用来让 `f_polys` 线性无关。
6. 构造 $f(X) = f_1(X) + x_2 f_2(X)$。
7. 采样一个随机数$x_3$，并计算$f(X)$在该点的取值：
    $$
    \begin{array}{rcccl}
    f(x_3) &=& f_1(x_3) &+& x_2 f_2(x_3) \\
           &=& \frac{q_1(x_3) - r_1(x_3)}{x_3 - x} &+& x_2\frac{q_2(x_3) - r_2(x_3)}{(x_3 - x)(x_3 - \omega x)}
    \end{array}
    $$
8. 采样一个随机数$x_4$，用来让 $f(X)$ 和 `q_polys` 线性无关。
9. 构造 `final_poly`多项式 $$\textrm{final_poly}(X) = f(X) + x_4 q_1(X) +x_4^2 q_2(X) $$
  `final_poly`多项式就是传入内积证明的多项式。
