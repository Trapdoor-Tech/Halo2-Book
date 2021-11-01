# Permutation argument

# 置换证明

Given that gates in halo2 circuits operate "locally" (on cells in the current row or
defined relative rows), it is common to need to copy a value from some arbitrary cell into
the current row for use in a gate. This is performed with an equality constraint, which
enforces that the source and destination cells contain the same value.

假设halo2电路中的门要操作"locally" （就是在当前行或者事先定义好的相关行进行操作）
这通常需要将某个任意cell处的值拷贝到当前行，以便在门中使用。该工作就由一条相等约束来描述，
该约束就强制要求源和目的cell中包含相同的值。

We implement these equality constraints by constructing a permutation that represents the
constraints, and then using a permutation argument within the proof to enforce them.

我们通过构造一个代表这些约束的置换来实现这些相等约束，进而在最终的证明中包含一个置换证明来强制执行(????)。

## Notation

## 相关记号

A permutation is a one-to-one and onto mapping of a set onto itself. A permutation can be
factored uniquely into a composition of cycles (up to ordering of cycles, and rotation of
each cycle).

置换就是一个集合以一对一的方式自己到自己的一个映射。一个置换可以唯一分解成多个轮换的组合（每个轮换都是首尾相接的，up to ordering of cycles）。

We sometimes use [cycle notation](https://en.wikipedia.org/wiki/Permutation#Cycle_notation)
to write permutations. Let $(a\ b\ c)$ denote a cycle where $a$ maps to $b,$ $b$ maps to
$c,$ and $c$ maps to $a$ (with the obvious generalization to arbitrary-sized cycles).
Writing two or more cycles next to each other denotes a composition of the corresponding
permutations. For example, $(a\ b)\ (c\ d)$ denotes the permutation that maps $a$ to $b,$
$b$ to $a,$ $c$ to $d,$ and $d$ to $c.$

通常，我们使用 [轮换记号](https://en.wikipedia.org/wiki/Permutation#Cycle_notation)
来表示置换。令 $(a\ b\ c)$ 代表如下轮换：$a$ 映射到 $b$，$b$ 映射到 $c$，$c$ 映射到 $a$
（先然，该法则可以推广到任意大小的轮换）。把两个或更多的轮换一个接一个的写在一起就代表了一个相关置换的组合。
比如， $(a\ b)\ (c\ d)$ 就代表了 $a$ 映射到 $b$，$b$ 映射到 $a$，$c$ 映射到 $d$，$d$ 映射到 $c$ 的置换。

## Constructing the permutation

## 构造置换

### Goal

### 目标

We want to construct a permutation in which each subset of variables that are in a
equality-constraint set form a cycle. For example, suppose that we have a circuit that
defines the following equality constraints:

我们希望构造这样一个置换，在同一个相等约束下的这些变量自己形成一个轮换。举例来说，假设我们有一个定义了如下相等约束的电路：

- $a \equiv b$
- $a \equiv c$
- $d \equiv e$

From this we have the equality-constraint sets $\{a, b, c\}$ and $\{d, e\}.$ We want to
construct the permutation:

由上可得相等约束集合 $\{a, b, c\}$ 和 $\{d, e\}$。于是我们构造了如下的置换：

$$(a\ b\ c)\ (d\ e)$$

该置换就定义了一个由 $[a, b, c, d, e]$ 到 $[b, c, a, e, d]$ 的一个映射。

### Algorithm

### 算法

We need to keep track of the set of cycles, which is a
[set of disjoint sets](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).
Efficient data structures for this problem are known; for the sake of simplicity we choose
one that is not asymptotically optimal but is easy to implement.

我们需要持续记录这些轮换的集合，实际上也就是
[一些不相交集合的集合](https://en.wikipedia.org/wiki/Disjoint-set_data_structure).
我们已经知道针对该问题的一种高效的数据结构；为了简单起见，我们选择了一个虽然不是渐进最优，但是确是很容易实现的方法。

We represent the current state as:

我们采取如下方法来表示当前的状态：

- an array $\mathsf{mapping}$ for the permutation itself;

- 数组 $\mathsf{mapping}$ 来表示置换本身；

- an auxiliary array $\mathsf{aux}$ that keeps track of a distinguished element of each
  cycle;

- 一个辅助的数组 $\mathsf{aux}$ 用来记录每个轮换中每个元素的代表元素；
  
- another array $\mathsf{sizes}$ that keeps track of the size of each cycle.

- 还有一个数组 $\mathsf{sizes}$ 用来记录每个轮换的元素个数。

We have the invariant that for each element $x$ in a given cycle $C,$ $\mathsf{aux}(x)$
points to the same element $c \in C.$ This allows us to quickly decide whether two given
elements $x$ and $y$ are in the same cycle, by checking whether
$\mathsf{aux}(x) = \mathsf{aux}(y).$ Also, $\mathsf{sizes}(\mathsf{aux}(x))$ gives the
size of the cycle containing $x.$ (This is guaranteed only for
$\mathsf{sizes}(\mathsf{aux}(x)),$ not for $\mathsf{sizes}(x).$)

我们在一个给定的轮换 $C$ 中选取一个不变的值 $c$，对每一个 $C$ 中的元素 $x$，$\mathsf{aux}(x)$
都得到相同的值，即 $c \in C$。有了这个值，对于给定的两个给定的元素 $x$ 和 $y$，我们可以通过检查 $\mathsf{aux}(x) = \mathsf{aux}(y)$ 是否成立，来快速判断它们是否在同一个轮换中。$\mathsf{sizes}(\mathsf{aux}(x))$ 也代表了包含 $x$ 的轮换的大小。（只有 $\mathsf{sizes}(\mathsf{aux}(x)) 是有保证的，而不是 $\mathsf{sizes}(x)$。？？？）


The algorithm starts with a representation of the identity permutation:
for all $x,$ we set $\mathsf{mapping}(x) = x,$ $\mathsf{aux}(x) = x,$ and
$\mathsf{sizes}(x) = 1.$

本算法是以表示一个单位置换开始：
对所有 $x$，我们令 $\mathsf{mapping}(x) = x$，$\mathsf{aux}(x) = x$，和
$\mathsf{sizes}(x) = 1$。

To add an equality constraint $\mathit{left} \equiv \mathit{right}$:

按如下步骤增加一条相等约束 $\mathit{left} \equiv \mathit{right}$：

1. Check whether $\mathit{left}$ and $\mathit{right}$ are already in the same cycle, i.e.
   whether $\mathsf{aux}(\mathit{left}) = \mathsf{aux}(\mathit{right}).$ If so, there is
   nothing to do.

1. 检查 $\mathit{left}$ 和 $\mathit{right}$ 是否已经在同一个轮换中了，也就是检查 $\mathsf{aux}(\mathit{left}) = \mathsf{aux}(\mathit{right})$
   是否成立。如果是此种情况，那么不许做任何事情。

2. Otherwise, $\mathit{left}$ and $\mathit{right}$ belong to different cycles. Make
   $\mathit{left}$ the larger cycle and $\mathit{right}$ the smaller one, by swapping them
   iff $\mathsf{sizes}(\mathsf{aux}(\mathit{left})) < \mathsf{sizes}(\mathsf{aux}(\mathit{right})).$

2. 否则，$\mathit{left}$ 和 $\mathit{right}$ 就一定分属不同的轮换。令 $\mathit{left}$ 是相对大的轮换，而 $\mathit{right}$ 则为相对小的那个，
   如果 $\mathsf{sizes}(\mathsf{aux}(\mathit{left})) < \mathsf{sizes}(\mathsf{aux}(\mathit{right}))$，那我们就交换一下使其满足需求。

3. Set $\mathsf{sizes}(\mathsf{aux}(\mathit{left})) :=
        \mathsf{sizes}(\mathsf{aux}(\mathit{left})) + \mathsf{sizes}(\mathsf{aux}(\mathit{right})).$

3. 令 $\mathsf{sizes}(\mathsf{aux}(\mathit{left})) :=
        \mathsf{sizes}(\mathsf{aux}(\mathit{left})) + \mathsf{sizes}(\mathsf{aux}(\mathit{right}))$。

4. Following the mapping around the right (smaller) cycle, for each element $x$ set
   $\mathsf{aux}(x) := \mathsf{aux}(\mathit{left}).$

4. 下一步对right（较小的）轮换中每一个元素 $x$ 令
   $\mathsf{aux}(x) := \mathsf{aux}(\mathit{left})$。

5. Splice the smaller cycle into the larger one by swapping $\mathsf{mapping}(\mathit{left})$
   with $\mathsf{mapping}(\mathit{right}).$

5. 通过交换 $\mathsf{mapping}(\mathit{left})$ 和 $\mathsf{mapping}(\mathit{right})$ 中的元素（的轮换），将较小的轮换接入到较大的轮换中去。

For example, given two disjoint cycles $(A\ B\ C\ D)$ and $(E\ F\ G\ H)$:

举个例子，假设两个不相交的轮换 $(A\ B\ C\ D)$ 和 $(E\ F\ G\ H)$：

```plaintext
A +---> B
^       +
|       |
+       v
D <---+ C       E +---> F
                ^       +
                |       |
                +       v
                H <---+ G
```

After adding constraint $B \equiv E$ the above algorithm produces the cycle:

在增加约束 $B \equiv E$ 后，上文算法将得出如下的轮换：

```plaintext
A +---> B +-------------+
^                       |
|                       |
+                       v
D <---+ C <---+ E       F
                ^       +
                |       |
                +       v
                H <---+ G
```

### Broken alternatives

If we did not check whether $\mathit{left}$ and $\mathit{right}$ were already in the same
cycle, then we could end up undoing an equality constraint. For example, if we have the
following constraints:

如果步检查 $\mathit{left}$ 和 $\mathit{right}$ 是否在同一个轮换中，那么我们可能就会丢掉某些相等约束。
举个例子，如果我们有如下的约束:

- $a \equiv b$
- $b \equiv c$
- $c \equiv d$
- $b \equiv d$

and we tried to implement adding an equality constraint just using step 5 of the above
algorithm, then we would end up constructing the cycle $(a\ b)\ (c\ d),$ rather than the
correct $(a\ b\ c\ d).$

如果在处理一条新的相等约束时仅仅只执行上述算法中的第五步，那么我们得到的最终结果将是 $(a\ b)\ (c\ d)$，而不是正确的结果 $(a\ b\ c\ d)$。

## Argument specification

## 证明说明

We need to check a permutation of cells in $m$ columns, represented in Lagrange basis by
polynomials $v_0, \ldots, v_{m-1}.$

我们需要检查 $m$ 列中所有cells的置换，该 $m$ 列分别由其拉格朗日基的多项式 $v_0, \ldots, v_{m-1}$ 来表示。

We will label *each cell* in those $m$ columns with a unique element of $\mathbb{F}^\times.$

我们用 $\mathbb{F}^\times$ 中不同的元素来标记 $m$ 列中 *每个cell*。 

Suppose that we have a permutation on these labels,

假设我们有几个基于这些标记的一个置换：
$$
\sigma(\mathsf{column}: i, \mathsf{row}: j) = (\mathsf{column}: i', \mathsf{row}: j').
$$
in which the cycles correspond to equality-constraint sets.

其中这些轮换就对应了相等约束。

> If we consider the set of pairs $\{(\mathit{label}, \mathit{value})\}$, then the values within
> each cycle are equal if and only if permuting the label in each pair by $\sigma$ yields the
> same set:
>
> ![An example for a cycle (A B C D). The set before permuting the labels is {(A, 7), (B, 7), (C, 7), (D, 7)}, and the set after is {(D, 7), (A, 7), (B, 7), (C, 7)} which is the same. If one of the 7s is replaced by 3, then the set after permuting labels is not the same.](./permutation-diagram.png)
>
> Since the labels are distinct, set equality is the same as multiset equality, which we can
> check using a product argument.

> 我们考虑对 $\{(\mathit{label}, \mathit{value})\}$ 的集合，当且仅当对每对的label都按照 $\sigma$ 进行置换后，得到了与原集合相同的集合， 
> 才能证明集合中的每个对的值(value)是相等的。
>
> ![An example for a cycle (A B C D). The set before permuting the labels is {(A, 7), (B, 7), (C, 7), (D, 7)}, and the set after is {(D, 7), (A, 7), (B, 7), (C, 7)} which is the same. If one of the 7s is replaced by 3, then the set after permuting labels is not the same.](./permutation-diagram.png)
>
> 因为所有的标记都是不同的，最终就可以用一个代表相等约束的集合来代替多个代表相等关系的集合，而这进一步使我们可以用乘积证明来检查置换的正确性。

Let $\omega$ be a $2^k$ root of unity and let $\delta$ be a $T$ root of unity, where
${T \cdot 2^S + 1 = p}$ with $T$ odd and ${k \leq S.}$
We will use $\delta^i \cdot \omega^j \in \mathbb{F}^\times$ as the label for the
cell in the $j$th row of the $i$th column of the permutation argument.

令 $\omega$ 是 $2^k$ 次单位根，$\delta$ 是 $T$ 次单位根，满足 ${T \cdot 2^S + 1 = p}$，其中 $T$ 是奇数并且 ${k \leq S}$。
在置换证明中，我们用 $\delta^i \cdot \omega^j \in \mathbb{F}^\times$ 作为第 $j$ 行、第 $i$ 列的cell的标记。

We represent $\sigma$ by a vector of $m$ polynomials $s_i(X)$ such that
$s_i(\omega^j) = \delta^{i'} \cdot \omega^{j'}.$

$\sigma$ 由 $m$ 个多项式组成的一个向量，其中对每一项 $s_i(X)$ 满足 $s_i(\omega^j) = \delta^{i'} \cdot \omega^{j'}$。

Notice that the identity permutation can be represented by the vector of $m$ polynomials
$\mathsf{ID}_i(\omega^j)$ such that $\mathsf{ID}_i(\omega^j) = \delta^i \cdot \omega^j.$

注意，可以用一个包含 $m$ 个多项式的向量来标识一个置换，其中每一项 $\mathsf{ID}_i(\omega^j)$ 都满足 $\mathsf{ID}_i(\omega^j) = \delta^i \cdot \omega^j$。

We will use a challenge $\beta$ to compress each ${(\mathit{label}, \mathit{value})}$ pair
to $\mathit{value} + \beta \cdot \mathit{label}.$ Just as in the product argument we used
for [lookups](lookup.md), we also use a challenge $\gamma$ to randomize each term of the
product.

我们用一个挑战 $\beta$ 将每一个 ${(\mathit{label}, \mathit{value})}$ 对压缩成 $\mathit{value} + \beta \cdot \mathit{label}$。与在 [lookups](lookup.md) 的积证明中所做的一样，我们也使用一个挑战 $\gamma$ 来盲化我们每一个积项。

Now given our permutation represented by $s_0, \ldots, s_{m-1}$ over columns represented by
$v_0, \ldots, v_{m-1},$ we want to ensure that:

假定有一个在 $m$ 列，即：$v_0, \ldots, v_{m-1}$ 上的一个置换，表示为 $s_0, \ldots, s_{m-1}$，我们想要确保如下等式成立：
$$
\prod\limits_{i=0}^{m-1} \prod\limits_{j=0}^{n-1} \left(\frac{v_i(\omega^j) + \beta \cdot \delta^i \cdot \omega^j + \gamma}{v_i(\omega^j) + \beta \cdot s_i(\omega^j) + \gamma}\right) = 1
$$

> Here ${v_i(\omega^j) + \beta \cdot \delta^i \cdot \omega^j}$ represents the unpermuted
> $(\mathit{label}, value)$ pair, and ${v_i(\omega^j) + \beta \cdot s_i(\omega^j)}$
> represents the permuted $(\sigma(\mathit{label}), value)$ pair.

> 上式中 ${v_i(\omega^j) + \beta \cdot \delta^i \cdot \omega^j}$ 代表置换前的对
> $(\mathit{label}, value)$，${v_i(\omega^j) + \beta \cdot s_i(\omega^j)}$
> 则代表其置换的对 $(\sigma(\mathit{label}), value)$。

Let $Z_P$ be such that $Z_P(\omega^0) = Z_P(\omega^n) = 1$ and for $0 \leq j < n$:
$$\begin{array}{rl}

令多项式 $Z_P$ 满足 $Z_P(\omega^0) = Z_P(\omega^n) = 1$ 并且对 $0 \leq j < n$:
$$\begin{array}{rl}
Z_P(\omega^{j+1}) &= \prod\limits_{h=0}^{j} \prod\limits_{i=0}^{m-1} \frac{v_i(\omega^h) + \beta \cdot \delta^i \cdot \omega^h + \gamma}{v_i(\omega^h) + \beta \cdot s_i(\omega^h) + \gamma} \\
                  &= Z_P(\omega^j) \prod\limits_{i=0}^{m-1} \frac{v_i(\omega^j) + \beta \cdot \delta^i \cdot \omega^j + \gamma}{v_i(\omega^j) + \beta \cdot s_i(\omega^j) + \gamma}
\end{array}$$

Then it is sufficient to enforce the rules:

接下来只需要强制满足如下的规则对证明来说就足够了：
$$
Z_P(\omega X) \cdot \prod\limits_{i=0}^{m-1} \left(v_i(X) + \beta \cdot s_i(X) + \gamma\right) - Z_P(X) \cdot \prod\limits_{i=0}^{m-1} \left(v_i(X) + \beta \cdot \delta^i \cdot X + \gamma\right) = 0 \\
\ell_0 \cdot (1 - Z_P(X)) = 0
$$

This assumes that the number of columns $m$ is such that the polynomial in the first
rule above fits within the degree bound of the PLONK configuration. We will see
[below](#spanning-a-large-number-of-columns) how to handle a larger number of columns.

注意，上述第一条规则对其处理的多项式的假设是，其约束的列数 $m$ 要符合 PLONK 设置的次数界限。接下来，将在[下文](#spanning-a-large-number-of-columns)阐述如何处理列数超出边界的情况。

> The optimization used to obtain the simple representation of the identity permutation was suggested
> by Vitalik Buterin for PLONK, and is described at the end of section 8 of the PLONK paper. Note that
> the $\delta^i$ are all distinct quadratic non-residues, provided that the number of columns that
> are enabled for equality is no more than $T$, which always holds in practice for the curves used in
> Halo 2.

> 用于表示一个置换的经过优化的简单方法是由Vitalik Buterin为 PLONK 提出的，并且在 PLONK 论文的第8节有阐述
> 注意在实践中，对 Halo 2 所用的曲线来说，其有相等约束的列数通常不会超过其边界 $T$，
> 在此一条件下，所有的 $\delta^i$ 就都是不同二次平方非剩余。

## Zero-knowledge adjustment

## 零知识调整

Similarly to the [lookup argument](lookup.md#zero-knowledge-adjustment), we need an
adjustment to the above argument to account for the last $t$ rows of each column being
filled with random values.

与[查找证明](lookup.md#zero-knowledge-adjustment)类似，我们也需要调整上述的证明来处理每列的最后 $t$ 行，因为它们被填入了随机值（也就是这些行不满足上面的积证明）。

We limit the number of usable rows to $u = 2^k - t - 1.$ We add two selectors,
defined in the same way as for the lookup argument:

我们限定有用的行数是 $u = 2^k - t - 1$，同时我们增加两个选择子，这两个选择子与查找证明中定义的是一样的：

* $q_\mathit{blind}$ is set to $1$ on the last $t$ rows, and $0$ elsewhere;
* $q_\mathit{last}$ is set to $1$ only on row $u,$ and $0$ elsewhere (i.e. it is set on
  the row in between the usable rows and the blinding rows).

* $q_\mathit{blind}$ 只在最后 $t$ 行设为 $1$ ，而在其他地方则为 $0$。
* $q_\mathit{last}$ 则旨在 $u$ 行设为 $1$，其他行则为 $0$ （这就是说，它只在有用行和盲化行之间的那一行起作用）。

We enable the product rule from above only for the usable rows:

我们将在那些有用的行真正应用上文的积规则：

$\big(1 - (q_\mathit{last}(X) + q_\mathit{blind}(X))\big) \cdot$
$\hspace{1em}\left(Z_P(\omega X) \cdot \prod\limits_{i=0}^{m-1} \left(v_i(X) + \beta \cdot s_i(X) + \gamma\right) - Z_P(X) \cdot \prod\limits_{i=0}^{m-1} \left(v_i(X) + \beta \cdot \delta^i \cdot X + \gamma\right)\right) = 0$

The rule that is enabled on row $0$ remains the same:

在 $0$ 行的的规则仍然是相同的，即：

$$
\ell_0(X) \cdot (1 - Z_P(X)) = 0
$$

Since we can no longer rely on the wraparound to ensure that each product $Z_P$ becomes
$1$ again at $\omega^{2^k},$ we would instead need to constrain $Z(\omega^u) = 1.$ This
raises the same problem that was described for the lookup argument. So we allow
$Z(\omega^u)$ to be either zero or one:

由于我们再也不能依赖全景来确保每一个积证明 $Z_P$ 都在 $\omega^{2^k}$ 等于 $1$，那么我们实际上应该约束 $Z(\omega^u) = 1$。这也就带来了在查找证明中同样的问题。于是，我们也就需要允许 $Z(\omega^u)$ 等于 $0$ 或者 $1$：

$$
q_\mathit{last}(X) \cdot (Z_P(X)^2 - Z_P(X)) = 0
$$

which gives perfect completeness and zero knowledge.

通过这种方式，我们就获得了良好的完备性和零知识性。

## Spanning a large number of columns

The halo2 implementation does not in practice limit the number of columns for which
equality constraints can be enabled. Therefore, it must solve the problem that the
above approach might yield a product rule with a polynomial that exceeds the PLONK
configuration's degree bound. The degree bound could be raised, but this would be
inefficient if no other rules require a larger degree.

在halo2的实现中，实际上并不限制相等约束可以占用的行数。因此，就必须解决上述方法中可能导致的积规则中的多项式可能超出PLONK设置的次数边界的问题。一个简单办法是直接升高次数边界，但是在没有其他规则需要用到如此大的次数的情况下，这一办法是低效的。

Instead, we split the product across $b$ sets of $m$ columns, using product columns
$Z_{P,0}, \ldots Z_{P,b-1},$ and we use another rule to copy the product from the end
of one column set to the beginning of the next.

另一种方法，我们把积证明分割成 $b$ 个，每个都是其中 $m$ 列的证明，即：$Z_{P,0}, \ldots Z_{P,b-1},$同时增加一条规则，就是每个积（证明）最终的积被设置成下一个积（证明）的起始值。

That is, for $0 \leq a < b$ we have:

这就是说，对于 $0 \leq a < b$ 我们有：

$\big(1 - (q_\mathit{last}(X) + q_\mathit{blind}(X))\big) \cdot$
$\hspace{1em}\left(Z_{P,a}(\omega X) \cdot \!\prod\limits_{i=am}^{(a+1)m-1}\! \left(v_i(X) + \beta \cdot s_i(X) + \gamma\right) - Z_P(X) \cdot \!\prod\limits_{i=am}^{(a+1)m-1}\! \left(v_i(X) + \beta \cdot \delta^i \cdot X + \gamma\right)\right)$
$\hspace{2em}= 0$

> For simplicity this is written assuming that the number of columns enabled for
> equality constraints is a multiple of $m$; if not then the products for the last
> column set will have fewer than $m$ terms.

> 简单起见，上述规则假设总列数是 $m$ 的整数倍；
> 如果不是，那么我们就让最后一个列集合少于 $m$ 个项。

For the first column set we have:

对第一个列集合，我们有

$$
\ell_0 \cdot (1 - Z_{P,0}(X)) = 0
$$

For each subsequent column set, $0 < a < b,$ we use the following rule to copy
$Z_{P,a-1}(\omega^u)$ to the start of the next column set, $Z_{P,a}(\omega^0)$:

而对于剩余的每一个列集合 $0 < a < b$，我们将使用如下规则将 $Z_{P,a-1}(\omega^u)$ 拷贝到下一个列集合的最开始，$Z_{P,a}(\omega^0)$：

$$
\ell_0 \cdot \left(Z_{P,a}(X) - Z_{P,a-1}(\omega^u X)\right) = 0
$$

For the last column set, we allow $Z_{P,b-1}(\omega^u)$ to be either zero or one:

与起始算法相同，对最后一个列集合，我们约束 $Z_{P,b-1}(\omega^u)$ 为 $0$ 或者 $1$。

$$
q_\mathit{last}(X) \cdot \left(Z_{P,b-1}(X)^2 - Z_{P,b-1}(X)\right) = 0
$$

which gives perfect completeness and zero knowledge as before.

与上文的证明相同，这也提供了同样的完备性和零知识性。
