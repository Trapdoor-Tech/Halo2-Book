# Halo2 协议

## 预备工作

我们定义 $\sec$ 作为一个安全参数，且除明确标注外，文档中所有的算法和敌手(adversaries)都是在该安全参数下运行时间为多项式的概率型图灵机。我们使用 $\negl$ 表示一个在 $\sec$ 下可忽略的函数。

### 密码学群

我们令 $\group$ 表示一个阶为素数 $p$ 的循环群，群的单位元记作 $\zero$ 。将 $\group$ 中元素的标量表示为阶为 $p$ 的标量域 $\field$ 中的元素。群元素用大写字母表示，标量则用小写字母或希腊字母表示。群元素或标量所组成的向量用粗体字表示，例如 $\mathbf{a} \in \field^n$ 以及 $\mathbf{G} \in \group^n$ 。群上的运算记作加法，群元素 $G$ 和标量 $a$ 的乘法记作 $[a]G$ 。

我们会经常使用 $\langle \mathbf{a}, \mathbf{b} \rangle$ 表示长度相等的， $\field^n$ 上 $\mathbf{a}, \mathbf{b}$ 两个向量的内积。同样地也用这种表示法描述群元素的一个线性组合 $\langle \mathbf{a}, \mathbf{G} \rangle$ ，其中 $\mathbf{a} \in \field^n, \mathbf{G} \in \group^n$ ，该内积的计算使用标量乘法。

我们使用 $\mathbf{0}^n$ 描述域 $\field$ 中长度为 $n$ 的，只包含零元的向量。

> ** 离散对数关系问题 ** 
Advantage metric
$$
\adv^\dlrel_{\group,n}(\a, \sec) = \textnormal{Pr} \left[ \mathsf{G}^\dlrel_{\group,n}(\a, \sec) \right]
$$
> 定义为以下的 game 。
$$
\begin{array}{ll}
&\underline{\bold{Game} \, \mathsf{G}^\dlrel_{\group,n}(\a, \sec):} \\
&\mathbf{G} \gets \group^n_\sec \\
&\mathbf{a} \gets \a(\mathbf{G}) \\
&\textnormal{Return} \, \left( \langle \mathbf{a}, \mathbf{G} \rangle = \zero \land \mathbf{a} \neq \mathbf{0}^n \right)
\end{array}
$$

给定一个长度为 $n$ 的向量 $\mathbf{G} \in \group^n$ ， _离散对数关系问题_ 指寻找一个 $\mathbf{g} \in \field^n$ 使得 $\mathbf{g} \neq \mathbf{0}^n$ 且 $\innerprod{\mathbf{g}}{\mathbf{G}} = \zero$ （我们称为 _非平凡_ 离散对数关系）。该问题的难度和群中的离散对数问题紧密相连（[[JT20]](https://eprint.iacr.org/2020/1213) 引理3）。我们使用上面定义的 game $\dlgame$ 描述该问题。

### 交互式证明

_Interactive proofs_ are a triple of algorithms $\ip = (\setup, \prover,
\verifier)$. The algorithm $\setup(1^\sec)$ produces as its output some _public
parameters_ commonly refered to by $\pp$. The prover $\prover$ and verifier
$\verifier$ are interactive machines (with access to $\pp$) and we denote by
$\langle \prover(x), \verifier(y) \rangle$ an algorithm that executes a
two-party protocol between them on inputs $x, y$. The output of this protocol, a
_transcript_ of their interaction, contains all of the messages sent between
$\prover$ and $\verifier$. At the end of the protocol, the verifier outputs a
decision bit.

交互式证明是一组算法 $\ip = (\setup, \prover, \verifier)$ 。 $\setup(1^\sec)$ 算法生成某些被 $\pp$ 引用的 _公开参数_ 。 prover $\prover$ 和 verifier $\verifier$ 是能够访问 $\pp$ 的交互式机器，另外我们用 $\langle \prover(x), \verifier(y) \rangle$ 表示在 $\prover$ 和 $\verifier$ 之间执行的两方协议，其输入为 $x$ 和 $y$ 。协议的输出是一个transcript ，包含了所有 $\prover$ 和 $\verifier$ 之间交互的消息。在协议的最后， verifier 输出一个决策位，表示它是否接受 prover 的证明。

### 关于知识的零知识证明

知识证明是一种交互式证明，是 prover 试图向 verifier 证明它知道一个 witness $w$ 使得 $(x, w) \in \relation$ ，其中 $x$ 为一个 statement ， $\relation$ 为多项式时间内可决定的 relation。我们假设 prover 是计算能力有限的，然后我们描述 arguments of knowledge 。

我们将从四个角度分析 arguments of knowledge 的安全程度：

* **完备性:** 如果 prover 有一个有效的 witness ，它是否 _总是_ 能够向 verifier 证明？理解这一点有助于理解以下的性质
* **可靠性:** 一个试图作弊的 prover 能否向 verifier 证明一个错误的 statement ? 我们将这种情况称为 _可靠性错误_ 。
* **知识可靠性:** 当 verifier 确信 statement 是正确的时候， prover 是否实际拥有（“知道”）一个有效的 witness ？我们将 verifier 确信，但 prover 并不拥有有效 witness 的情况称为 _知识错误_ 。
* **零知识性:** 除了 statement 的正确性和 prover 实际拥有该有效 witness 之外， ???? 是否还获得了额外的知识？

首先，我们给出完备性的一个简单定义：

> **完美完备性** 一个交互式 argument $(\setup, \prover, \verifier)$
> 是 _完美完备的_ 当对于所有的多项式时间内可决定的 releation $\relation$ 和所有 non-uniform 多项式时间 adversaries $\a$ 都有
$$
Pr \left[
    (x, w) \notin \relation \lor
    \langle \prover(\pp, x, w), \verifier(\pp, x) \rangle \, \textnormal{accepts}
    \, \middle| \,
    \begin{array}{ll}
    &\pp \gets \setup(1^\sec) \\
    &(x, w) \gets \a(\pp)
    \end{array}
\right] = 1
$$

#### 可靠性

我们分析的复杂性在于：协议是交互式的，但实际上通过 Fiat-Shamir 变换，它被实现为 _非交互式_ 的。

> **Public coin** 我们将那种 verifier 发送的每个消息都来自新的随机性采样的交互式 argument 称为 public coin 式。
> 
> **Fiat-Shamir 变换** 将 verifier 发送的消息替换为密码学上 strong 的哈希函数以产生足够的随机性，即可将 public coin 式 argument 在 _随机预言机模型_ 中转换为 _非交互式_ 。

这种形式变换意味着在实际的协议中，一个试图作弊的 prover 可以轻易的“回卷”，从某个时间点发送新的消息给 verifier 。因此研究变换之后的安全性是非常重要的。幸运的是我们可以依照 Ghoshal 和 Tessaro 提出的框架 ([[GT20]]((https://eprint.iacr.org/2020/1351))) 来进行分析。

我们通过 _状态恢复可靠性_ 的概念分析我们的协议。在该模型中，一个作弊的 prover 可以“回卷” verifier 至之前任意的状态。如果 prover 能通过这种方式使 verifier 接受证明，则是对我们协议一次成功的攻击。

> **状态恢复可靠性** 令 $\ip$ 是一个交互式的 argument ，包含有 $r > = r(\sec)$ 个 verifier 挑战。令第 $i$th 个挑战从 $\ch_i$ 中采样获得。则一个状态恢复 prover $\prover$ 的 advantage metric
$$
\adv^\srs_\ip(\prover, \sec) = \textnormal{Pr} \left[ \srs^\ip_\prover(\sec) \right]
$$

通过以下 game 定义：

$$
\begin{array}{ll}
\begin{array}{ll}
&\underline{\bold{Game} \, \srs_\ip^\prover(\sec):} \\
&\textnormal{win} \gets \tt{false}; \\
&\tr \gets \epsilon \\
&\pp \gets \ip.\setup(1^\sec) \\
&(x, \textsf{st}_\prover) \gets \prover_\sec(\pp) \\
&\textnormal{Run} \, \prover^{\oracle_\srs}_\sec(\textsf{st}_\prover) \\
&\textnormal{Return win}
\end{array} &
\begin{array}{ll}
&\underline{\bold{Oracle} \, \oracle_\srs(\tau = (a_1, c_1, ..., a_{i - 1}, c_{i - 1}), a_i):} \\
& \textnormal{If} \, \tau \in \tr \, \textnormal{then} \\
& \, \, \textnormal{If} \, i \leq r \, \textnormal{then} \\
& \, \, \, \, c_i \gets \ch_i; \tr \gets \tr || (\tau, a_i, c_i); \textnormal{Return} \, c_i \\
& \, \, \textnormal{Else if} \, i = r + 1 \, \textnormal{then} \\
& \, \, \, \, d \gets \ip.\verifier (\pp, x, (\tau, a_i)); \tr \gets (\tau, a_i) \\
& \, \, \, \, \textnormal{If} \, d = 1 \, \textnormal{then win} \gets \tt{true} \\
& \, \, \, \, \textnormal{Return} \, d \\
&\textnormal{Return} \bottom
\end{array}
\end{array}
$$

如 [[GT20]]((https://eprint.iacr.org/2020/1351)) （定理1）所示，状态恢复可靠性和 Fiat-Shamir 变换后的可靠性紧密相关。

#### 知识可靠性

我们将会展示我们的协议满足 _witness extended emulation_ ，即一种加强的知识可靠性。这意味着，对于任意成功的 prover 算法，都存在一个高效的 _emulator_ ，通过“回卷”和提供一些新构造的随机数， _emulator_ 可以从算法中提取出 witness 。

但我们必须调整一下我们对于 _witness extended emulation_ 的定义，以描述一个事实，即 prover 都是状态恢复 prover ，能够“回卷” verifier 。另外，为避免在提取 witness 时回卷 prover，我们在 _代数群模型_ 中研究我们的协议。

> **代数群模型（Algebraic Group Model, AGM）** 若 Adversary $\alg{\prover}$ 在每次输出群元素
> $X$ 的时候同时也输出 $X$ 在 $\field^n$ 中的一个 _表示_ $\mathbf{x} \in \field^n$ ，使得 $\langle \mathbf{x}, \mathbf{G} \rangle = X$ ， $\mathbf{G} \in \group^n$ 是 $\alg{\prover}$ 所知的群元素的向量，则我们说 $\alg{\prover}$ 是 _代数的_ 。
> 
> 我们用 $\left[X\right]$ 表示群元素 $X$ ，定义 $[X]^{\mathbf{G}}_i$ 为 $X$ 用 $\mathbf{G}_i$ 表示的系数，即
$$
X = \sum\limits_{i=0}^{n - 1} \left[ [X]^{\mathbf{G}}_i \right] \mathbf{G}_i
$$

代数群模型允许我们对某些协议进行所谓的“在线”提取： extractor 可以从某个被接受的 transcript 表示方式中提取出 witness 。

> **状态恢复 witness extended emulation** 令 $\ip$ 是一个 relation $\relation$ 的交互式 argument ，有 $r = r(\sec)$ 个挑战。我们定义对所有 non-uniform 的代数 prover $\alg{\prover}$ ， extractor $\extractor$ ，以及拥有无限计算能力的 distinguisher $\distinguisher$ ，其 advantage metric 
$$
\adv^\srwee_{\ip, \relation}(\alg{\prover}, \distinguisher, \extractor, \sec) = \textnormal{Pr} \left[ \weereal^{\prover,\distinguisher}_{\ip,\relation}(\sec) \right] - \textnormal{Pr} \left[ \weeideal^{\extractor,\prover,\distinguisher}_{\ip,\relation}(\sec) \right]
$$
> 定义为如下 game 的衡量？？？
$$
\begin{array}{ll}
\begin{array}{ll}
&\underline{\bold{Game} \, \weereal_{\ip,\relation}^{\alg{\prover},\distinguisher}(\sec):} \\
&\tr \gets \epsilon \\
&\pp \gets \ip.\setup(1^\sec) \\
&(x, \state{\prover}) \gets \alg{\prover}(\pp) \\
&\textnormal{Run} \, \alg{\prover}^{\oracle_\real}(\state{\prover}) \\
&b \gets \distinguisher(\tr) \\
&\textnormal{Return} \, b = 1 \\
&\underline{\bold{Game} \, \weeideal_{\ip,\relation}^{\extractor,\alg{\prover},\distinguisher}(\sec):} \\
&\tr \gets \epsilon \\
&\pp \gets \ip.\setup(1^\sec) \\
&(x, \state{\prover}) \gets \alg{\prover}(\pp) \\
&\state{\extractor} \gets (1^\sec, \pp, x) \\
&\textnormal{Run} \, \alg{\prover}^{\oracle_\ideal}(\state{\prover}) \\
&w \gets \extractor(\state{\extractor}, \bottom) \\
&b \gets \distinguisher(\tr) \\
&\textnormal{Return} \, (b = 1) \\
&\, \, \land (\textnormal{Acc}(\tr) \implies (x, w) \in \relation) \\
\end{array} &
\begin{array}{ll}
&\underline{\bold{Oracle} \, \oracle_\real(\tau = (a_1, c_1, ..., a_{i - 1}, c_{i - 1}), a_i):} \\
& \textnormal{If} \, \tau \in \tr \, \textnormal{then} \\
& \, \, \textnormal{If} \, i \leq r \, \textnormal{then} \\
& \, \, \, \, c_i \gets \ch_i; \tr \gets \tr || (\tau, a_i, c_i); \textnormal{Return} \, c_i \\
& \, \, \textnormal{Else if} \, i = r + 1 \, \textnormal{then} \\
& \, \, \, \, d \gets \ip.\verifier (\pp, x, (\tau, a_i)); \tr \gets (\tau, a_i) \\
& \, \, \, \, \textnormal{If} \, d = 1 \, \textnormal{then win} \gets \tt{true} \\
& \, \, \, \, \textnormal{Return} \, d \\
& \, \, \textnormal{Return} \, \bottom \\
\\
\\
&\underline{\bold{Oracle} \, \oracle_\ideal(\tau, a):} \\
& \textnormal{If} \, \tau \in \tr \, \textnormal{then} \\
& \, \, (r, \state{\extractor}) \gets \extractor(\state{\extractor}, \left[(\tau, a)\right]) \\
& \, \, \tr \gets \tr || (\tau, a, r) \\
& \, \, \textnormal{Return} \, r \\
&\textnormal{Return} \, \bottom
\end{array}
\end{array}
$$

#### 零知识性

若 verifier 在协议的交互中除了知道存在一个有效的 $w$ 之外没有获得任何其他额外的知识，则我们说该关于知识的 argument 是 _零知识性_ 的。

> **完美 special 诚实 verifier 零知识性** 一个交互式的 public coin argument $(\setup, \prover, \verifier)$ 拥有 _完美 special 诚实 verifier 零知识性_ (PSHVZK) 仅当对于所有多项式时间可决定的 relation $\relation$ 和所有 $(x, w) \in \relation$ 及所有 non-uniform 多项式时间的 adversaries $\a_1$ ， $\a_2$ ，存在一个概率性的多项式时间 simulator $\sim$ ，使得
$$
\begin{array}{rl}
&Pr \left[ \a_1(\sigma, x, \tr) = 1 \, \middle| \,
\begin{array}{ll}
&\pp \gets \setup(1^\lambda); \\
&(x, w, \rho) \gets \a_2(\pp); \\
&tr \gets \langle \prover(\pp, x, w), \verifier(\pp, x, \rho) \rangle
\end{array}
 \right] \\
 \\
=&Pr \left[ \a_1(\sigma, x, \tr) = 1 \, \middle| \,
\begin{array}{ll}
&\pp \gets \setup(1^\lambda); \\
&(x, w, \rho) \gets \a_2(\pp); \\
&tr \gets \sim(\pp, x, \rho)
\end{array}
\right]
\end{array}
$$
> $\rho$ 表示 verifier 内在的随机性。

在这种零知识性的定义下， verifier 预期将会“诚实地”交互，并发送只与它的内在随机性相关的挑战。它们不能根据 prover 发送的消息改变自己的挑战。我们使用该定义的一种强化形式，要求 simulator 输出包含和 verifier 所发送的相同的挑战的 transcript 。

## 协议

令 $\omega \in \field$ 是 $n = 2^k$ 的单位根，组成一个 domain $D = (\omega^0, \omega^1, ..., \omega^{n - 1})$ ， $t(X) = X^n - 1$ 为该 domain 上的消去多项式。令 $k, n_g, n_a$ 为正整数。对于 relation 
$$
\relation = \left\{
\begin{array}{ll}
&\left(
\begin{array}{ll}
\left(
g(X, C_0, C_1, ..., C_{n_a - 1})
\right); \\
\left(
a_0(X), a_1(X, C_0), ..., a_{n_a - 1}(X, C_0, C_1, ..., C_{n_a - 1})
\right)
\end{array}
\right) : \\
\\
& g(\omega^i, C_0, C_1, ..., C_{n_a - 1}) = 0 \, \, \, \, \forall i \in [0, 2^k)
\end{array}
\right\}
$$

我们提出一种交互式 argument $\halo = (\setup, \prover, \verifier)$ ，其中 $a_0, a_1, ..., a_{n_a - 1}$ 是关于 $X$ 的度数为 $n -1$ 的（多变量）多项式， $g$ 是关于 $X$ 的度数为 $n_g(n - 1)$ 的多项式。

$\setup(\sec)$ 返回 $\pp = (\group, \field, \mathbf{G} \in \group^n, U, W \in \group)$.

对于所有的 $i \in [0, n_a)$:
* 令 $\mathbf{p_i}$ 为整数 $j$ （模 $n$ ）的完备集，使得 $a_i(\omega^j X, \cdots)$ 为 $g(X, \cdots)$ 中的成员。
* 令 $\mathbf{q}$ 为一组包含 $\mathbf{p}_i$ 的不相交的整数集合， $\mathbf{q_0} = \{0\}$.
* 令 $\sigma(i) = \mathbf{q}_j$ ，若 $\mathbf{q}_j = \mathbf{p_i}$.

令 $n_q$ 表示集合 $\mathbf{q}$ 的大小，不失一般性地，令 $n_e$ 表示每个 $\mathbf{p_i}$ 集合的大小。

在下述的协议中，我们默认每个 $a_i(X,\cdots)$ 多项式为 $n_e + 1$ 个盲因子均为 prover 新采样的，且以 domain $D$ 上各点值的形式表示。

1. $\prover$ 和 $\verifier$ 进行 $n_a$ 轮交互，在第 $j$ 轮中（轮次从0开始）
  * $\prover$ 设置 $a'_j(X) = a_j(X, c_0, c_1, ..., c_{j - 1})$
  * $\prover$ 发送一个隐藏用的承诺 $A_j = \innerprod{\mathbf{a'}}{\mathbf{G}} + [\cdot] W$ ， 其中 $\mathbf{a'}$ 为单变量多项式 $a'_j(X)$ 的系数， $\cdot$ 为某个随机且独立采样的盲因子。
  * $\verifier$ 回应 $\prover$ 一个挑战 $c_j$.
2. $\prover$ 和 $\verifier$ 设置 $g'(X) = g(X, c_0, c_1, ..., c_{n_a - 1})$.
3. $\prover$ 发送一个承诺 $R = \innerprod{\mathbf{r}}{\mathbf{G}} + [\cdot] W$ ，其中 $\mathbf{r} \in \field^n$ 为随机采样的单变量多项式 $r(X)$ 的系数，其度数为 $n - 1$ 。
4. $\prover$ 计算单变量多项式 $h(X) = \frac{g'(X)}{t(X)}$ ，其度数为 $n_g(n - 1) - n$.
5. $\prover$ 计算度数至多为 $n - 1$ 的多项式 $h_0(X), h_1(X), ..., h_{n_g - 2}(X)$ ，使得 $h(X) = \sum\limits_{i=0}^{n_g - 2} X^{ni} h_i(X)$.
6. $\prover$ 将所有的承诺 $H_i = \innerprod{\mathbf{h_i}}{\mathbf{G}} + [\cdot] W$ 发送给 $\verifier$ ，其中 $h_i(X)$ 为多项式系数组成的向量。
7. $\verifier$ 回应一个挑战 $x$ 并计算 $H' = \sum\limits_{i=0}^{n_g - 2} [x^{ni}] H_i$.
8. $\prover$ 设置 $h'(X) = \sum\limits_{i=0}^{n_g - 2} x^{ni} h_i(X)$.
9. $\prover$ 发送 $r = r(x)$ 并且对所有的 $i \in [0, n_a)$ ，发送 $\mathbf{a_i}$ ，使得对所有的 $j \in [0, n_e]$ ，都有 $(\mathbf{a_i})_j = a'_i(\omega^{(\mathbf{p_i})_j} x)$ 。
10. 对所有的 $i \in [0, n_a)$ ， $\prover$ 和 $\verifier$ 设置 $s_i(X)$ 为度数最低的单变量多项式，使得对所有的 $j \in [0, n_e)$ ， $s_i(\omega^{(\mathbf{p_i})_j} x) = (\mathbf{a_i})_j$ 。
11. $\verifier$ 发送两个挑战 $x_1, x_2$ 以回应，并初始化 $Q_0, Q_1, ..., Q_{n_q - 1} = \zero$.
  * 从 $i=0$ 到 $i =n_a - 1$ ， $\verifier$ 设置 $Q_{\sigma(i)} := [x_1] Q_{\sigma(i)} + A_i$.
  * 最后 $\verifier$ 设置 $Q_0 := [x_1^2] Q_i + [x_1] H' + R$.
12. $\prover$ 初始化 $q_0(X), q_1(X), ..., q_{n_q - 1}(X) = 0$.
  * 从 $i=0$ 开始到 $i=n_a - 1$ ， $\prover$ 设置 $q_{\sigma(i)} := x_1 q_{\sigma(i)} + a'(X)$.
  * 最后 $\prover$ 设置 $q_0(X) := x_1^2 q_0(X) + x_1 h'(X) + r(X)$.
13. $\prover$ 和 $\verifier$ 初始化 $r_0(X), r_1(X), ..., r_{n_q - 1}(X) = 0$.
  * 从 $i=0$ 开始到 $i=n_a - 1$ ， $\prover$ 和 $\verifier$ 设置 $r_{\sigma(i)}(X) := x_1 r_{\sigma(i)}(X) + s_i(X)$.
  * 最后 $\prover$ 和 $\verifier$ 设置 $r_0 := x_1^2 r_0 + x_1 h + r$ ， $\verifier$ 使用由 $\prover$ 提供的 $r$ 和 $\mathbf{a}$ 计算 $h = \frac{g'(x)}{t(x)}$ 
14. $\prover$ 发送 $Q' = \innerprod{\mathbf{q'}}{\mathbf{G}} + [\cdot] W$ ，其中 $\mathbf{q'}$ 为下列多项式的系数：

$$q'(X) = \sum\limits_{i=0}^{n_q - 1}
x_2^i
  \left(
  \frac
  {q_i(X) - r_i(X)}
  {\prod\limits_{j=0}^{n_e - 1}
    \left(
      X - \omega^{\left(
        \mathbf{q_i}
      \right)_j} x
    \right)
  }
  \right)
$$

15. $\verifier$ 回以挑战 $x_3$.
16. $\prover$ 发送 $\mathbf{u} \in \field^{n_q}$ 使得 $\mathbf{u}_i = q_i(x_3)$ for all $i \in [0, n_q)$.
17. $\verifier$ 回以挑战 $x_4$.
18. $\prover$ 和 $\verifier$ 设置 $P = Q' + x_4 \sum\limits_{i=0}^{n_q - 1} [x_4^i] Q_i$ ， $v = $

$$
\sum\limits_{i=0}^{n_q - 1}
\left(
x_2^i
  \left(
  \frac
  { \mathbf{u}_i - r_i(x_3) }
  {\prod\limits_{j=0}^{n_e - 1}
    \left(
      x_3 - \omega^{\left(
        \mathbf{q_i}
      \right)_j} x
    \right)
  }
  \right)
\right)
+
x_4 \sum\limits_{i=0}^{n_q - 1} x_4 \mathbf{u}_i
$$

19. $\prover$ 设置 $p(X) = q'(X) + [x_4] \sum\limits_{i=0}^{n_q - 1} x_4^i q_i(X)$.
20. $\prover$ 采样一个随机的度数为 $n-1$ 的多项式 $s(X)$ （其中一个根为 $x_3$ ） ，并发送其承诺 $S = \innerprod{\mathbf{s}}{\mathbf{G}} + [\cdot] W$ ， $\mathbf{s}$ 为 $s(X)$ 多项式的系数。
21. $\verifier$ 回以挑战 $\xi, z$.
22. $\prover$ 和 $\verifier$ 设置 $P' = P - [v] \mathbf{G}_0 + [\xi] S$.
23. $\prover$ 设置 $p'(X) = p(X) - v + \xi s(X)$.
24. 初始化 $\mathbf{p'}$ 作为多项式 $p'(X)$ 的系数，令 $\mathbf{G'} = \mathbf{G}$ ， $\mathbf{b} = (x_3^0, x_3^1, ..., x_3^{n - 1})$. $\prover$ 和 $\verifier$ 将以如下方式交互 $k$ 轮，$j$ 的取值范围从 $j=0$ 到 $j=k-1$ ：
  * $\prover$ 发送 $L_j = \innerprod{\mathbf{p'}_\hi}{\mathbf{G'}_\lo} + [z \innerprod{\mathbf{p'}_\hi}{\mathbf{b}_\lo}] U + [\cdot] W$ and $R_j = \innerprod{\mathbf{p'}_\lo}{\mathbf{G'}_\hi} + [z \innerprod{\mathbf{p'}_\lo}{\mathbf{b}_\hi}] U + [\cdot] W$.
  * $\verifier$ 回以挑战 $u_j$.
  * $\prover$ 和 $\verifier$ 设置 $\mathbf{G'} := \mathbf{G'}_\lo + u_j \mathbf{G'}_\hi$ 且 $\mathbf{b} = \mathbf{b}_\lo + u_j \mathbf{b}_\hi$.
  * $\prover$ 设置 $\mathbf{p'} := \mathbf{p'}_\lo + u_j^{-1} \mathbf{p'}_\hi$.
25. $\prover$ 发送 $c = \mathbf{p'}_0$ 以及盲因子 $f$.
26. 若 $\sum_{j=0}^{k - 1} [u_j^{-1}] L_j + P' + \sum_{j=0}^{k - 1} [u_j] R_j = [c] \mathbf{G'}_0 + [c \mathbf{b}_0 z] U + [f] W$ ，则 $\verifier$ 接受证明。
