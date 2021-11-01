# Elliptic curves

# 椭圆曲线
Elliptic curves constructed over finite fields are another important cryptographic tool.

在有限域上构造的椭圆曲线是密码学的另一个重要工具。

We use elliptic curves because they provide a cryptographic [group](fields.md#Groups),
i.e. a group in which the discrete logarithm problem (discussed below) is hard.

椭圆曲线就是一个密码学的[群](fields.md#Groups)，即：该群上的离散对数问题是困难的。

There are several ways to define the curve equation, but for our purposes, let
$\mathbb{F}_p$ be a large (255-bit) field, and then let the set of solutions $(x, y)$ to
$y^2 = x^3 + b$ for some constant $b$ define the $\mathbb{F}_p$-rational points on an
elliptic curve $E(\mathbb{F}_p)$. These $(x, y)$ coordinates are called "affine
coordinates". Each of the $\mathbb{F}_p$-rational points, together with a "point at
infinity" $\mathcal{O}$ that serves as the group identity, can be interpreted as an
element of a group. By convention, elliptic curve groups are written additively.

有许多种定义椭圆方程的方法，单就我们来说，我们做如下定义，
$\mathbb{F}_p$ 是一个巨大的域 (255-bit)，有方程 $y^2 = x^3 + b$ 其中 $b$ 是一个常数，
$(x, y)$ 是它的解，其所有解组成一个集合，我们就称该集合就是在椭圆曲线 $E(\mathbb{F}_p)$ 上的 $\mathbb{F}_p$-有理点集。
这些坐标 $(x, y)$ 就被称为“仿射坐标”。$\mathbb{F}_p$-有理点集中的所有点与“无穷远点” $\mathcal{O}$ 就组成了一个群，其中 $\mathcal{O}$ 就是该群的单位元。通常椭圆曲线群用加法来表示。

![](https://i.imgur.com/JvLS6yE.png)
*"Three points on a line sum to zero, which is the point at infinity."*

*"共线三点的和是 $0$, 也就是无穷远点。"*

The group addition law is simple: to add two points together, find the line that
intersects both points and obtain the third point, and then negate its $y$-coordinate. The
case that a point is being added to itself, called point doubling, requires special
handling: we find the line tangent to the point, and then find the single other point that
intersects this line and then negate. Otherwise, in the event that a point is being
"added" to its negation, the result is the point at infinity.

群加法法则很简单：两个点相加，就是找到过两个点的直线并与椭圆曲线相交于第三个点，并将该点的 $y$ 坐标取相反数。
自己加自己，也就是二倍点运算，需要进行一些特殊处理：我们求出直线在该点的正切值，进而求出该直线交椭圆曲线的另一点，然后再取逆。
反之，若一个点与它的逆元相加，结果就是无穷远点。

The ability to add and double points naturally gives us a way to scale them by integers,
called _scalars_. The number of points on the curve is the group order. If this number
is a prime $q$, then the scalars can be considered as elements of a _scalar field_,
$\mathbb{F}_q$.

通过点加和二倍点运算，我们很自然就能计算点的标量乘法，即点乘以一个整数，这些整数就被称为 _数量_。
椭圆曲线点的个数就是群的阶。如果这个阶是个素数 $q$，于是所有的数量就可以视为 _数量域_ $\mathbb{F}_q$ 中的元素。

Elliptic curves, when properly designed, have an important security property. Given two
random elements $G, H \in E(\mathbb{F}_p)$ finding $a$ such that $[a] G = H$, otherwise
known as the discrete log of $H$ with respect to $G$, is considered computationally
infeasible with classical computers. This is called the elliptic curve discrete log
assumption.

精心设计的椭圆曲线有一个很重要的安全性质。假定 $G, H$ 是 $E(\mathbb{F}_p)$ 上的两个随机元素，找到一个 $a$ 使得 $[a] G = H$，
也就是求解关于 $H$ 的对应于 $G$ 的离散对数，在当前经典计算机上，是计算不可能的。这就是椭圆曲线的离散对数假设。


If an elliptic curve group $\mathbb{G}$ has prime order $q$ (like the ones used in Halo 2),
then it is a finite cyclic group. Recall from the section on [groups](fields.md#Groups)
that this implies it is isomorphic to $\mathbb{Z}/q\mathbb{Z}$, or equivalently, to the
scalar field $\mathbb{F}_q$. Each possible generator $G$ fixes the isomorphism; then
an element on the scalar side is precisely the discrete log of the corresponding group
element with respect to $G$. In the case of a cryptographically secure elliptic curve,
the isomorphism is hard to compute in the $\mathbb{G} \rightarrow \mathbb{F}_q$ direction
because the elliptic curve discrete log problem is hard.

如果椭圆曲线群 $\mathbb{G}$ 的阶是素数 $q$ (正如我们在 Halo 2 中所使用的)，
则它是一个有限循环群。由上一节 [群](fields.md#groups) 可知，这就是说这个椭圆曲线群与 $\mathbb{Z}/q\mathbb{Z}$ 是同构的，
等价地，与scalar域 $\mathbb{F}_q$ 是同构的。Each possible generator $G$ fixes the isomorphism;（？？？）
进而scalar中的元素就是由 $G$ 产生的一个群元素的离散对数。在密码学安全的椭圆曲线下，即便同构，也很难在 $\mathbb{G} \rightarrow \mathbb{F}_q$
这种情况下计算，因为椭圆曲线上的离散对数问题是困难的。

> It is sometimes helpful to make use of this isomorphism by thinking of group-based
> cryptographic protocols and algorithms in terms of the scalars instead of in terms of
> the group elements. This can make proofs and notation simpler.
>
> 有时利用这种同构性质是很有帮助的，方式是在scalars上思考基于群的密码学的协议或者算法，而不是在群元素上。
> 这会使得证明和表示更加简单。
>
> For instance, it has become common in papers on proof systems to use the notation $[x]$
> to denote a group element with discrete log $x$, where the generator is implicit.
>
> 举例来说，现在在有关证明系统的论文中已经普遍采用 $[x]$ 来代表一个有着离散对数 $x$ 的群元素，显然这种表示中已经隐去了生成元。
>
> We also used this idea in the
> "[distinct-x theorem](https://zips.z.cash/protocol/protocol.pdf#thmdistinctx)",
> in order to prove correctness of optimizations
> [for elliptic curve scalar multiplication](https://github.com/zcash/zcash/issues/3924)
> in Sapling, and an endomorphism-based optimization in Appendix C of the original
> [Halo paper](https://eprint.iacr.org/2019/1021.pdf).
>
> 我们也如下课题中采用了此一思想，
> "[distinct-x theorem](https://zips.z.cash/protocol/protocol.pdf#thmdistinctx)",
> 用来证明
> [for elliptic curve scalar multiplication](https://github.com/zcash/zcash/issues/3924)中优化的正确性
> 以及原始
> [Halo paper](https://eprint.iacr.org/2019/1021.pdf) 附录C中所提及的基于同态的优化。

## Curve arithmetic

## 椭圆曲线算术

### Point doubling

### 二倍点运算

The simplest situation is doubling a point $(x_0, y_0)$. Continuing with our example
$y^2 = x^3 + b$, this is done first by computing the derivative

求点 $(x_0, y_0)$ 的 $2$ 倍是最简单的情况。继续我们前文的例子，设椭圆曲线是 $y^2 = x^3 + b$， 所以第一步计算就是求如下导数：
$$
\lambda = \frac{\mathrm{d}y}{\mathrm{d}x} = \frac{3x^2}{2y}.
$$

To obtain expressions for $(x_1, y_1) = (x_0, y_0) + (x_0, y_0),$ we consider 

为了得到计算 $(x_1, y_1) = (x_0, y_0) + (x_0, y_0)$ 的公式，考虑

$$
\begin{aligned}
\frac{-y_1 - y_0}{x_1 - x_0} = \lambda &\implies -y_1 = \lambda(x_1 - x_0) + y_0 \\
&\implies \boxed{y_1 = \lambda(x_0 - x_1) - y_0}.
\end{aligned}
$$

To get the expression for $x_1,$ we substitute $y = \lambda(x_0 - x) - y_0$ into the
elliptic curve equation:

为了得到 $x_1$ 的公式，将 $y = \lambda(x_0 - x) - y_0$ 带入椭圆曲线方程：

$$
\begin{aligned}
y^2 = x^3 + b &\implies (\lambda(x_0 - x) - y_0)^2 = x^3 + b \\
&\implies x^3 - \lambda^2 x^2 + \cdots = 0 \leftarrow\text{(rearranging terms)} \\
&= (x - x_0)(x - x_0)(x - x_1) \leftarrow\text{(known roots $x_0, x_0, x_1$)} \\
&= x^3 - (x_0 + x_0 + x_1)x^2 + \cdots.
\end{aligned}
$$

Comparing coefficients for the $x^2$ term gives us
$\lambda^2 = x_0 + x_0 + x_1 \implies \boxed{x_1 = \lambda^2 - 2x_0}.$

通过比较 $x^2$ 项的系数，我们就得到了 $\lambda^2 = x_0 + x_0 + x_1 \implies \boxed{x_1 = \lambda^2 - 2x_0}$。


### Projective coordinates
### 射影坐标
This unfortunately requires an expensive inversion of $2y$. We can avoid this by arranging
our equations to "defer" the computation of the inverse, since we often do not need the
actual affine $(x', y')$ coordinate of the resulting point immediately after an individual
curve operation. Let's introduce a third coordinate $Z$ and scale our curve equation by
$Z^3$ like so:

上面计算中有求 $2y$ 的逆元的计算，而求逆运算是十分昂贵的。我们可以通过对椭圆曲线方程进行变换来“推迟”求逆运算，
因为通常在一个单独的曲线运算之后，通常并不马上就要得到相关结果的仿射坐标 $(x', y')$。基于此一想法，我们引入第三个坐标
$Z$ 并在方程两边同乘以 $Z^3$。如下所示：

$$
Z^3 y^2 = Z^3 x^3 + Z^3 b
$$

特别的，我们最开始的椭圆曲线就是 $Z = 1$ 的情况。令仿射坐标点 $(x, y)$ 可以表示成以下方式
$X = xZ$, $Y = yZ$ and $Z \neq 0$，则我们就得到了一条[同态射影曲线](https://en.wikipedia.org/wiki/Homogeneous_coordinates)

$$
Y^2 Z = X^3 + Z^3 b.
$$

Obtaining $(x, y)$ from $(X, Y, Z)$ is as simple as computing $(X/Z, Y/Z)$ when
$Z \neq 0$. (When $Z = 0,$ we are dealing with the point at infinity $O := (0:1:0)$.) In
this form, we now have a convenient way to defer the inversion required by doubling a
point. The general strategy is to express $x', y'$ as rational functions using $x = X/Z$
and $y = Y/Z$, rearrange to make their denominators the same, and then take the resulting
point $(X, Y, Z)$ to have $Z$ be the shared denominator and $X = x'Z, Y = y'Z$.

将 $(X, Y, Z)$ 转成 $(x, y)$ 是很简单的，只需要在当 $Z \neq 0$ 时计算 $(X/Z, Y/Z)$。
(当 $Z = 0$ 时，它就是一个无穷远点 $O := (0:1:0)$)。在该种表示形式下，计算二倍点时所需要的求逆运算就被推迟了。
一个通常的方法是将 $x', y'$ 表示成 $x = X/Z$
和 $y = Y/Z$，转换方程使得分母相同，进一步使结果点 $(X, Y, Z)$ 也有一样的分母，使得 $X = x'Z, Y = y'Z$。

> Projective coordinates are often, but not always, more efficient than affine
> coordinates. There may be exceptions to this when either we have a different way to
> apply Montgomery's trick, or when we're in the circuit setting where multiplications and
> inversions are about equally as expensive (at least in terms of circuit size).

> 射影坐标只是经常，但是并不总是比仿射坐标效率更好。当我们采用不同的方式运用Montgomery's trick，
> 或者在我们的电路设置中乘法和求逆复杂度形同的情况下（至少是电路规模），就绝非如此。

The following shows an example of doubling a point $(X, Y, Z) = (xZ, yZ, Z)$ without an
inversion. Substituting with $X, Y, Z$ gives us

以下，就给出了基于 $(X, Y, Z) = (xZ, yZ, Z)$ 的二倍点的计算方法，该方法就是不用求逆的。代入 $X, Y, Z$ 可得：

$$
\lambda = \frac{3x^2}{2y} = \frac{3(X/Z)^2}{2(Y/Z)} = \frac{3 X^2}{2YZ}
$$

and gives us

进一步得出
$$
\begin{aligned}
x' &= \lambda^2 - 2x \\
&= \lambda^2 - \frac{2X}{Z} \\
&= \frac{9 X^4}{4Y^2Z^2} - \frac{2X}{Z} \\
&= \frac{9 X^4 - 8XY^2Z}{4Y^2Z^2} \\
&= \frac{18 X^4 Y Z - 16XY^3Z^2}{8Y^3Z^3} \\
\\
y' &= \lambda (x - x') - y \\
&= \lambda (\frac{X}{Z} - \frac{9 X^4 - 8XY^2Z}{4Y^2Z^2}) - \frac{Y}{Z} \\
&= \frac{3 X^2}{2YZ} (\frac{X}{Z} - \frac{9 X^4 - 8XY^2Z}{4Y^2Z^2}) - \frac{Y}{Z} \\
&= \frac{3 X^3}{2YZ^2} - \frac{27 X^6 - 24X^3Y^2Z}{8Y^3Z^3} - \frac{Y}{Z} \\
&= \frac{12 X^3Y^2Z - 8Y^4Z^2 - 27 X^6 + 24X^3Y^2Z}{8Y^3Z^3}
\end{aligned}
$$

Notice how the denominators of $x'$ and $y'$ are the same. Thus, instead of computing
$(x', y')$ we can compute $(X, Y, Z)$ with $Z = 8Y^3Z^3$ and $X, Y$ set to the
corresponding numerators such that $X/Z = x'$ and $Y/Z = y'$. This completely avoids the
need to perform an inversion when doubling, and something analogous to this can be done
when adding two distinct points.

注意我们是如何让 $x'$ 和 $y'$ 的分母相同的。综上所述，我们通过令 $Z = 8Y^3Z^3$，并随之令 $X/Z = x'$ 和 $Y/Z = y'$，
就用射影坐标 $(X, Y, Z)$ 的二倍点计算完全替代了直接求仿射坐标 $(x', y')$ 二倍点的计算。通过这种方式，
我们完全避免了在仿射坐标二倍点的计算中所必需的求逆运算，此一方法也会运用到两个不同点的加法中去。

### Point addition

### 点加
We now add two points with distinct $x$-coordinates, $P = (x_0, y_0)$ and $Q = (x_1, y_1),$
where $x_0 \neq x_1,$ to obtain $R = P + Q = (x_2, y_2).$ The line $\overline{PQ}$ has slope
$$\lambda = frac{y_1 - y_0}{x_1 - x_0} \implies y - y_0 = \lambda \cdot (x - x_0).$$

现在计算两个不同点的加法, $P = (x_0, y_0)$ 加 $Q = (x_1, y_1),$
其中 $x_0 \neq x_1,$ 得到 $R = P + Q = (x_2, y_2).$ 直线 $\overline{PQ}$ 的斜率是
$$\lambda = frac{y_1 - y_0}{x_1 - x_0} \implies y - y_0 = \lambda \cdot (x - x_0).$$

Using the expression for $\overline{PQ}$, we compute $y$-coordinate $-y_2$ of $-R$ as:
$$-y_2 - y_0 = \lambda \cdot (x_2 - x_0) \implies \boxed{y_2 = (x_0 - x_2) - y_0}.$$

通过 $\overline{PQ}$ 的表达式，我们可以计算 $-R$ 的 $y$-坐标，即 $-y_2$ 为：
$$-y_2 - y_0 = \lambda \cdot (x_2 - x_0) \implies \boxed{y_2 = (x_0 - x_2) - y_0}.$$

Plugging the expression for $\overline{PQ}$ into the curve equation $y^2 = x^3 + b$ yields

把直线 $\overline{PQ}$ 的方程带入曲线方程 $y^2 = x^3 + b$ 可得
$$
\begin{aligned}
y^2 = x^3 + b &\implies (\lambda \cdot (x - x_0) + y_0)^2 = x^3 + b \\
&\implies x^3 - \lambda^2 x^2 + \cdots = 0 \leftarrow\text{(rearranging terms)} \\
&= (x - x_0)(x - x_1)(x - x_2) \leftarrow\text{(known roots $x_0, x_1, x_2$)} \\
&= x^3 - (x_0 + x_1 + x_2)x^2 + \cdots.
\end{aligned}
$$

Comparing coefficients for the $x^2$ term gives us
$\lambda^2 = x_0 + x_1 + x_2 \implies \boxed{x_2 = \lambda^2 - x_0 - x_1}$.

比较 $x^2$ 的系数，可知
$\lambda^2 = x_0 + x_1 + x_2 \implies \boxed{x_2 = \lambda^2 - x_0 - x_1}$.

----------

Important notes:

* There exist efficient formulae[^complete-formulae] for point addition that do not have
  edge cases (so-called "complete" formulae) and that unify the addition and doubling
  cases together. The result of adding a point to its negation using those formulae
  produces $Z = 0$, which represents the point at infinity.

* 对于无边界情况的点加，存在一个高效的公式[^complete-formulae] (所谓的“完全”公式)，该公式统一了
  点加和二倍点的计算。使用该公式计算一个点和它的逆的加法，将得到 $Z = 0$，也就意味着无穷远点。

* In addition, there are other models like the Jacobian representation where
  $(x, y) = (xZ^2, yZ^3, Z)$ where the curve is rescaled by $Z^6$ instead of $Z^3$, and
  this representation has even more efficient arithmetic but no unified/complete formulae.

* 另外，还有其他的计算模型，比如雅可比表示形式，即：$(x, y) = (xZ^2, yZ^3, Z)$，相比于 $Z^3$，这是由乘以
  $Z^6$ 的变换得到的。该表示有更好的算术性能，但是却没有一个统一的抑或是完全的公式。

* We can easily compare two curve points $(X_1, Y_1, Z_1)$ and $(X_2, Y_2, Z_2)$ for
  equality in the homogenous projective coordinate space by "homogenizing" their
  Z-coordinates; the checks become $X_1 Z_2 = X_2 Z_1$ and $Y_1 Z_2 = Y_2 Z_1$.

* 在两个同质化的射影坐标空间中，我们可以很方便地比较两个曲线上的点 $(X_1, Y_1, Z_1)$ 和 $(X_2, Y_2, Z_2)$ 
  是否相等。方法就是“同一化”他们的 Z-坐标，所以比较两个点是否是同一个点，只需检验如下等式是否成立：
  $X_1 Z_2 = X_2 Z_1$ 并且 $Y_1 Z_2 = Y_2 Z_1$。

## Curve endomorphisms

Imagine that $\mathbb{F}_p$ has a primitive cube root of unity, or in other words that
$3 | p - 1$ and so an element $\zeta_p$ generates a $3$-order multiplicative subgroup.
Notice that a point $(x, y)$ on our example elliptic curve $y^2 = x^3 + b$ has two cousin
points: $(\zeta_p x, \zeta_p^2 x)$, because the computation $x^3$ effectively kills the
$\zeta$ component of the $x$-coordinate. Applying the map $(x, y) \mapsto (\zeta_p x, y)$
is an application of an endomorphism over the curve. The exact mechanics involved are
complicated, but when the curve has a prime $q$ number of points (and thus a prime
"order") the effect of the endomorphism is to multiply the point by a scalar in
$\mathbb{F}_q$ which is also a primitive cube root $\zeta_q$ in the scalar field.

## Curve point compression

## 椭圆曲线点压缩
Given a point on the curve $P = (x,y)$, we know that its negation $-P = (x, -y)$ is also
on the curve. To uniquely specify a point, we need only encode its $x$-coordinate along
with the sign of its $y$-coordinate.

给定曲线上一个点 $P = (x,y)$，它的逆 $-P = (x, -y)$ 也在曲线上。
为了唯一表示该点，我们可以仅将其 $x$-坐标和 $y$-坐标的符号进行编码即可。

### Serialization

### 序列化
As mentioned in the [Fields](./fields.md) section, we can interpret the least significant
bit of a field element as its "sign", since its additive inverse will always have the
opposite LSB. So we record the LSB of the $y$-coordinate as `sign`.

正如在 [Fields](./fields.md) 这一节所提到的，我们可以将一个域元素的最低位作为其“符号”。
因为它的加法逆元总是有相反的最低位。所以我们就把 $y$-坐标的最低为作为其“符号”。

Pallas and Vesta are defined over the $\mathbb{F}_p$ and $\mathbb{F}_q$ fields, which
elements can be expressed in $255$ bits. This conveniently leaves one unused bit in a
32-byte representation. We pack the $y$-coordinate `sign` bit into the highest bit in
the representation of the $x$-coordinate:

Pallas 和 Vesta 是定义在域 $\mathbb{F}_p$ 和域 $\mathbb{F}_q$ 上的，二者的元素都是 $255$ bits。
它的表示需要32个字节，同时还会剩余一个bit。于是，我们就将 $y$-坐标的 `符号位` 放入 $x$-坐标的最高位即可：

```text
         <----------------------------------- x --------------------------------->
Enc(P) = [_ _ _ _ _ _ _ _] [_ _ _ _ _ _ _ _] ... [_ _ _ _ _ _ _ _] [_ _ _ _ _ _ _ sign]
          ^                <------------------------------------->                 ^
         LSB                              30 bytes                                MSB
```

The "point at infinity" $\mathcal{O}$ that serves as the group identity, does not have an
affine $(x, y)$ representation. However, it turns out that there are no points on either
the Pallas or Vesta curve with $x = 0$ or $y = 0$. We therefore use the "fake" affine
coordinates $(0, 0)$ to encode $\mathcal{O}$, which results in the all-zeroes 32-byte
array.

“无穷远点” $\mathcal{O}$ 作为群的单位元并没有相应的仿射坐标 $(x, y)$ 表示。但是，可以证明，不论是
Pallas 还是 Vesta 曲线都没有 $x = 0$ 或 $y = 0$ 的点。因此，我们就可以使用“假的”仿射坐标 $(0, 0)$
来代表 $\mathcal{O}$，那么它的实际表示也就是一个全为 $0$ 的32字节的数组。

### Deserialization

### 反序列化
When deserializing a compressed curve point, we first read the most significant bit as
`ysign`, the sign of the $y$-coordinate. Then, we set this bit to zero to recover the
original $x$-coordinate.

一个压缩的椭圆曲线的点的放序列化方法是，首先读取最高位作为 `y符号`，这个符号就是 $y$-坐标的符号。
第二步，将该位设为0以得到真正的 $x$-坐标。

If $x = 0, y = 0,$ we return the "point at infinity" $\mathcal{O}$. Otherwise, we proceed
to compute $y = \sqrt{x^3 + b}.$ Here, we read the least significant bit of $y$ as `sign`.
If `sign == ysign`, we already have the correct sign and simply return the curve point
$(x, y)$. Otherwise, we negate $y$ and return $(x, -y)$.

如果 $x = 0, y = 0,$ 我们就返回一个“无穷远点” $\mathcal{O}$。反之，我们就进一步计算 $y = \sqrt{x^3 + b}。
接下来，我们可以读取计算出来的 $y$-坐标的最低位，也就是它的 `符号`，如果
`sign == ysign`, 我们就得到了正确的结果，只需将 $(x, y)$ 返回即可。反之，我们就对 $y$ 取反并返回 $(x, -y)$。

## Cycles of curves

## 曲线循环
Let $E_p$ be an elliptic curve over a finite field $\mathbb{F}_p,$ where $p$ is a prime.
We denote this by $E_p/\mathbb{F}_p.$ and we denote the group of points of $E_p$ over
$\mathbb{F}_p,$ with order $q = \#E(\mathbb{F}_p).$ For this curve, we call $\mathbb{F}_p$
the "base field" and  $\mathbb{F}_q$ the "scalar field".

令 $E_p$ 是定义在有限域 $\mathbb{F}_p,$ 上的椭圆曲线，其中 $p$ 是素数，记为 $E_p/\mathbb{F}_p$，
并且我们记定义在 $\mathbb{F}_p$ 上的群 $E_p$ 的阶是 q = \#E(\mathbb{F}_p)$。对这条曲线，我们称
$\mathbb{F}_p$ 为“基域”，$\mathbb{F}_q$ 为“数量域”。

We instantiate our proof system over the elliptic curve $E_p/\mathbb{F}_p$. This allows us
to prove statements about $\mathbb{F}_q$-arithmetic circuit satisfiability.

我们在 $E_p/\mathbb{F}_p$ 实例化我们的证明系统。这允许我们证明关于 $\mathbb{F}_q$ 算术电路的满足性的命题。

> **(aside) If our curve $E_p$ is over $\mathbb{F}_p,$ why is the arithmetic circuit instead in $\mathbb{F}_q$?**
> The proof system is basically working on encodings of the scalars in the circuit (or
> more precisely, commitments to polynomials whose coefficients are scalars). The scalars
> are in $\mathbb{F}_q$ when their encodings/commitments are elliptic curve points in
> $E_p/\mathbb{F}_p$.

> **(备注) 既然椭圆曲线 $E_p$ 是定义在 $\mathbb{F}_p$ 上，那为社么算术电路确实是现在 $\mathbb{F}_q$ 上?**
> 证明系统基本上就是工作在由scalars组成的电路上 （或者，更准确地，scalars其实是要承诺的多项式的系数）。
> scalars就在 $\mathbb{F}_q$ 上，当它们编码或者承诺的对象是椭圆曲线 $E_p/\mathbb{F}_p$ 上的点。


However, most of the verifier's arithmetic computations are over the base field
$\mathbb{F}_p,$ and are thus efficiently expressed as an $\mathbb{F}_p$-arithmetic
circuit.

与之相应，绝大多数验证者的算术运算都是在基域 $\mathbb{F}_p$ 上的，因此可以用 $\mathbb{F}_p$ 的算数电路高效的表示。

> **(aside) Why are the verifier's computations (mainly) over $\mathbb{F}_p$?**
> The Halo 2 verifier actually has to perform group operations using information output by
> the circuit. Group operations like point doubling and addition use arithmetic in
> $\mathbb{F}_p$, because the coordinates of points are in $\mathbb{F}_p.$ 

> **(备注)为什么验证计算 (主要) 发生在 $\mathbb{F}_p$? 上**
> 事实上，Halo 2 对电路的输出做一些群运算。而群运算，就如二倍点和点加就会用到域 $\mathbb{F}_p$ 上的算术运算，
> 因为点的坐标都在域 $\mathbb{F}_p$ 上。

This motivates us to construct another curve with scalar field $\mathbb{F}_p$, which has
an $\mathbb{F}_p$-arithmetic circuit that can efficiently verify proofs from the first
curve. As a bonus, if this second curve had base field $E_q/\mathbb{F}_q,$ it would
generate proofs that could be efficiently verified in the first curve's
$\mathbb{F}_q$-arithmetic circuit. In other words, we instantiate a second proof system
over $E_q/\mathbb{F}_q,$ forming a 2-cycle with the first:

这促使我们构造以 $\mathbb{F}_p$ 为数量域的另外一条曲线，这条曲线实现一个基于 $\mathbb{F}_p$ 的算术电路，该电路就可以有效地
来验证来自第一条曲线的证明。更美好的是，如果这条曲线的基域又是 $E_q/\mathbb{F}_q$，那么它生成的证明就又可以被第一条曲线的 $\mathbb{F}_q$
算术电路所验证。于是就形成了一个2-曲线循环：

![](https://i.imgur.com/bNMyMRu.png)

### TODO: Pallas-Vesta curves
Reference: https://github.com/zcash/pasta

## Hashing to curves

## Hashing到椭圆曲线（？？）

Sometimes it is useful to be able to produce a random point on an elliptic curve
$E_p/\mathbb{F}_p$ corresponding to some input, in such a way that no-one will know its
discrete logarithm (to any other base).

有时将某个输入映射到椭圆曲线 $E_p/\mathbb{F}_p$ 上的随机一个点是很有用的，这样，就没人能知道它的离散对数（相对其他基）。

This is described in detail in the [Internet draft on Hashing to Elliptic Curves][cfrg-hash-to-curve].
Several algorithms can be used depending on efficiency and security requirements. The
framework used in the Internet Draft makes use of several functions:

This is described in detail in the [Internet draft on Hashing to Elliptic Curves][cfrg-hash-to-curve].
Several algorithms can be used depending on efficiency and security requirements. The
framework used in the Internet Draft makes use of several functions:

相关细节在 [Internet draft on Hashing to Elliptic Curves][cfrg-hash-to-curve] 中有所描述。
针对效率和安全性，有各种各样的算法完成这个工作。Internet Draft 所使用的框架就运用了多种功能：

* ``hash_to_field``: takes a byte sequence input and maps it to a element in the base
  field $\mathbb{F}_p$

* ``hash_to_field``: 将一个字节序列映射到基域 $\mathbb{F}_p$ 上的一个元素
  
* ``map_to_curve``: takes an $\mathbb{F}_p$ element and maps it to $E_p$.

* ``map_to_curve``: 将 $\mathbb{F}_p$ 上的元素映射到 $E_p$ 上。

[cfrg-hash-to-curve]: https://datatracker.ietf.org/doc/draft-irtf-cfrg-hash-to-curve/?include_text=1

### TODO: Simplified SWU
Reference: https://eprint.iacr.org/2019/403.pdf

## References
[^complete-formulae]: [Renes, J., Costello, C., & Batina, L. (2016, May). "Complete addition formulas for prime order elliptic curves." In Annual International Conference on the Theory and Applications of Cryptographic Techniques (pp. 403-428). Springer, Berlin, Heidelberg.](https://eprint.iacr.org/2015/1060)
