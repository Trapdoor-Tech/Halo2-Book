# 域

对于众多密码学协议来说，[域][field] —— 一种代数结构，都是一个最基本的组件。域，就是指拥有 $+$ 和 $\times$ 两个二元运算的集合（通常是数集），并且满足一系列 [域公理][field-axioms]。举例来说，实数集 $\mathbb{R}$ 就是一个拥有不可数无穷多元素的域。

[field]: https://en.wikipedia.org/wiki/Field_(mathematics)
[field-axioms]: https://en.wikipedia.org/wiki/Field_(mathematics)#Classic_definition

Halo运用有限域，所谓有限域是指域的元素个数是有限的。有限域完全分类如下：

- 如果 $\mathbb{F}$ 是有限域，那么它包含 $|\mathbb{F}| = p^k$ 个元素，其中 $k$ 是整数且 $k \geq 1$， $p$是一个素数。

- 任意两个有相同元素个数的有限域是同构的。特别的，素域 $\mathbb{F}_p$ 上的算术运算与整数模$p$ 上的加法和乘法是同构的，也就是说，$\mathbb{Z}_p$。这就是我们将 $p$ 作为 _模数_ 的原因。

我们定义一个域 $\mathbb{F}_q$ 其中 $q = p^k$. 素数 $p$ 称为它的 _特征_。
如果 $k \gt 1$ ， 那么域 $\mathbb{F}_q$ 就是域 $\mathbb{F}_p$ 的 $k$阶扩张。 (相类似，复数 $\mathbb{C} = \mathbb{R}(i)$ 就是实数域的扩张）。但是, 在Halo中，我们并不使用扩域。当我们写 $\mathbb{F}_p$ 我们指的是拥有素数 $p$ 个元素的 _素域_，也就是说此时 $k = 1$。

重要注释：

* 任何域中，都有两个特殊的元素： $0$，加法的单位元； $1$，乘法的单位元。

* 一个域中的元素，其在二进制表示中的最低位，可以用来代表它在域上的“符号”以区分它的加法逆元（即负数）。这是因为，对任意一个最低位为 $0$ 的非零元素 $a$ 其加法逆元 $-a = p - a$ 的最低位一定是 $1$，反之亦然。我们也可以用一个元素是不是大于 $(p - 1) / 2$ 来作为其“符号”。


有限域对于后文中构建 [多项式](polynomials.md) 和 [椭圆曲线](curves.md) 是很有用处的。 椭圆曲线是一个群，我们将马上对群展开讨论。

## 群

与域相比，群更加简单，并且更多限制；群只有一个二元运算 $\cdot$ ，和更少的公理。群也有一个单位元，我们用 $1$ 表示。

[group]: https://en.wikipedia.org/wiki/Group_(mathematics)
[group-axioms]: https://en.wikipedia.org/wiki/Group_(mathematics)#Definition

群中任意一个非零元素 $a$ 必有一个 _唯一_ 的 _逆元_，记为 $b = a^{-1}$，满足 $a \cdot b = 1$。

举个例子，域 $\mathbb{F}_p$中所有非零元素形成一个群，同时定义该群的运算就是域上的乘法。

[group]: https://en.wikipedia.org/wiki/Group_(mathematics)

> #### (备注) 加法 vs 乘法符号 
> 
> 如果将 $\cdot$ 写做 $\times$ 或是干脆不写 （比如： 将 $a \cdot b$ 写做 $ab$), 那么正如上文所述，单位元
> 就写做 $1$, 它的逆元就是 $a^{-1}$，我们说这样的群“被写作乘”。而如果将 $\cdot$ 视为 $+$, 那么它的单位元就是 $0$ 或者
> $\mathcal{O}$, 它的逆元就是 $-a$, 我们说这样的群“被写做加”。
> 通常，椭圆曲线群用加法符号来表示, 而对于有限域中的元素构成的群用乘法符号表示
>
> 当我们使用加法符号时，将写如下式子：
>
> $$[k] A = \underbrace{A + A + \cdots + A}_{k \text{ times}}$$
>
> 对于任意非负 $k$ 我们称上式为“标量乘法”;通常，我们使用大写字母变量来表示群的元素。当我们使用乘法符号时，如下等式
>
> $$a^k = \underbrace{a \times a \times \cdots \times a}_{k \text{ times}}$$
> 就被称为“指数”。无论上述那种情况，我们都将满足 $[k] g = a$ 或 $g^k = a$ 的 $k$ 称为 $a$ 在基 $g$ 下的“离散对数”。 我们可以通过求逆运算
> 将标量扩展到其逆，也就是说：$[-k] A + [k] A = \mathcal{O}$ 或者 $a^{-k} \times a^k = 1$.

有限域元素 $a$ 的 _阶_ 定义为满足如下条件的最小的正整数 $k$，$a^k = 1$ （乘法符号）或者 $[k] a = \mathcal{0}$ （加法表示）。
_群的阶_ 就是指群包含的元素的个数。

群总有一个 [生成集][generating set]，所谓生成集是这样一个集合，可以使用该集合中的元素通过这些元素的乘方运算（乘法术语）来生成群中的任意一个元素。
如果，生成集是 $g_{1..k}$， 我们就可以通过 $\prod\limits_{i=1}^{k} g_i^{a_i}$ 生成群的所有元素。对于一个给定的群，它有很多不同的生成集。

[generating set]: https://en.wikipedia.org/wiki/Generating_set_of_a_group


如果一个群的存在一个生成集（不一定是唯一的）只有一个元素，记为 $g$ ，那么这个群就是一个[循环群][cyclic]。进而，我们称 $g$ 生成了这个群，并且 $g$ 的阶就是该群的阶。

任意一个阶为 $n$ 的有限循环群 $\mathbb{G}$ 与整数modulo $n$ （记为 $\mathbb{Z}/n\mathbb{Z}$）是[同构][isomorphic]的，满足：

- $\mathbb{G}$ 上的运算 $\cdot$ 与modulo $n$ 的加法对应；
- $\mathbb{G}$ 的单位元与 $0$ 对应;
- 有一个生成元 $g \in \mathbb{G}$ 与 $1$ 对应.

对于一个给定的生成元 $g$ ，容易证明 $\mathbb{Z}/n\mathbb{Z} \rightarrow \mathbb{G}$ 是成立的，因为$a \mapsto g^a$（或是用加法符号， $a \mapsto [a] g$）。通常，证明 $\mathbb{G} \rightarrow \mathbb{Z}/n\mathbb{Z}$ 会难一些。我们将在[椭圆曲线](curves.md)中进一步讨论这个问题。

如果一个有限群的阶 $n$ 是一个素数，那么该群是个循环群，并且每个非单位元元素都是其生成元。

[isomorphic]: https://en.wikipedia.org/wiki/Isomorphism
[cyclic]: https://en.wikipedia.org/wiki/Cyclic_group

### 有限域的乘法子群

我们用符号 $\mathbb{F}_p^\times$ 来代表集合 $\mathbb{F}_p - \{0\}$ 上的乘法群（也就是说，群的运算就是 $\mathbb{F}_p$ 的乘法）。

在$\mathbb{F}_p^\times$子群，一个快速求逆的方法是 $a^{-1} = a^{p - 2}$ 。这是因为，由[费马小定理][fermat-little]可知，如果 $p$ 是一个素数，
对任意整数 $a$ 且 $a$ 不是 $p$ 的倍数，都有 $a^p = a \pmod p$ 。如果 $a$ 是非零整数，那么我们可以除两次 $a$ 就获得 $a^{p-2} = a^{-1}.$。

[fermat-little]: https://en.wikipedia.org/wiki/Fermat%27s_little_theorem

设 $\alpha$ 是 $\mathbb{F}_p^\times$ 的生成元，所以 $\alpha$ 的阶是 $p-1$ （等于 $\mathbb{F}_p^\times$ 的元素的个数）. 因此，对任意一个
$a \in \mathbb{F}_p^\times$ 都存在唯一一个 $i \in \{0..p-2\}$ 满足 $a = \alpha^i$.

值得注意的是 $a \times b$ 其中 $a, b \in \mathbb{F}_p^\times$ 确实可以表示成
$\alpha^i \times \alpha^j$ 其中 $a = \alpha^i$ 并且 $b = \alpha^j$。实际上，对任意 $0 \leq i, j \lt p - 1$ 它也保持了如下运算
$\alpha^i \times \alpha^j = \alpha^{i + j}$ 。因此，域上非零元素的乘法就可以表示成关于某个特定生成元 $ \alpha$ 的在 模 $p - 1$ 上的加法。这个加法，就出现在“指数”上。

这是另一种理解 $a^{p - 2}$ 就是域元素的逆元的方法：

$$p - 2 \equiv -1 \pmod{p - 1},$$

所以 $a^{p - 2} = a^{-1}$。

### 蒙哥马利技巧

蒙哥马利技巧，以彼得 · 蒙哥马利 (RIP) 得名，是同时计算很多群元素的逆元的一种方法。 它通常用于在 $\mathbb{F}_p^\times$ 中求逆，在其上求逆运算
要比乘法运算代价大得多。

假设我们要计算三个非零元素 $a, b, c \in \mathbb{F}_p^\times$ 的逆元。不似常规，我们将计算如下两个积 $x = ab$ 和 $y = xc = abc$，并且计算如下的逆

$$z = y^{p - 2} = \frac{1}{abc}.$$

现在我们可以 $z$ 乘以 $x$ 以获得 $\frac{1}{c}$ ，并且 $z$ 乘以 $c$ 得到
$\frac{1}{ab}$, 进一步该值分别乘以 $a, b$ 就分别获得了想要的逆元。

该技术可以推广，仅仅一次求逆运算就可以计算出任意个数的群元素的逆。

## 乘法子群

所谓群 $G$ 在 $\cdot$ 下的 _子群_ 就是，$G$ 中元素的一个子集在 $\cdot$ 下仍然是一个群。

在前面的章节中，我们说 $\alpha$ 是 $(p - 1)$-阶群 $\mathbb{F}_p^\times$ 的生成元。
该群拥有 _合数_ 阶，所以依据中国剩余定理[^chinese-remainder] 它有真子群。举个例子，假设 $p = 11$，那么 $p - 1$ 可以分解为 $5 \cdot 2$。因此，存在一个
以 $\beta$ 为生成元的 $5$-阶子群和一个以 $\gamma$ 为生成元的 $2$-阶子群。进一步，$\mathbb{F}_p^\times$ 中所有元素都可以表示为 $\beta^i \cdot \gamma^j$ 其中 $i$ (modulo $5$) 和 $j$ (modulo $2$).

如果我们有 $a = \beta^i \cdot \gamma^j$ 请注意如下计算过程

$$
a^5 = (\beta^i \cdot \gamma^j)^5
    = \beta^{i \cdot 5} \cdot \gamma^{j \cdot 5}
    = \beta^0 \cdot \gamma^{j \cdot 5}
    = \gamma^{j \cdot 5};
$$

我们“消灭”了 $5$-阶子群部分，仅仅只用 $2$-阶子群就生成了这个值。

[拉格朗日定理 (群论)][lagrange-group] 告诉我们，有限群 $G$ 的任意一个子群 $H$ 的阶能整除 $G$ 的阶。因此 $\mathbb{F}_p^\times$ 的任何子群的阶必定能整除 $p-1$ 。

[lagrange-group]: https://en.wikipedia.org/wiki/Lagrange%27s_theorem_(group_theory)

[PLONK-based] 证明系统， 如 Halo 2，可以更方便的利用拥有大量乘法子群的域，并且这些子群是“平滑”分布（好处是使得性能差距更小，并且可以以更细的力度增长电路规模）。Pallas 和 Vesta 曲线有如下形式的素数

$$T \cdot 2^S = p - 1$$

其中 $S = 32$ 并且 $T$ 奇数 （这就是说， $p - 1$ 的低32位都是 $0$）。这意味着，它们有阶为 $2^k$ 的乘法子群，其中 $k \leq 32$。这些 2进制乘法子群对[efficient FFTs] 十分友好，而且也可以支持多种电路规模。

[PLONK-based]: upa.md
[efficient FFTs]: polynomials.md#fast-fourier-transform-fft

## 平方根 

在域 $\mathbb{F}_p$ 有一半的非零元素都是完全平方元素；剩下的就是非平方或者“二次非剩余”。为了说明此一事实，考虑 $\alpha$ 生成的 $\mathbb{F}_p^\times$ 的 2-阶乘法子群（该子群必然存在，因为当 $p$ 是一个大于等于 $2$ 的素数的时候，$p - 1$ 能被 $2$ 整除）和一个由 $\beta$ 生成的 $\mathbb{F}_p^\times$ 的
$t$-阶乘法子群，其中 $p - 1 = 2t$。进而每个元素 $a \in \mathbb{F}_p^\times$ 均可以唯一地表示为 $\alpha^i \cdot \beta^j$ 其中 $i \in \mathbb{Z}_2$ 并且 $j \in \mathbb{Z}_t$。那么有一半的元素满足 $i = 0$ 而另一半元素满足 $i = 1$。

考虑一个简单的例子， $p \equiv 3 \pmod{4}$ 那么 $t$ 必是一个奇数 （如果 $t$ 是偶数，则 $p - 1$ 必能被 $4$ 整除，而这与 $p$ 是 $3 \pmod{4}$ 矛盾）。 如果 $a \in \mathbb{F}_p^\times$ 是一个完全平方数，则必存在一个 $b = \alpha^i \cdot \beta^j$ 使得 $b^2 = a$。这就意味着，

$$a = (\alpha^i \cdot \beta^j)^2 = \alpha^{2i} \cdot \beta^{2j} = \beta^{2j}.$$

换句话说，某个特定域中的完全平方数不可能生成其 $2$-阶乘法子群，而且由于有一半的元素可以产生其 $2$-阶子群，所以该域中至多有一半的元素是完全平方数。事实上，有且仅有一半的元素是完全平方数（这是因为，每一个非平方元素的平方都是唯一的）。这就意味着，我们可以假设所有的完全平方数都可以表示为 $\beta^m$ 对某个 $m$，因此求平方根就变成了一个指数运算，指数就是 $2^{-1} \pmod{t}$。

而如果 $p \equiv 1 \pmod{4}$ 那问题就变得复杂得多，因为 $2^{-1} \pmod{t}$ 并不存在。
记 $p - 1$ = $2^k \cdot t$ 其中 $t$ 是奇数。$k=0$不可能，$k=1$上面已经讨论，现在考虑
$k \geq 2$。 $\alpha$ 生成一个 $2^k$-阶乘法子群而 $\beta$ 生成一个奇数 $t$-阶得乘法子群。
进而每个元素 $a \in \mathbb{F}_p^\times$ 都可以表示为 $\alpha^i \cdot \beta^j$ 其中 $i \in \mathbb{Z}_{2^k}$ 并且
$j \in \mathbb{Z}_t$。如果某元素是完全平方数，就是说存在 $b = \sqrt{a}$ 而 $b$ 本身可以表示为
$b = \alpha^{i'} \cdot \beta^{j'}$ 对于 $i' \in \mathbb{Z}_{2^k}$ 和
$j' \in \mathbb{Z}_t$。这意味着 $a = b^2 = \alpha^{2i'} \cdot \beta^{2j'}$，
因此我们推出 $i \equiv 2i' \pmod{2^k}$，并且 $j \equiv 2j' \pmod{t}$。那么在这种情况下 $i$ 就必须是一个偶数，否则对任意 $i'$, 
$i \equiv 2i' \pmod{2^k}$ 就不能成立。反之，如果 $a$ 不是一个完全平方数，那么 $i$就是一个奇数。综上，一半的元素是完全平方数。

为了计算平方根，我们首先将 $a = \alpha^i \cdot  \beta^j$ 升次到 $t$ 以“消灭” $t$-阶部分，计算

$$a^t = \alpha^{it \pmod 2^k} \cdot \beta^{jt \pmod t} = \alpha^{it \pmod 2^k}$$

然后在此基础上计算 $t^{-1} \pmod{2^k}$ 次方，以消除其 $2^k$-阶部分原始指数的影响：

$$(\alpha^{it \bmod 2^k})^{t^{-1} \pmod{2^k}} = \alpha^i$$

（因为 $t$ 与 $2^k$ 是互素的）。这就暴露出 $\alpha^i$ 的值可以让我们进一步处理。
相似的，我们可以消除 $2^k$-阶部分以获得 $\beta^{j \cdot 2^{-1} \pmod{t}}$ （译者注：此处应该是笔误，$\beta^{j \pmod{t}}$），将这两个值合起来就得出了平方根。

已经证明当 $k = 2, 3$ 时，有更简便的方法来合并这些指数以提高效率。
而对于其他的 $k$ 值，唯一获取 $i$ 的方法就是，不断平方以最终确定 $i$ 的每一位。
这就是 [Tonelli-Shanks square root algorithm][ts-sqrt] 算法的本质，并且该算法还描述了一半的策略。
（当然还有另外一种用二次扩域的求取平方根的算法，但是直到素数变得相当大时，该算法才有效率优势）。

[ts-sqrt]: https://en.wikipedia.org/wiki/Tonelli%E2%80%93Shanks_algorithm

## 单位根

在前一章节，我们令 $p - 1 = 2^k \cdot t$ 其中 $t$ 是奇数，并且阐明
元素 $\alpha \in \mathbb{F}_p^\times$ 生成其 $2^k$-阶子群。
为方便起见，不妨设 $n := 2^k$。那么元素 $\{1, \alpha, \ldots, \alpha^{n-1}\}$ 都被称为 $n$ 次
[单位根](https://en.wikipedia.org/wiki/Root_of_unity)。

所谓 **本原单位根**，$\omega$，是满足如下条件的 $n$ 次单位根
$\omega^i \neq 1$ 除非 $i \equiv 0 \pmod{n}$.

重要补充：

- 如果 $\alpha$ 是一个 $n$ 次单位根，那么 $\alpha$ 满足 $\alpha^n - 1 = 0$。如果
  $\alpha \neq 1$，那么
  $$1 + \alpha + \alpha^2 + \cdots + \alpha^{n-1} = 0$$。
- 相应地，单位根集合就是如下方程的解
  $$X^n - 1 = (X - 1)(X - \alpha)(X - \alpha^2) \cdots (X - \alpha^{n-1}).$$

- **$\boxed{\omega^{\frac{n}{2}+i} =  -\omega^i}$ ("负数引理")**。证明：
  $$
  \begin{aligned}
  \omega^n = 1 &\implies \omega^n - 1 = 0 \\
  &\implies (\omega^{n/2} + 1)(\omega^{n/2} - 1) = 0.
  \end{aligned}
  $$
  因为 $\omega$ 的阶是 $n$, $\omega^{n/2} \neq 1$。因此，$\omega^{n/2} = -1$。

- **$\boxed{(\omega^{\frac{n}{2}+i})^2 =  (\omega^i)^2}$ ("折半引理")**。证明：  
  $$
  (\omega^{\frac{n}{2}+i})^2 = \omega^{n + 2i} = \omega^{n} \cdot \omega^{2i} = \omega^{2i} = (\omega^i)^2.
  $$
  换句话说，如果我们平方每一个 $n$ 次单位根集合中的元素，我们只会得到一半的元素
  $\{(\omega_n^i)^2\} = \{\omega_{n/2}\}$ （这就是 $\frac{n}{2}$ 次单位根）。
  在元素和它们的平方之间存在一个 $2 \to 1$ 的映射。

  ## 参考

[^chinese-remainder]: [Friedman, R. (n.d.) "Cyclic Groups and Elementary Number Theory II" (p. 5).](http://www.math.columbia.edu/~rf/numbertheory2.pdf)
