# Vanishing argument

# 消除多项式

Having committed to the circuit assignments, the prover now needs to demonstrate that the
various circuit relations are satisfied:

在电路的所有赋值都已经承诺之后，证明者现在需要证明各种各样的电路关系都是满足的：

- The custom gates, represented by polynomials $\text{gate}_i(X)$.
- The rules of the lookup arguments.
- The rules of the equality constraint permutations.

- custom gates，用多项式 $\text{gate}_i(X)$ 表示。
- 查找证明的规则
- 相等约束置换的规则

Each of these relations is represented as a polynomial of degree $d$ (the maximum degree
of any of the relations) with respect to the circuit columns. Given that the degree of the
assignment polynomials for each column is $n - 1$, the relation polynomials have degree
$d(n - 1)$ with respect to $X$.

每个关系都由一个 $d$ 次多项式表示（在所有关系中最大的的次数），该关系对应于某些电路的列。假设对应一列的 assignment 多项式的次数是 $n - 1$，那么关系多项式的 $X$ 的次数就是 $d(n - 1)$

> In our [example](../proving-system.md#example), these would be the gate polynomials, of
> degree $3n - 3$:

> 在我们的[例子](../proving-system.md#example)中，门多项式的次数是 $3n - 3$。
>
> - $\text{gate}_0(X) = a_0(X) \cdot a_1(X) \cdot a_2(X \omega^{-1}) - a_3(X)$
> - $\text{gate}_1(X) = f_0(X \omega^{-1}) \cdot a_2(X)$
> - $\text{gate}_2(X) = f_0(X) \cdot a_3(X) \cdot a_0(X)$

A relation is satisfied if its polynomial is equal to zero. One way to demonstrate this is
to divide each polynomial relation by the vanishing polynomial $t(X) = (X^n - 1)$, which
is the lowest-degree monomial that has roots at every $\omega^i$. If relation's polynomial
is perfectly divisible by $t(X)$, it is equal to zero over the domain (as desired).

如果描述关系的多项式为 $0$，我们就说关系是满足的。一种表示这种此种约束的方法，就是将每个多项式关系都除以消除多项式 $t(X) = (X^n - 1)$，消除多项式是以 $\omega^i$ 作为根的最低次数的单项式。如果关系多项式能被 $t(X) 整除，那么在域上这个关系多项式就等于 $0$。

This simple construction would require a polynomial commitment per relation. Instead, we
commit to all of the circuit relations simultaneously: the verifier samples $y$, and then
the prover constructs the quotient polynomial

这种朴素的构造方式的弊端在于，其需要对每个关系都需要有一个多项式承诺。相对的，我们可以对电路中所有关系同时进行承诺：验证者取一个 $y$，然后证明者构造一个商多项式，

$$h(X) = \frac{\text{gate}_0(X) + y \cdot \text{gate}_1(X) + \dots + y^i \cdot \text{gate}_i(X) + \dots}{t(X)},$$

where the numerator is a random (the prover commits to the cell assignments before the
verifier samples $y$) linear combination of the circuit relations.

其中分子是电路关系的随机的线性组合（证明者需要在验证者给出 $y$ 之前先承诺cell assignments）。

- If the numerator polynomial (in formal indeterminate $X$) is perfectly divisible by
  $t(X)$, then with high probability all relations are satisfied.
- Conversely, if at least one relation is not satisfied, then with high probability
  $h(x) \cdot t(x)$ will not equal the evaluation of the numerator at $x$. In this case,
  the numerator polynomial would not be perfectly divisible by $t(X)$.

- 如果分子多项式（以 $X$ 为自变量）能够被 $t(X)$ 整除，那么所有的关系都满足的概率就是非常高的
- 相反地，哪怕只有一个关系是不满足的，那么在点 $x$ 处，$h(x) \cdot t(x)$ 与分子多项式在该处的值就几乎不可能相等。也就是说，在此种情况下，分子多项式不能整除 $t(X)$。

## Committing to $h(X)$

## 承诺 $h(X)$

$h(X)$ has degree $(d - 1)n - d$ (because the divisor $t(X)$ has degree $n$). However, the
polynomial commitment scheme we use for Halo 2 only supports committing to polynomials of
degree $n - 1$ (which is the maximum degree that the rest of the protocol needs to commit
to). Instead of increasing the cost of the polynomial commitment scheme, the prover split
$h(X)$ into pieces of degree $n - 1$

$h(X)$ 的次数是 $(d - 1)n - d$ （因为分母 $t(X)$ 的次数是 $n$）。但是我们在 Halo 2 中的多项式承诺机制仅仅支持次数 $n - 1$ 次的多项式的承诺（这是因为除了关系多项式的承诺，协议其他部分所需承诺的多项式的最大次数就是 $n - 1$）。为了不给多项式承诺机制增加额外的成本，验证者就需要把 $h(X)$ 分割成多个，每个只有 $n - 1$ 次的多项式的和，

$$h_0(X) + X^n h_1(X) + \dots + X^{n(d-1)} h_{d-1}(X),$$

and produces blinding commitments to each piece

并对每一部分产生一个具有隐藏性的承诺。

$$\mathbf{H} = [\text{Commit}(h_0(X)), \text{Commit}(h_1(X)), \dots, \text{Commit}(h_{d-1}(X))].$$

## Evaluating the polynomials

## 多项式求值

At this point, all properties of the circuit have been committed to. The verifier now
wants to see if the prover committed to the correct $h(X)$ polynomial. The verifier
samples $x$, and the prover produces the purported evaluations of the various polynomials
at $x$, for all the relative offsets used in the circuit, as well as $h(X)$.

到这一步，电路所有的特点都已经被承诺了。现在，验证者想要验证一下证明者提供的承诺是不是正确的 $h(X)$。于是，验证者选取一个 $x$，证明者需要提供其所声称的所有多项式在 $x$ 处的值，包括电路中使用的所有 relative offsets 和 $h(X)$的值。

> In our [example](../proving-system.md#example), this would be:
> 在我们的 [例子]](../proving-system.md#example)中，这就是：
>
> - $a_0(x)$
> - $a_1(x)$
> - $a_2(x)$, $a_2(x \omega^{-1})$
> - $a_3(x)$
> - $f_0(x)$, $f_0(x \omega^{-1})$
> - $h_0(x)$, ..., $h_{d-1}(x)$

The verifier checks that these evaluations satisfy the form of $h(X)$:

验证者现在要验证这些值是不是满足 $h(X)$ 的形式：

$$\frac{\text{gate}_0(x) + \dots + y^i \cdot \text{gate}_i(x) + \dots}{t(x)} = h_0(x) + \dots + x^{n(d-1)} h_{d-1}(x)$$

Now content that the evaluations collectively satisfy the gate constraints, the verifier
needs to check that the evaluations themselves are consistent with the original
[circuit commitments](circuit-commitments.md), as well as $\mathbf{H}$. To implement this
efficiently, we use a [multipoint opening argument](multipoint-opening.md).

如果这些值确实满足的门约束，那么验证者接下来就需要验证这些值本身与最初的[电路承诺](circuit-commitments.md)以及承诺 $\mathbf{H}$ 是否一致。为了高效实现此一目的，我们使用了[多点打开证明](multipoint-opening.md)。