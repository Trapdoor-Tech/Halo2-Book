# 电路承诺

## 对电路赋值进行承诺

在生成证明之前，证明者已经有了一张能够满足约束条件的单元格赋值表。赋值表有 $n = 2^k$ 行；其列被划分为三类： advice, instance, fixed 。我们定义 $F_{i,j}$ 表示第 $j$ 行的第 $i$ 个fixed列，不失一般性，定义 $A_{i, j}$ 为第 $j$ 行的第 $i$ 个advice或instance列。

> 我们区分开fixed 列的原因是它们是由验证者提供的，而advice 列和instance 列是由证明者提供的。实际上，证明者和验证者都需要计算instance 列和fixed 列的承诺，只有对advice 列的承诺在证明中。

为对表中的各个赋值进行承诺，我们为每列构造一个次数为 $n - 1$ 的拉格朗日多项式，其取值域大小为 $n$ （令 $\omega$ 为其 $n$ 次单位根）。

- 令 $a_i(\omega^j) = A_{i,j}$ ，通过插值得到 $a_i(X)$ 多项式
- 令 $f_i(\omega^j) = F_{i,j}$ ，通过插值得到 $f_i(X)$ 多项式

然后对每列的多项式创建一个盲化承诺（blinding commitment） ：

$$\mathbf{A} = [\text{Commit}(a_0(X)), \dots, \text{Commit}(a_i(X))]$$
$$\mathbf{F} = [\text{Commit}(f_0(X)), \dots, \text{Commit}(f_i(X))]$$

其中 $\mathbf{F}$ 在生成密钥的时候被创建，使用 $1$ 作为盲因子。
$\mathbf{A}$ 由证明者计算并发送给验证者 。


## 对查找表置换进行承诺

验证者随机选择一个 $\theta$ ，用于确保同一个查找表内不同的列线性无关。接着证明者对每个查找表的置换进行承诺

- 给定一个查找表，其输入列多项式为 $[A_0(X), \dots, A_{m-1}(X)]$ ，其真值表列的多项式为 $[S_0(X), \dots, S_{m-1}(X)]$, 证明者构造两个压缩后的多项式

  $$A_\text{compressed}(X) = \theta^{m-1} A_0(X) + \theta^{m-2} A_1(X) + \dots + \theta A_{m-2}(X) + A_{m-1}(X)$$
  $$S_\text{compressed}(X) = \theta^{m-1} S_0(X) + \theta^{m-2} S_1(X) + \dots + \theta S_{m-2}(X) + S_{m-1}(X)$$

- 之后证明者对 $A_\text{compressed}(X)$ 和 $S_\text{compressed}(X)$ 依照 [查找表证明](lookup.md) 的规则进行排列，得到 $A'(X)$ 和 $S'(X)$.

证明者为所有的查找表创建盲化承诺：

$$\mathbf{L} = \left[ (\text{Commit}(A'(X))), \text{Commit}(S'(X))), \dots \right]$$

并发送给验证者 。

当验证者接收到 $\mathbf{A}$ ， $\mathbf{F}$ ，和 $\mathbf{L}$ 后，随机采样 $\beta$ 和 $\gamma$ 作为挑战，用于后续的置换和查找表证明验证。（随机采样是可以重复利用的，因为证明是相互独立的）。

## 对相等约束置换进行承诺

令 $c$ 为 相等约束所涉及的列数。

令 $m$ 为能容纳的 [最大列数](permutation.md#spanning-a-large-number-of-columns) ， $m$ 不能超过 PLONK 配置中多项式的次数上限。

令 $u$ 为 [置换证明](permutation.md#zero-knowledge-adjustment) 章节中定义的可以“使用”的行数。

令 $b = \mathsf{ceiling}(c/m).$

证明者构造一个长度为 $bu$ 的向量 $\mathbf{P}$ ，对于每个列集合，有， $0 \leq a < b$ ，对于每一行有 $0 \leq j < u$

$$
\mathbf{P}_{au + j} = \prod\limits_{i=am}^{\min(c, (a+1)m)-1} \frac{v_i(\omega^j) + \beta \cdot \delta^i \cdot \omega^j + \gamma}{v_i(\omega^j) + \beta \cdot s_i(\omega^j) + \gamma}.
$$

证明者计算 $\mathbf{P}$ 的”滚动乘积“, 从 $1$ 开始，并且多项式向量 $Z_{P,0..b-1}$ 每个都有拉格朗日基表示，基于滚动乘积的长度为 $u$ 的切片，如 [置换证明](permutation.md#argument-specification) 章节中所描述的。

最后 证明者为每个 $Z_{P,a}$ 多项式创建盲化承诺：

$$\mathbf{Z_P} = \left[\text{Commit}(Z_{P,0}(X)), \dots, \text{Commit}(Z_{P,b-1}(X))\right]$$

并发送给 验证者 。

## 对查找表置换进行承诺

除了需要对单独的相等约束进行承诺外，对每一个查找表， 证明者也需要对置换进行承诺。

- 证明者构造一个向量 $P$:

$$
P_j = \frac{(A_\text{compressed}(\omega^j) + \beta)(S_\text{compressed}(\omega^j) + \gamma)}{(A'(\omega^j) + \beta)(S'(\omega^j) + \gamma)}
$$

- 证明者构造多项式 $Z_L$ ，其拉格朗日基表示为 $P$ 多项式的“滚动乘积”，从 $Z_L(1) = 1$ 开始。

$\beta$ 和 $\gamma$ 用于在组合 $A'(X)$ 和 $S'(X)$ 的同时保持这两者无关。 $\beta$ 和 $\gamma$ 是验证者在证明者创建多项式 $\mathbf{A}$ ， $\mathbf{F}$ ，和 $\mathbf{L}$ 之后采样的（因此承诺了在 查找表列中用到的单元格的值，以及每个 查找表的 $A'(X)$ 和 $S'(X)$ ）。

如之前一样， 证明者对每个 $Z_L$ 多项式都创建一个盲化承诺并发送给 验证者 ：

$$\mathbf{Z_L} = \left[\text{Commit}(Z_L(X)), \dots \right]$$
