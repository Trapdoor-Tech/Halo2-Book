# Comparison to other work

# 与其他工作的比较

## BCMS20 附录 A.2

Appendix A.2 of [BCMS20] describes a polynomial commitment scheme that is similar to the
one described in [BGH19] (BCMS20 being a generalization of the original Halo paper). Halo
2 builds on both of these works, and thus itself uses a polynomial commitment scheme that
is very similar to the one in BCMS20.

[BCMS20] 附录 A.2 提出了一种与 [BGH19] 中类似的多项式承诺机制（BCMS20 是对原始的 Halo 论文的推广）。
Halo 2 这些工作的基础上，因此其自身也就使用了一种与 BCMS20 中很类似的多项式承诺机制。

[BGH19]: https://eprint.iacr.org/2019/1021
[BCMS20]: https://eprint.iacr.org/2020/499

The following table provides a mapping between the variable names in BCMS20, and the
equivalent objects in Halo 2 (which builds on the nomenclature from the Halo paper):

下表提供了一个BCMS20和Halo 2中具有相同含义的变量的对照表（Halo 2的术语是建立在Halo论文基础上的）：

|     BCMS20     |       Halo 2        |
| :------------: | :-----------------: |
|      $S$       |         $H$         |
|      $H$       |         $U$         |
|      $C$       |    `msm` or $P$     |
|    $\alpha$    |       $\iota$       |
|    $\xi_0$     |         $z$         |
|    $\xi_i$     |    `challenge_i`    |
|      $H'$      |       $[z] U$       |
|   $\bar{p}$    |      `s_poly`       |
| $\bar{\omega}$ |   `s_poly_blind`    |
|   $\bar{C}$    | `s_poly_commitment` |
|     $h(X)$     |       $g(X)$        |
|   $\omega'$    |   `blind` / $\xi$   |
|  $\mathbf{c}$  |    $\mathbf{a}$     |
|      $c$       | $a = \mathbf{a}_0$  |
|      $v'$      |        $ab$         |

Halo 2's polynomial commitment scheme differs from Appendix A.2 of BCMS20 in two ways:

Halo 2的多项式承诺机制与BCMS20附录A.2的机制有两点不同：

1. Step 8 of the $\text{Open}$ algorithm computes a "non-hiding" commitment $C'$ prior to
   the inner product argument, which opens to the same value as $C$ but is a commitment to
   a randomly-drawn polynomial. The remainder of the protocol involves no blinding. By
   contrast, in Halo 2 we blind every single commitment that we make (even for instance
   and fixed polynomials, though using a blinding factor of 1 for the fixed polynomials);
   this makes the protocol simpler to reason about. As a consequence of this, the verifier
   needs to handle the cumulative blinding factor at the end of the protocol, and so there
   is no need to derive an equivalent to $C'$ at the start of the protocol.

1. $\text{Open}$ 算法的第8步将在内积证明之前计算一个“非隐藏”的承诺 $C'$。
   该承诺与 $C$ 拥有相同的打开之，只不过它是仅是一个对一个随便写的多项式的承诺。
   协议的剩余部分则是没有盲化的。与之相比，Halo 2中我们会盲化我们产生的所有的承诺（即便对于instance和fixed多项式也是如此，
   尽管对fixed多项式的盲化因子是1），这使得协议更容易推理(???)。该方法的结果是，验证者在协议的最后要去
   处理累计的盲化因子，所以在协议一开始就推导一个（与第一种方法）等价的 $C'$ 就没有必要了。

   - $C'$ is also an input to the random oracle for $\xi_0$; in Halo 2 we utilize a
     transcript that has already committed to the equivalent components of $C'$ prior to
     sampling $z$.（？？？）


2. The $\text{PC}_\text{DL}.\text{SuccinctCheck}$ subroutine (Figure 2 of BCMS20) computes
   the initial group element $C_0$ by adding $[v] H' = [v \epsilon] H$, which requires two
   scalar multiplications. Instead, we subtract $[v] G_0$ from the original commitment $P$,
   so that we're effectively opening the polynomial at the point to the value zero. The
   computation $[v] G_0$ is more efficient in the context of recursion because $G_0$ is a
   fixed base (so we can use lookup tables).

2. $\text{PC}_\text{DL}.\text{SuccinctCheck}$ 子程序 (BCMS20的图2所示) 会通过增加 $[v] H' = [v \epsilon] H$
   来产生最开始的群元素 $C_0$，而这需要两次scalar乘。与之不同，我们的方法是用最初的承诺 $P$ 减去 $[v] G_0$，如此
   我们就可以在0处很高效地打开多项式。而$[v] G_0$的计算在递归证明的情况下具有更高效率，这是因为 $G_0$ 是一个固定的基
   （所以我们可以用查表(lookup tables)方法）。

