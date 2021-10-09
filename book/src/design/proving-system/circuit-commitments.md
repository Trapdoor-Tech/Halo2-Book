# 电路承诺

## 对电路赋值进行承诺

在生成证明之前，prover 已经有了一张能够满足约束条件的 cell 赋值表。赋值表有 $n = 2^k$ 行；其列被划分为三类： advice, instance, fixed 。我们定义 $F_{i,j}$ 表示第 $j$ 行的第 $i$ 个fixed列，不失一般性，定义 $A_{i, j}$ 为第 $j$ 行的第 $i$ 个advice或instance列。

> 我们区分开fixed 列的原因是它们是由verifier 提供的，而advice 列和instance 列是由prover 提供的。实际上，对instance 列和fixed 列的承诺由prover 和verifier 直接计算得出，只有对advice 列的承诺会存放在证明中。

为对表中的各个赋值进行承诺，我们为每列构造一个度数为 $n - 1$ 的拉格朗日多项式，其 evaluation domain 大小为 $n$ （令 $\omega$ 为其 $n$ 次单位根）。

- 令 $a_i(\omega^j) = A_{i,j}$ ，通过插值得到 $a_i(X)$ 多项式
- 令 $f_i(\omega^j) = F_{i,j}$ ，通过插值得到 $f_i(X)$ 多项式

然后对每列的多项式创建一个 blinding commitment ：

$$\mathbf{A} = [\text{Commit}(a_0(X)), \dots, \text{Commit}(a_i(X))]$$
$$\mathbf{F} = [\text{Commit}(f_0(X)), \dots, \text{Commit}(f_i(X))]$$

其中 $\mathbf{F}$ 在生成密钥的时候被创建，使用 $1$ 作为盲因子。
$\mathbf{A}$ 由prover 计算并发送给verifier 。


## 对 lookup permutation 进行承诺

verifier 随机选择一个 $\theta$ ，用于确保同一个lookup 内不同的列线性无关。接着 prover 对每个lookup 的permutation 进行承诺

- 给定一个 lookup ，其input列多项式为 $[A_0(X), \dots, A_{m-1}(X)]$ ，其真值表列的多项式为 $[S_0(X), \dots, S_{m-1}(X)]$, prover 构造两个压缩后的多项式

  $$A_\text{compressed}(X) = \theta^{m-1} A_0(X) + \theta^{m-2} A_1(X) + \dots + \theta A_{m-2}(X) + A_{m-1}(X)$$
  $$S_\text{compressed}(X) = \theta^{m-1} S_0(X) + \theta^{m-2} S_1(X) + \dots + \theta S_{m-2}(X) + S_{m-1}(X)$$

- 之后 prover 对 $A_\text{compressed}(X)$ 和 $S_\text{compressed}(X)$ 依照 [rules of the lookup argument](lookup.md) 的规则进行排列，得到 $A'(X)$ 和 $S'(X)$.

prover 为所有的lookup 创建blinding commitment ：

$$\mathbf{L} = \left[ (\text{Commit}(A'(X))), \text{Commit}(S'(X))), \dots \right]$$

并发送给 verifier 。

当 verifier 接收到 $\mathbf{A}$ ， $\mathbf{F}$ ，和 $\mathbf{L}$ 后，随机采样 $\beta$ 和 $\gamma$ 作为挑战，用于后续的permutation 和 lookup证明验证。（随机采样是可以重复利用的，因为证明是相互独立的）。

## 对equality permutation 约束进行承诺

令 $c$ 为 equality 约束所涉及的列数。

令 $m$ 为最大能容纳一个 [column set](permutation.md#spanning-a-large-number-of-columns) 的列数， $m$ 不能超过 PLONK 配置中多项式的度数上限。

令 $u$ 为 [Permutation argument](permutation.md#zero-knowledge-adjustment) 章节中定义的可以“使用”的行数。

令 $b = \mathsf{ceiling}(c/m).$

prover 构造一个长度为 $bu$ 的向量 $\mathbf{P}$ ，对于每个 column set ，有， $0 \leq a < b$ ，对于每一行有 $0 \leq j < u,$

## 对 lookup permutation product 列进行承诺

除了需要对单独的 permuted lookup 进行承诺外，对每一个 lookup ， prover 也需要对 permutation product 列进行承诺。

- prover 构造一个向量 $P$:

$$
P_j = \frac{(A_\text{compressed}(\omega^j) + \beta)(S_\text{compressed}(\omega^j) + \gamma)}{(A'(\omega^j) + \beta)(S'(\omega^j) + \gamma)}
$$

- prover 构造多项式 $Z_L$ ，其拉格朗日基表示为 $P$ 多项式的“滚动乘积”，从 $Z_L(1) = 1$ 开始。

$\beta$ 和 $\gamma$ 用于在组合 $A'(X)$ 和 $S'(X)$ 的 permutation argument 的同时保持这两者无关。 $\beta$ 和 $\gamma$ 是 verifier 在 prover 创建多项式 $\mathbf{A}$ ， $\mathbf{F}$ ，和 $\mathbf{L}$ 之后采样的（因此承诺了在 lookup 列中用到的 cell value ，以及
每个 lookup 的 $A'(X)$ 和 $S'(X)$ ）。

如之前一样， prover 对每个 $Z_L$ 多项式都创建一个 blinding commitment 并发送给 verifier ：

$$\mathbf{Z_L} = \left[\text{Commit}(Z_L(X)), \dots \right]$$
