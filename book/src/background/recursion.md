## 递归
> Alternative terms: Induction; Accumulation scheme; Proof-carrying data

计算 $G$ 需要计算长度 $2^k$ 多倍点
$\langle \mathbf{G}, \mathbf{s}\rangle,$ 其中 $\mathbf{s}$ 每一轮挑战 $u_1, \cdots, u_k$ 按照二进制查数的顺序进行排列组成的。
在多个证明的验证中，我们希望平摊这个线性计算。
和计算 $G$不同， 我们可以将 $G$ 表示成一个多项式的承诺

$$G = \text{Commit}(\sigma, g(X, u_1, \cdots, u_k)),$$

其中 $g(X, u_1, \cdots, u_k) := \prod_{i=1}^k (u_i + u_i^{-1}X^{2^{i-1}})$ 是一个 $2^k - 1$ 次多项式。 

|  |  |
| -------- | -------- |
| <img src="https://i.imgur.com/vMXKFDV.png" width=1900> | 由于 $G$ 是一个承诺，它就可以在内积证明中被验证。验证电路的 witnesses $G$ 并产生 $G, u_1, \cdots, u_k$ 做为证明 $\pi$ 的公开输入。下一个验证者将运用内积证明来检查 $\pi$ ；这包括对给定的挑战 $u_1, \cdots, u_k.$ ，查验 $G = \text{Commit}(g(X, u_1, \cdots, u_k))$ 在某个随机点处的值是不是正确。依据 [上一章节](#Polynomial-commitment-using-inner-product-argument) 我们知道，该检查仅需要 $\log d$ 的工作。<br><br> 在检查 $\pi$ 和 $G$ 的最后阶段，验证电路将产生 $G'$，和 $u_1', \cdots, u_k'$ 挑战。为了彻底证明 $\pi$ 合法的，我们需要计算 $G' = \langle\mathbf{G}, \mathbf{s}'\rangle$，这是一个线性时间复杂度的计算。接下来，我们通过将 $G'$ 作为电路的witness和公开 $G, u_1, \cdots, u_k$ 做为证明 $\pi$ 的公开输入推迟了这个计算。<br><br> 由此，我们可以不断地从一个证明到另外一个证明，直到满足了我们需要的证明的个数，该过程才会中止。我们只需在最后做一次线性时间复杂度的计算，就可以证明之前产生的所有证明都是合法的。  |

依据上文所述 [曲线的循环](curves.md#cycles-of-curves)，我们可以用2-cycle的曲线来实现一个协议，这就是说，在一条曲线上产生证明，
而在另一条曲线的电路中进行验证。但是又有某些验证计算在原有的曲线上（证明产生的曲线）效率更高；这些就被“推迟”到下一个本地电路（请看下图）
而不需要立即送到另外一条曲线上。

![](https://i.imgur.com/l4HrYgE.png)
