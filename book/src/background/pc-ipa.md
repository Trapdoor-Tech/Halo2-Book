# 使用内积证明的多项式承诺

我们欲对 $p(X) \in \mathbb{F}_p[X]$ 进行承诺，并且可以在任意一点验证承诺的多项式的正确性。
最简单的解决方案就是证明者将多项式的系数发给验证者。但是这种方法需要 $O(n)$ 次通信。我们的
多项式承诺机制可以仅用 $O(\log n)$ 次通信就完成工作。

### `Setup`

给定一个参数 $d = 2^k$，并且生成了CRS（common reference string）$\sigma = (\mathbb{G}, \mathbf{G}, H, \mathbb{F}_p)$,
并定义了该机制中的一些常数：

* $\mathbb{G}$ 是一个 $p$ 阶的群，其中 $p$ 为一个素数；
* $\mathbf{G} \in \mathbb{G}^d$ 是 $d$ 维向量，它的元素都是在群元素中随机挑选；
* $H \in \mathbb{G}$ 是一个随机的群元素；
* $\mathbb{F}_p$ 一个 $p$-阶有限域。

### `Commit`

定义Pedersen向量承诺为 $\text{Commit}$

$$\text{Commit}(\sigma, p(X); r) = \langle\mathbf{a}, \mathbf{G}\rangle + [r]H,$$

其中  $p(X) \in \mathbb{F}_p[X]$ 是欲承诺的多项式，$r \in \mathbb{F}_p$ 是盲化因子。
向量 $a$ 中的每一个元素 $\mathbf{a}_i \in \mathbb{F}_p$ 就是多项式 $p(X)$，的第 $i$ 项的系数。而 $p(X)$ 的最大次数是 $d - 1$。

### `Open` (prover) and `OpenVerify` (verifier)

经过修改的内积证明是对如下关系信息的一种证明：

$$\boxed{\{((P, x, v); (\mathbf{a}, r)): P = \langle\mathbf{a}, \mathbf{G}\rangle + [r]H, v = \langle\mathbf{a}, \mathbf{b}\rangle\}},$$

其中 $\mathbf{b} = (1, x, x^2, \cdots, x^{d-1})$ 而 $x$ 是欲求值的点。 
内积证明就使证明者能够向验证者证明，承诺 $P$ 中所承诺的多项式，在 $x$ 处的值就是 $v$，还有
承诺的多项式的最大阶数是 $d - 1$。

内积证明将进行 $k = \log_2 d$ 轮。就我们的目的来说，我们只要知道最终输出就足够了，所以我们只是对
中间轮提供一个直观的描述。(请参考 [Halo] 论文，以获取全面的解释)。

[Halo]: https://eprint.iacr.org/2019/1021.pdf

在开始证明之前，验证者要随机选取一个群元素 $U$ 并将它发送给证明者。
我们在第 $k$ 轮将对所需向量做如下初始化，$\mathbf{a}^{(k)} := \mathbf{a},$ $\mathbf{G}^{(k)} := \mathbf{G}$，
$\mathbf{b}^{(k)} := \mathbf{b}$。在每一轮 $j = k, k-1, \cdots, 1$:

* 证明者都要用 $\mathbf{a}^{(j)}$，$\mathbf{G}^{(j)}$ 和 $\mathbf{b}^{(j)}$ 进行内积计算得到两个值 $L_j$ 和 $R_j$。
  这里要“交叉项”("cross-terms")有个概念，即：$\mathbf{a}$ 的低半部分与 $\mathbf{G}$ 和 $\mathbf{b}$ 的高半部分运算，反之亦然。

$$
\begin{aligned}
L_j &= \langle\mathbf{a_{lo}^{(j)}}, \mathbf{G_{hi}^{(j)}}\rangle + [l_j]H + [\langle\mathbf{a_{lo}^{(j)}}, \mathbf{b_{hi}^{(j)}}\rangle] U\\
R_j &= \langle\mathbf{a_{hi}^{(j)}}, \mathbf{G_{lo}^{(j)}}\rangle + [l_j]H + [\langle\mathbf{a_{hi}^{(j)}}, \mathbf{b_{lo}^{(j)}}\rangle] U\\
\end{aligned}
$$

* 验证者发送一个随机挑战 $u_j$；

* 证明者就使用 $u_j$ 将 $\mathbf{a}^{(j)}$的低半部分与高半部分压缩乘只有原向量一半的新的向量。
  $$\mathbf{a}^{(j-1)} = \mathbf{a_{hi}^{(j)}}\cdot u_j^{-1} + \mathbf{a_{lo}^{(j)}}\cdot u_j.$$
  向量 $\mathbf{G}^{(j)}$ 和 $\mathbf{b}^{(j)}$ 也进行类似的压缩，从而得到
  $\mathbf{G}^{(j-1)}$ 和 $\mathbf{b}^{(j-1)}$.

* $\mathbf{a}^{(j-1)}$, $\mathbf{G}^{(j-1)}$ and $\mathbf{b}^{(j-1)}$ 是下一轮，即 $j - 1$ 轮的输入。

注意在最后一轮，即 $j = 1$ 轮，我们只剩下 $a := \mathbf{a}^{(0)}$,
$G := \mathbf{G}^{(0)}$, $b := \mathbf{b}^{(0)},$ 每个向量的长度都是1。直观讲，
这些最终的标量，和每一轮的 $\{u_j\}$ 以及每一轮的“交叉项”
$\{L_j, R_j\}$， 实际上是编码了每一轮的压缩操作。由于证明者并不能事先知道挑战
$U, \{u_j\}$，他们就不能人为操控每一轮的压缩。因此，验证最后这些项的约束就能证明每一轮的压缩都是正确的，
同时也证明了在压缩开始前原始 $a$ 就满足特定的关系。

事实上，每一轮的 $G, b$ 就是公开信息 $\mathbf{G}, \mathbf{b},$ 与 $\{u_j\}$ 计算出来的，
因此验证者可以自己计算每一轮的 $G, b$，以验证证明者是否提供了正确的值（二者值应该相等）。