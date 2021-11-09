## 实用的小方法

本节包含几个写 halo2 电路时能用帮上忙的小策略。

### 小范围（or 区间?）约束

在 R1CS 电路中，经常用到的一种约束就是布尔约束: $b * (1-b) = 0$. 这种约束限定了 $b = 0$ 或者 $b = 1$. 

类似地，在 halo2 电路中，你也可以限定一个 *单元格* 在一个小集合内取值。比如，若要限定 $a$的值落在区间 $[0..5]$，你可以构建如下形式的门: 

$$ a \cdot (1-a) \cdot (2-a) \cdot (3- a) \cdot (4-a) = 0$$

又比如，要限定 $c$ 等于 7 或 13，你可以用如下的门:

$$(7-c) \cdot (13 - c) = 0$$

> 基本原理就是，我们构建一个多项式约束，使得其根集合与我们想限定的取值集合相同。
> R1CS 电路支持的多项式次数最大为2（因为所有的约束方程必须具有 $a * b = c$ 的形式）。
> 而 halo2 电路支持任意次数的多项式，只不过更高次数会带来更大的代价。

需要注意，这些根不必是常量；比如，可以用 $ (a -x) \cdot (a - y) \cdot (a - z) = 0$ 来限定 $a$ 等于集合 $\{ x, y, z\}$ 中的任一元素。而且，

### 小集合插值
我们可以用 Lagrange 插值法构造一个多项式约束，证明 $f(X) = Y$，其中 $X \in \{ x_i \}, Y \in \{ y_i \}$ 。

例如，假设我们想把两个比特的值通过插零扩展为“Spread”4个比特值。我们首先预计算出每一个点上的取值：

$$
\begin{array}{rcl}
00 \rightarrow 0000 &\implies& 0 \rightarrow 0 \\
01 \rightarrow 0001 &\implies& 1 \rightarrow 1 \\
10 \rightarrow 0100 &\implies& 2 \rightarrow 4 \\
11 \rightarrow 0101 &\implies& 3 \rightarrow 5
\end{array}
$$

然后，为每一个点计算出对应的 Lagrange 基多项式(basis polynomial)，其中 $k$ 是点的个数（在我们的例子中，$k = 4$）：

$$\mathcal{l}_j(X) = \prod_{0 \leq m < k,\; m \neq j} \frac{x - x_m}{x_j - x_m},$$


回顾一下，Lagrange 基多项式 $l_j(X)$ 满足如下性质：若 $X = x_j$, 则 $l_j(X) = 1$；否则，$X = x_i (i \neq j)$，一定有 $l_j(X) = 0$。

回到我们刚才举的例子，我们就计算出了4个 Lagrange 基多项式：

$$
\begin{array}{ccc}
l_0(X) &=& \frac{(X - 3)(X - 2)(X - 1)}{(-3)(-2)(-1)} \\[1ex]
l_1(X) &=& \frac{(X - 3)(X - 2)(X)}{(-2)(-1)(1)} \\[1ex]
l_2(X) &=& \frac{(X - 3)(X - 1)(X)}{(-1)(1)(2)} \\[1ex]
l_3(X) &=& \frac{(X - 2)(X - 1)(X)}{(1)(2)(3)}
\end{array}
$$


那么，我们就得到了如下所示的多项式约束：

$$
\begin{array}{cccccccccccl}
&f(0) \cdot l_0(X) &+& f(1) \cdot l_1(X) &+& f(2) \cdot l_2(X) &+& f(3) \cdot l_3(X) &-& f(X) &=& 0 \\
\implies& 0 \cdot l_0(X) &+& 1 \cdot l_1(X) &+& 4 \cdot l_2(X) &+& 5 \cdot l_3(X) &-& f(X) &=& 0. \\
\end{array}
$$

