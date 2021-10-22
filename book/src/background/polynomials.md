# 多项式

$A(X)$ 是一个以 $X$ 为自变量的 $\mathbb{F}_p$ 上的多项式。例如，

$$
A(X) = a_0 + a_1 X + a_2 X^2 + a_3 X^3
$$

就定义了一个 $3$次多项式。$a_0$ 是常数项。$n-1$ 次多项式有 $n$ 个系数。
我们经常想要将自变量 $X$ 用一个具体值 $x$ 替换掉，以计算其在该具体值 $x$ 处的值，表示为 $A(x)$。

> 在数学上这通常被称为 “求 $A(X)$ 在点 $x$ 处的值”。
> 所谓“点”，是从多项式的几何表示来的，
> 将多项式表示为 $y = A(x)$，那么 $(x, y)$ 就是一个二维空间中的一个点的坐标。
> 但是我们处理的多项式通常总是等于零，并且 $x$ 是一个[某个域中的元素](fields.md)。
> 不应该与[椭圆曲线](curves.md)的点混淆，椭圆曲线上的点的说法我们也要经常使用，但是却从不在多项式求值的语境下使用。（？？？）

重要说明：

* 多项式相乘，将产生这样一个积多项式，该多项式的次数是其因子多项式的次数的和。多项式除法，则会减去相应的次数。
  $$\deg(A(X)B(X)) = \deg(A(X)) + \deg(B(X)),$$
  $$\deg(A(X)/B(X)) = \deg(A(X)) -\deg(B(X)).$$

* 给定一个 $n-1$ 次多项式 $A(X)$，如果我们在 $n$ 个不同点求取了多项式的值，那么这些值就能唯一确定该多项式。
  换句话说，给定 $n$ 个不同点值，我们就能通过多项式插值来唯一确定 $n-1$ 次多项式 $A(X)$。

* $[a_0, a_1, \cdots, a_{n-1}]$ 是多项式 $A(X)$ 的 **系数表示**,
  我们也可以用它在 $n$ 个不同点处的 **点值表示**
  $$[(x_0, A(x_0)), (x_1, A(x_1)), \cdots, (x_{n-1}, A(x_{n-1}))]$$
  来表示它。不管那种表示方法，都唯一确定了相同的多项式。

> #### （备注） 霍纳法则
> 霍纳法则可以仅仅使用 $n-1$ 次乘法和 $n-1$ 次加法就高效地求得一个 $n-1$ 次多项式在某处的取值。
> 霍纳法则如下：
> $$\begin{aligned}a_0 &+ a_1X + a_2X^2 + \cdots + a_{n-1}X^{n-1} \\ &= a_0 + X\bigg( a_1 + X \Big( a_2 + \cdots + X(a_{n-2} + X a_{n-1}) \Big)\!\bigg),\end{aligned}$$

## 快速傅里叶变换 (FFT)

FFT是一种在系数表示和点值表示之间进行转换的高效方法。它在多项式的 $n$ 个 $n$ 次单位根上求取多项式的值。
这 $n$ 个 $n$ 次单位根是 $\{\omega^0, \omega^1, \cdots, \omega^{n-1}\},$ 其中 $\omega$ 是多项式的主单位根。
通过利用单位根中的对称性，FFT的每一轮计算都只计算问题规模的一半。绝大多数情况下，我们都要求多项式的次数是 $2$ 的
幂次，即 $n = 2^k$，并递归运用减半算法。

### 动机：快速计算多项式乘法

在系数表示下，计算多项式相乘需要 $O(n^2)$ 次计算。
比如，$A(X)\cdot B(X) = C(X)$：

$$
\begin{aligned}
A(X) &= a_0 + a_1X + a_2X^2 + \cdots + a_{n-1}X^{n-1}, \\
B(X) &= b_0 + b_1X + b_2X^2 + \cdots + b_{n-1}X^{n-1}, \\
C(X) &= a_0\cdot (b_0 + b_1X + b_2X^2 + \cdots + b_{n-1}X^{n-1}) \\
&+ a_1X\cdot (b_0 + b_1X + b_2X^2 + \cdots + b_{n-1}X^{n-1})\\
&+ \cdots \\
&+ a_{n-1}X^{n-1} \cdot (b_0 + b_1X + b_2X^2 + \cdots + b_{n-1}X^{n-1}),
\end{aligned}
$$

显而易见，第一个多项式中的 $n$ 项必须与第二个多项式的 $n$ 项相乘。

而在点值表示下，多项式相乘仅仅需要 $O(n)$ 计算：

$$
\begin{aligned}
A&: \{(x_0, A(x_0)), (x_1, A(x_1)), \cdots, (x_{n-1}, A(x_{n-1}))\}, \\
B&: \{(x_0, B(x_0)), (x_1, B(x_1)), \cdots, (x_{n-1}, B(x_{n-1}))\}, \\
C&: \{(x_0, A(x_0)B(x_0)), (x_1, A(x_1)B(x_1)), \cdots, (x_{n-1}, A(x_{n-1})B(x_{n-1}))\},
\end{aligned}
$$

每个值都按照点的顺序相应计算即可。

这就提示了如下的快速计算多项式乘法的策略：

1. 对多项式在 $n$ 个点处求值（转成点值表示）

2. 在点值表示下，按照点的顺序依次计算积多项式的点 ($O(n)$) （计算积多项式的点值表示）;

3. 将积多项式的点值表示转换成系数表示。

现在的挑战就是如何高效地进行多项式 **求值** 和 **插值**。
最朴素的方法，对多项式在 $n$ 个点处进行求值将需要 $O(n^2)$ 计算（我们在每一个点处都使用 $O(n)$ 的霍纳法则）：


$$
\begin{bmatrix}
A(1) \\
A(\omega) \\
A(\omega^2) \\
\vdots \\
A(\omega^{n-1})
\end{bmatrix} =
\begin{bmatrix}
1&1&1&\dots&1 \\
1&\omega&\omega^2&\dots&\omega^{n-1} \\
1&\omega^2&\omega^{2\cdot2}&\dots&\omega^{2\cdot(n-1)} \\
\vdots&\vdots&\vdots& &\vdots \\
1&\omega^{n-1}&\omega^{2(n-1)}&\cdots&\omega^{(n-1)^2}\\
\end{bmatrix} \cdot
\begin{bmatrix}
a_0 \\
a_1 \\
a_2 \\
\vdots \\
a_{n-1}
\end{bmatrix}.
$$

方便起见，我们将如下表示上面的矩阵：
$$\hat{\mathbf{A}} = \mathbf{V}_\omega \cdot \mathbf{A}. $$

（$\hat{\mathbf{A}}$ 就是 $\mathbf{A}$ 的 *离散傅里叶变换*；
$\mathbf{V}_\omega$ 也被称为 *范德蒙矩阵*。）

### (radix-2) Cooley-Tukey 算法

我们的策略就是将一个规模为 $n$ 的 DFT 分成两个规模分别为 $n/2$ 的 DFT。假定多项式
$A(X) = a_0 + a_1X + a_2X^2 + \cdots + a_{n-1}X^{n-1},$ 我们将其按照奇数项和偶数项进行拆分：

$$
\begin{aligned}
A_{\text{even}} &= a_0 + a_2X + \cdots + a_{n-2}X^{\frac{n}{2} - 1}, \\
A_{\text{odd}} &= a_1 + a_3X + \cdots + a_{n-1}X^{\frac{n}{2} - 1}. \\
\end{aligned}
$$

于是原多项式就是：
$A(X) = A_{\text{even}} (X^2) + X A_{\text{odd}}(X^2).$

尝试在 $\omega_n^i$ and $\omega_n^{\frac{n}{2} + i}$,
$i \in [0..\frac{n}{2}-1]$ 处求值，我们就能发现某些对称性：

$$
\begin{aligned}
A(\omega_n^i) &= A_{\text{even}} ((\omega_n^i)^2) + \omega_n^i A_{\text{odd}}((\omega_n^i)^2), \\
A(\omega_n^{\frac{n}{2} + i}) &= A_{\text{even}} ((\omega_n^{\frac{n}{2} + i})^2) + \omega_n^{\frac{n}{2} + i} A_{\text{odd}}((\omega_n^{\frac{n}{2} + i})^2) \\
&= A_{\text{even}} ((-\omega_n^i)^2) - \omega_n^i A_{\text{odd}}((-\omega_n^i)^2) \leftarrow\text{(negation lemma)} \\
&= A_{\text{even}} ((\omega_n^i)^2) - \omega_n^i A_{\text{odd}}((\omega_n^i)^2).
\end{aligned}
$$

值得注意的是，我们仅仅在一半的domain上求取 $A_{\text{even}}(X)$ 和 $A_{\text{odd}}(X)$
这是因为 $\{(\omega_n^0)^2, (\omega_n)^2, \cdots, (\omega_n^{\frac{n}{2} -1})^2\} = \{\omega_{n/2}^i\}, i = [0..\frac{n}{2}-1]$ （折半引理）。

但是这些值却能帮助我们在完整domain $\{\omega^0, \omega, \cdots, \omega^{n -1}\}$ 上求出 $A(X)$。
这就意味着，我们将一个成都为 $n$ 的 DFT 转化成了两个长度为 $n/2$ 的 DFT。

我们选择 $n = 2^k$ 为 $2$ 的幂次（必要时，可以添 $0$ 补齐），然后递归地运用这种分治策略。
由主定理可知[^master-thm]，这种多项式求值算法仅需要 $O(n\log_2n)$ 计算，这就是著名的快速傅里叶变换（FFT）。

### 逆FFT

我们已经求得了多项式的值并将它们按照点的顺序进行了相乘。剩下的工作就是要把积的点值表示转换成系数表示。
做这件事，我们仍然是在点值上调用FFT。只不过，这次我们将做如下修正：
- 将范德蒙矩阵中的 $\omega^i$ 用 $\omega^{-i}$ 替换掉
- 把最后的结果乘以 $1/n$。

也就是：
$$\mathbf{A} = \frac{1}{n} \mathbf{V}_{\omega^{-1}} \cdot \hat{\mathbf{A}}. $$

(为了理解为什么逆FFT与FFT有相似的形式，请参考 13-1
[^ifft]。下图也是从 [^ifft]来的。）

![](https://i.imgur.com/lSw30zo.png)


## Schwartz-Zippel 引理

Schwartz-Zippel 引理的通俗解释就是 “不同的多项式在绝大多数点上的值都是不同的”。
它的正式表述如下：

> 令 $p(x_1, x_2, \cdots, x_n)$ 是有 $n$ 个变量的 $d$ 次非零多项式。
> $S$ 是一个至少有 $d$ 个元素的集合。如果我们从 $S$ 中选择随机数
> $\alpha_1, \alpha_1, \cdots, \alpha_n$,
> $$\text{Pr}[p(\alpha_1, \alpha_2, \cdots, \alpha_n) = 0] \leq \frac{d}{|S|}.$$

在 $p$ 是我们更熟悉的单变量多项式的情况下，如 $p(X)$, 这就退化成，一个 $d$ 次的非零多项式至多有 $d$ 个根。

Schwartz-Zippel引理可以用来检验两个多项式是否相等。设有两个多变量的多项式
$p_1(x_1,\cdots,x_n)$ 和 $p_2(x_1,\cdots,x_n)$，它们的次数分别是 $d_1，d_2$。
我们选取随机数 $\alpha_1, \cdots, \alpha_n \leftarrow S$，其中 $S$ 的元素个数至少是 $|S| \geq (d_1 + d_2)$，
进而验证 $p_1(\alpha_1, \cdots, \alpha_n) - p_2(\alpha_1, \cdots, \alpha_n) = 0$ 是否都成立。
如果两个多项式相等，那么上式会一直成立，而如果两个多项式不相等，那么上式成立的概率至多就是 $\frac{\max(d_1,d_2)}{|S|}$ 。

## 消除多项式

考虑这样一个 $n$-阶乘法子群 $\mathcal{H}$，它的主单位根是 $\omega$ 。
对所有 $\omega^i \in \mathcal{H}, i \in [n-1]$，有 
$(\omega^i)^n = (\omega^n)^i = (\omega^0)^i = 1$。这就是说
$\mathcal{H}$ 中的每个元素都满足如下等式 

$$
\begin{aligned}
Z_H(X) &= X^n - 1 \\
&= (X-\omega^0)(X-\omega^1)(X-\omega^2)\cdots(X-\omega^{n-1}),
\end{aligned}
$$

这就是说每个元素都是 $Z_H(X)$ 的根。我们称 $Z_H(X)$ 就是在 $\mathcal{H}$ 上的 **消除多项式**
因为该多项式在 $\mathcal{H}$ 中每个元素处的取值都为 $0$。

这在检验多项式约束的时候尤其有用。举个例子,
检验 $A(X) + B(X) = C(X)$ 在 $\mathcal{H}$ 上的正确性，我们只需简单地验证
$A(X) + B(X) - C(X)$ 的 $Z_H(X)$ 倍数。这就是说，如果我们的约束除以消除多项式，仍然是一个多项式，即：
$\frac{A(X) + B(X) - C(X)}{Z_H(X)} = H(X)$，那么我们就说在 $\mathcal{H}$ 上 $A(X) + B(X) - C(X) = 0$ 成立。

## 拉格朗日基函数

> TODO: explain what a basis is in general (briefly).

多项式通常以单项式基(例如，$X, X^2, ... X^n$)来表示。但是当我们在一个 $n$ 阶乘法子群上工作时，我们
找到了更加自然表示方式，就是以拉格朗日基来表示。

考虑一个 $n$-阶乘法子群 $\mathcal{H}$，它的主单位根是
$\omega$。则该子群相应的拉格朗日基就是这样一个函数集合
$\{\mathcal{L}_i\}_{i = 0}^{n-1}$, 其中

$$
\mathcal{L_i}(\omega^j) = \begin{cases}
1 & \text{if } i = j, \\
0 & \text{otherwise.}
\end{cases}
$$

我们可以更简洁地写为 $\mathcal{L_i}(\omega^j) = \delta_{ij},$ 其中 $\delta$ 就是Kronecker delta函数。 

于是，我们可以用拉格朗日基函数的线性组合来表示我们的多项式,

$$A(X) = \sum_{i = 0}^{n-1} a_i\mathcal{L_i}(X), X \in \mathcal{H},$$

上式其实就等价于我们说 $p(X)$ 在 $\omega^0$ 的值是 $a_0$，$\omega^1$ 的值是 $a_1$，在 $\omega^2$ 的值是 $a_2$，$\cdot$ 等等。

当我们在一个乘法子群上工作时，拉格朗日基函数有一个很方便的稀疏表示形式：

$$
\mathcal{L}_i(X) = \frac{c_i\cdot(X^{n} - 1)}{X - \omega^i},
$$

其中 $c_i$ 质心权重。(欲理解该形式如何推导，请参考[^barycentric]。) 对 $i = 0$，有

$c = 1/n \implies \mathcal{L}_0(X) = \frac{1}{n} \frac{(X^{n} - 1)}{X - 1}$.

假设我们有一系列多项式的值 $\{x_0, x_1, \cdots, x_{n-1}\}$。
由于我们不能假设 $x_i$ 会形成一个乘法子群，我们就用拉格朗日多项式 $\mathcal{L}_i$ 的一般形式
于是我们构建：

$$
\mathcal{L}_i(X) = \prod_{j\neq i}\frac{X - x_j}{x_i - x_j}, i \in [0..n-1].
$$

这里，任何 $X = x_j \neq x_i$ 都会产生一个分子为 $0$ 的项 $(x_j - x_j)$，这就导致
整个的积就是 $0$。另一方面，当 $X= x_i$ 时，每一项都是 $\frac{x_i - x_j}{x_i - x_j}$ 
因此整体的积就是 $1$。于是我们给出了在集合 $\{x_0, x_1, \cdots, x_{n-1}\}$ 上的 Kronecker delta 函数 $\mathcal{L_i}(x_j) = \delta_{ij}$。

### 拉格朗日插值

设给定一个多项式的点值表示如下，

$$A: \{(x_0, A(x_0)), (x_1, A(x_1)), \cdots, (x_{n-1}, A(x_{n-1}))\},$$

我们可以利用拉格朗日基重建其系数表示：

$$A(X) = \sum_{i = 0}^{n-1} A(x_i)\mathcal{L_i}(X), $$

其中 $X \in \{x_0, x_1,\cdots, x_{1-n}\}$。

## References
[^master-thm]: [Dasgupta, S., Papadimitriou, C. H., & Vazirani, U. V. (2008). "Algorithms" (ch. 2). New York: McGraw-Hill Higher Education.](https://people.eecs.berkeley.edu/~vazirani/algorithms/chap2.pdf)

[^ifft]: [Golin, M. (2016). "The Fast Fourier Transform and Polynomial Multiplication" [lecture notes], COMP 3711H Design and Analysis of Algorithms, Hong Kong University of Science and Technology.](http://www.cs.ust.hk/mjg_lib/Classes/COMP3711H_Fall16/lectures/FFT_Slides.pdf)

[^barycentric]: [Berrut, J. and Trefethen, L. (2004). "Barycentric Lagrange Interpolation."](https://people.maths.ox.ac.uk/trefethen/barycentric.pdf)
