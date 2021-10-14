## Recursion

## 递归
> Alternative terms: Induction; Accumulation scheme; Proof-carrying data

However, the computation of $G$ requires a length-$2^k$ multiexponentiation
$\langle \mathbf{G}, \mathbf{s}\rangle,$ where $\mathbf{s}$ is composed of the round
challenges $u_1, \cdots, u_k$ arranged in a binary counting structure. This is the
linear-time computation that we want to amortise across a batch of proof instances.
Instead of computing $G,$ notice that we can express $G$ as a commitment to a polynomial

计算 $G$ 需要计算长度 $2^k$ multiexp
$\langle \mathbf{G}, \mathbf{s}\rangle,$ 其中 $\mathbf{s}$ 每一轮挑战 $u_1, \cdots, u_k$ 按照二进制查数的顺序进行排列组成的。
This is the linear-time computation that we want to amortise across a batch of proof instances.（？？？）
不计算 $G,$ 我们也可以将 $G$ 表示成一个多项式的承诺

$$G = \text{Commit}(\sigma, g(X, u_1, \cdots, u_k)),$$

where $g(X, u_1, \cdots, u_k) := \prod_{i=1}^k (u_i + u_i^{-1}X^{2^{i-1}})$ is a
polynomial with degree $2^k - 1.$ 

其中 $g(X, u_1, \cdots, u_k) := \prod_{i=1}^k (u_i + u_i^{-1}X^{2^{i-1}})$ 是一个 $2^k - 1$ 次多项式。 

 
|  |  | 
| -------- | -------- | 
| <img src="https://i.imgur.com/vMXKFDV.png" width=1900> | Since $G$ is a commitment, it can be checked in an inner product argument. The verifier circuit witnesses $G$ and brings $G, u_1, \cdots, u_k$ out as public inputs to the proof $\pi.$ The next verifier instance checks $\pi$ using the inner product argument; this includes checking that $G = \text{Commit}(g(X, u_1, \cdots, u_k))$ evaluates at some random point to the expected value for the given challenges $u_1, \cdots, u_k.$ Recall from the [previous section](#Polynomial-commitment-using-inner-product-argument) that this check only requires $\log d$ work. <br><br> At the end of checking $\pi$ and $G,$ the circuit is left with a new $G',$ along with the $u_1', \cdots, u_k'$ challenges sampled for the check. To fully accept $\pi$ as valid, we should perform a linear-time computation of $G' = \langle\mathbf{G}, \mathbf{s}'\rangle$. Once again, we delay this computation by witnessing $G'$ and bringing $G, u_1, \cdots, u_k$ out as public inputs to the proof $\pi.$ <br><br> This goes on from one proof instance to the next, until we are satisfied with the size of our batch of proofs. We finally perform a single linear-time computation, thus deciding the validity of the whole batch.   |

|  |  | 
| -------- | -------- | 
| <img src="https://i.imgur.com/vMXKFDV.png" width=1900> | 由于 $G$ 是一个承诺，它就可以在内积证明中被验证。验证电路的 witnesses $G$ 并产生 $G, u_1, \cdots, u_k$ 做为证明 $\pi$ 的公开输入。下一个验证者将运用内积证明来检查 $\pi$ ；这包括对给定的挑战 $u_1, \cdots, u_k.$ ，查验 $G = \text{Commit}(g(X, u_1, \cdots, u_k))$ 在某个随机点处的值是不是正确。依据 [上一章节](#Polynomial-commitment-using-inner-product-argument) 我们知道，该检查仅需要 $\log d$ 的工作。<br><br> 在检查 $\pi$ 和 $G$ 的最后阶段，验证电路将产生 $G'$，和 $u_1', \cdots, u_k'$ 挑战。为了彻底证明 $\pi$ 合法的，我们需要计算 $G' = \langle\mathbf{G}, \mathbf{s}'\rangle$，这是一个线性时间复杂度的计算。接下来，我们通过将 $G'$ 作为电路的witness和公开 $G, u_1, \cdots, u_k$ 做为证明 $\pi$ 的公开输入推迟了这个计算。<br><br> 由此，我们可以不断地从一个证明到另外一个证明，直到满足了我们需要的证明的个数，该过程才会中止。我们只需在最后做一次线性时间复杂度的计算，就可以证明之前产生的所有证明都是合法的。  |

We recall from the section [Cycles of curves](curves.md#cycles-of-curves) that we can
instantiate this protocol over a two-cycle, where a proof produced by one curve is
efficiently verified in the circuit of the other curve. However, some of these verifier
checks can actually be efficiently performed in the native circuit; these are "deferred"
to the next native circuit (see diagram below) instead of being immediately passed over to
the other curve. 

依据上文所述 [曲线的循环](curves.md#cycles-of-curves)，我们可以用2-cycle的曲线来实现一个协议，这就是说，在一条曲线上产生证明，
而在另一条曲线的电路中进行验证。但是又有某些验证计算在原有的曲线上（证明产生的曲线）效率更高；这些就被“推迟”到下一个本地电路（请看下图）
而不需要立即送到另外一条曲线上。

![](https://i.imgur.com/l4HrYgE.png)
