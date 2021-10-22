# 椭圆曲线

在有限域上构造的椭圆曲线是密码学的另一个重要工具。

椭圆曲线就是一个密码[群](fields.md#Groups)，即：该群上的离散对数问题是困难的。

有许多种定义椭圆方程的方法，就我们而言，我们做如下定义，
$\mathbb{F}_p$ 是一个巨大的域 (255-bit)，有方程 $y^2 = x^3 + b$ 其中 $b$ 是一个常数，
$(x, y)$ 是它的解，其所有解组成一个集合，我们就称该集合就是在椭圆曲线 $E(\mathbb{F}_p)$ 上的 $\mathbb{F}_p$-有理点集。
这些坐标 $(x, y)$ 就被称为“仿射坐标”。$\mathbb{F}_p$-有理点集中的所有点与“无穷远点” $\mathcal{O}$ 就组成了一个群，其中 $\mathcal{O}$ 就是该群的单位元。通常椭圆曲线群用加法来表示。

![](https://i.imgur.com/JvLS6yE.png)
*"共线三点的和是 $0$, 也就是无穷远点。"*

群加法法则很简单：两个点相加，就是找到过两个点的直线并与椭圆曲线相交于第三个点，并将该点的 $y$ 坐标取相反数。
自己加自己，也就是二倍点运算，需要进行一些特殊处理：我们求出直线在该点的正切值，进而求出该直线交椭圆曲线的另一点，然后再取逆。
反之，若一个点与它的逆元相加，结果就是无穷远点。

通过点加和二倍点运算，我们很自然就能计算点的标量乘法，即点乘以一个整数，这些整数就被称为 _数量_。
椭圆曲线点的个数就是群的阶。如果这个阶是个素数 $q$，于是所有的数量就可以视为 _数量域_ $\mathbb{F}_q$ 中的元素。

精心设计的椭圆曲线有一个很重要的安全性质。假定 $G, H$ 是 $E(\mathbb{F}_p)$ 上的两个随机元素，找到一个 $a$ 使得 $[a] G = H$，
也就是求解关于 $H$ 的对应于 $G$ 的离散对数，在当前经典计算机上，是计算不可能的。这就是椭圆曲线的离散对数假设。

如果椭圆曲线群 $\mathbb{G}$ 的阶是素数 $q$ (正如我们在 Halo 2 中所使用的)，
则它是一个有限循环群。由上一节 [群](fields.md#groups) 可知，这就是说这个椭圆曲线群与 $\mathbb{Z}/q\mathbb{Z}$ 是同构的，
等价地，与scalar域 $\mathbb{F}_q$ 是同构的。Each possible generator $G$ fixes the isomorphism;（？？？）
进而scalar中的元素就是由 $G$ 产生的一个群元素的离散对数。在密码学安全的椭圆曲线下，即便同构，也很难在 $\mathbb{G} \rightarrow \mathbb{F}_q$
这种情况下计算，因为椭圆曲线上的离散对数问题是困难的。

> 有时利用这种同构性质是很有帮助的，方式是在scalars上思考基于群的密码学的协议或者算法，而不是在群元素上。
> 这会使得证明和表示更加简单。
> 举例来说，现在在有关证明系统的论文中已经普遍采用 $[x]$ 来代表一个有着离散对数 $x$ 的群元素，显然这种表示中已经隐去了生成元。
>
> 我们也如下课题中采用了此一思想，
> "[distinct-x theorem](https://zips.z.cash/protocol/protocol.pdf#thmdistinctx)",
> 用来证明
> [for elliptic curve scalar multiplication](https://github.com/zcash/zcash/issues/3924)中优化的正确性
> 以及原始
> [Halo paper](https://eprint.iacr.org/2019/1021.pdf) 附录C中所提及的基于同态的优化。

## 椭圆曲线算术

### 二倍点运算

求点 $(x_0, y_0)$ 的 $2$ 倍是最简单的情况。继续我们前文的例子，设椭圆曲线是 $y^2 = x^3 + b$， 所以第一步计算就是求如下导数：
$$
\lambda = \frac{\mathrm{d}y}{\mathrm{d}x} = \frac{3x^2}{2y}.
$$

为了得到计算 $(x_1, y_1) = (x_0, y_0) + (x_0, y_0)$ 的公式，考虑

$$
\begin{aligned}
\frac{-y_1 - y_0}{x_1 - x_0} = \lambda &\implies -y_1 = \lambda(x_1 - x_0) + y_0 \\
&\implies \boxed{y_1 = \lambda(x_0 - x_1) - y_0}.
\end{aligned}
$$

为了得到 $x_1$ 的公式，将 $y = \lambda(x_0 - x) - y_0$ 带入椭圆曲线方程：

$$
\begin{aligned}
y^2 = x^3 + b &\implies (\lambda(x_0 - x) - y_0)^2 = x^3 + b \\
&\implies x^3 - \lambda^2 x^2 + \cdots = 0 \leftarrow\text{(rearranging terms)} \\
&= (x - x_0)(x - x_0)(x - x_1) \leftarrow\text{(known roots $x_0, x_0, x_1$)} \\
&= x^3 - (x_0 + x_0 + x_1)x^2 + \cdots.
\end{aligned}
$$

通过比较 $x^2$ 项的系数，我们就得到了 $\lambda^2 = x_0 + x_0 + x_1 \implies \boxed{x_1 = \lambda^2 - 2x_0}$。

### 射影坐标

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

将 $(X, Y, Z)$ 转成 $(x, y)$ 是很简单的，只需要在当 $Z \neq 0$ 时计算 $(X/Z, Y/Z)$。
(当 $Z = 0$ 时，它就是一个无穷远点 $O := (0:1:0)$)。在该种表示形式下，计算二倍点时所需要的求逆运算就被推迟了。
一个通常的方法是将 $x', y'$ 表示成 $x = X/Z$
和 $y = Y/Z$，转换方程使得分母相同，进一步使结果点 $(X, Y, Z)$ 也有一样的分母，使得 $X = x'Z, Y = y'Z$。

> 射影坐标只是在大多数时候，但是并不总是比仿射坐标效率更好。当我们采用不同的方式运用Montgomery's trick，
> 或者在我们的电路设置中乘法和求逆复杂度形同的情况下（至少是电路规模），就绝非如此。

以下，就给出了基于 $(X, Y, Z) = (xZ, yZ, Z)$ 的二倍点的计算方法，该方法就是不用求逆的。代入 $X, Y, Z$ 可得：

$$
\lambda = \frac{3x^2}{2y} = \frac{3(X/Z)^2}{2(Y/Z)} = \frac{3 X^2}{2YZ}
$$

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

注意我们是如何让 $x'$ 和 $y'$ 的分母相同的。综上所述，我们通过令 $Z = 8Y^3Z^3$，并随之令 $X/Z = x'$ 和 $Y/Z = y'$，
就用射影坐标 $(X, Y, Z)$ 的二倍点计算完全替代了直接求仿射坐标 $(x', y')$ 二倍点的计算。通过这种方式，
我们完全避免了在仿射坐标二倍点的计算中所必需的求逆运算，此一方法也会运用到两个不同点的加法中去。


### 点加

现在计算两个不同点的加法, $P = (x_0, y_0)$ 加 $Q = (x_1, y_1),$
其中 $x_0 \neq x_1,$ 得到 $R = P + Q = (x_2, y_2).$ 直线 $\overline{PQ}$ 的斜率是
$$\lambda = frac{y_1 - y_0}{x_1 - x_0} \implies y - y_0 = \lambda \cdot (x - x_0).$$


通过 $\overline{PQ}$ 的表达式，我们可以计算 $-R$ 的 $y$-坐标，即 $-y_2$ 为：
$$-y_2 - y_0 = \lambda \cdot (x_2 - x_0) \implies \boxed{y_2 = (x_0 - x_2) - y_0}.$$

把直线 $\overline{PQ}$ 的方程带入曲线方程 $y^2 = x^3 + b$ 可得
$$
\begin{aligned}
y^2 = x^3 + b &\implies (\lambda \cdot (x - x_0) + y_0)^2 = x^3 + b \\
&\implies x^3 - \lambda^2 x^2 + \cdots = 0 \leftarrow\text{(rearranging terms)} \\
&= (x - x_0)(x - x_1)(x - x_2) \leftarrow\text{(known roots $x_0, x_1, x_2$)} \\
&= x^3 - (x_0 + x_1 + x_2)x^2 + \cdots.
\end{aligned}
$$

比较 $x^2$ 的系数，可知
$\lambda^2 = x_0 + x_1 + x_2 \implies \boxed{x_2 = \lambda^2 - x_0 - x_1}$.

----------

重要提示：

* 对于无边界情况的点加，存在一个高效的公式[^complete-formulae] (所谓的“完全”公式)，该公式统一了
  点加和二倍点的计算。使用该公式计算一个点和它的逆的加法，将得到 $Z = 0$，也就意味着无穷远点。

* 另外，还有其他的计算模型，比如雅可比表示形式，即：$(x, y) = (xZ^2, yZ^3, Z)$，相比于 $Z^3$，这是由乘以
  $Z^6$ 的变换得到的。该表示有更好的算术性能，但是却没有一个统一的抑或是完全的公式。

* 在两个同质化的射影坐标空间中，我们可以很方便地比较两个曲线上的点 $(X_1, Y_1, Z_1)$ 和 $(X_2, Y_2, Z_2)$ 
  是否相等。方法就是“同一化”他们的 Z-坐标，所以比较两个点是否是同一个点，只需检验如下等式是否成立：
  $X_1 Z_2 = X_2 Z_1$ 并且 $Y_1 Z_2 = Y_2 Z_1$。

## 椭圆曲线自同态

假设 $\mathbb{F}_p$ 有三次本原单位根，或者说 $3 | p - 1$，所以就存在一个元素 $\zeta_p$，其能够产生一个 $3$-阶乘法子群。
注意，在示例椭圆曲线 $y^2 = x^3 + b$ 上的点 $(x, y)$ 有两个相关的点：$(\zeta_p x，\zeta_p^2 x)$，因为通过计算 $x^3$ 都能有效
消除 $x$-坐标的 $\zeta$ 组成部分。建立映射 $(x, y) \mapsto (\zeta_p x, y)$ 就是在曲线上建立了一个自同态映射。这里所涉及到的精确
机制是很复杂的，但是当一条曲线有素数 $q$ 个点（因而它的阶也是素数），自同态的作用就是对一个点 multiply the point by a scalar in
$\mathbb{F}_q$ which is also a primitive cube root $\zeta_q$ in the scalar field.（？？）

## 椭圆曲线点压缩

给定曲线上一个点 $P = (x,y)$，它的逆 $-P = (x, -y)$ 也在曲线上。
为了唯一表示该点，我们可以仅将其 $x$-坐标和 $y$-坐标的符号进行编码即可。

### 序列化

正如在 [Fields](./fields.md) 这一节所提到的，我们可以将一个域元素的最低位作为其“符号”。
因为它的加法逆元总是有相反的最低位。所以我们就把 $y$-坐标的最低为作为其“符号”。

Pallas 和 Vesta 是定义在域 $\mathbb{F}_p$ 和域 $\mathbb{F}_q$ 上的，二者的元素都是 $255$ bits。
它的表示需要32个字节，同时还会剩余一个bit。于是，我们就将 $y$-坐标的 `符号位` 放入 $x$-坐标的最高位即可：

```text
         <----------------------------------- x --------------------------------->
Enc(P) = [_ _ _ _ _ _ _ _] [_ _ _ _ _ _ _ _] ... [_ _ _ _ _ _ _ _] [_ _ _ _ _ _ _ sign]
          ^                <------------------------------------->                 ^
         LSB                              30 bytes                                MSB
```

“无穷远点” $\mathcal{O}$ 作为群的单位元并没有相应的仿射坐标 $(x, y)$ 表示。但是，可以证明，不论是
Pallas 还是 Vesta 曲线都没有 $x = 0$ 或 $y = 0$ 的点。因此，我们就可以使用“假的”仿射坐标 $(0, 0)$
来代表 $\mathcal{O}$，那么它的实际表示也就是一个全为 $0$ 的32字节的数组。


### 反序列化

一个压缩的椭圆曲线的点的放序列化方法是，首先读取最高位作为 `y符号`，这个符号就是 $y$-坐标的符号。
第二步，将该位设为0以得到真正的 $x$-坐标。

如果 $x = 0, y = 0,$ 我们就返回一个“无穷远点” $\mathcal{O}$。反之，我们就进一步计算 $y = \sqrt{x^3 + b}。
接下来，我们可以读取计算出来的 $y$-坐标的最低位，也就是它的 `符号`，如果
`sign == ysign`, 我们就得到了正确的结果，只需将 $(x, y)$ 返回即可。反之，我们就对 $y$ 取反并返回 $(x, -y)$。

## 曲线循环

令 $E_p$ 是定义在有限域 $\mathbb{F}_p,$ 上的椭圆曲线，其中 $p$ 是素数，记为 $E_p/\mathbb{F}_p$，
并且我们记定义在 $\mathbb{F}_p$ 上的群 $E_p$ 的阶是 q = \#E(\mathbb{F}_p)$。对这条曲线，我们称
$\mathbb{F}_p$ 为“基域”，$\mathbb{F}_q$ 为“数量域”。

我们在 $E_p/\mathbb{F}_p$ 实例化我们的证明系统。这允许我们证明关于 $\mathbb{F}_q$ 算术电路的满足性的命题。

> **(备注) 既然椭圆曲线 $E_p$ 是定义在 $\mathbb{F}_p$ 上，那为社么算术电路确实是现在 $\mathbb{F}_q$ 上?**
> 证明系统基本上就是工作在由scalars组成的电路上 （或者，更准确地，scalars其实是要承诺的多项式的系数）。
> scalars就在 $\mathbb{F}_q$ 上，当它们编码或者承诺的对象是椭圆曲线 $E_p/\mathbb{F}_p$ 上的点。

与之相应，绝大多数验证者的算术运算都是在基域 $\mathbb{F}_p$ 上的，因此可以用 $\mathbb{F}_p$ 的算数电路高效的表示。

> **(备注)为什么验证计算 (主要) 发生在 $\mathbb{F}_p$? 上**
> 事实上，Halo 2 对电路的输出做一些群运算。而群运算，就如二倍点和点加就会用到域 $\mathbb{F}_p$ 上的算术运算，
> 因为点的坐标都在域 $\mathbb{F}_p$ 上。

这促使我们构造以 $\mathbb{F}_p$ 为数量域的另外一条曲线，这条曲线实现一个基于 $\mathbb{F}_p$ 的算术电路，该电路就可以有效地
来验证来自第一条曲线的证明。更美好的是，如果这条曲线的基域又是 $E_q/\mathbb{F}_q$，那么它生成的证明就又可以被第一条曲线的 $\mathbb{F}_q$
算术电路所验证。于是就形成了一个2-曲线循环：

![](https://i.imgur.com/bNMyMRu.png)

### TODO: Pallas-Vesta curves
Reference: https://github.com/zcash/pasta

## Hashing到椭圆曲线（？？）

有时将某个输入映射到椭圆曲线 $E_p/\mathbb{F}_p$ 上的随机一个点是很有用的，这样，就没人能知道它的离散对数（相对其他基）。

相关细节在 [Internet draft on Hashing to Elliptic Curves][cfrg-hash-to-curve] 中有所描述。
针对效率和安全性，有各种各样的算法完成这个工作。Internet Draft 所使用的框架就运用了多种功能：

* ``hash_to_field``: 将一个字节序列映射到基域 $\mathbb{F}_p$ 上的一个元素

* ``map_to_curve``: 将 $\mathbb{F}_p$ 上的元素映射到 $E_p$ 上。

[cfrg-hash-to-curve]: https://datatracker.ietf.org/doc/draft-irtf-cfrg-hash-to-curve/?include_text=1

### TODO: Simplified SWU
Reference: https://eprint.iacr.org/2019/403.pdf

## References
[^complete-formulae]: [Renes, J., Costello, C., & Batina, L. (2016, May). "Complete addition formulas for prime order elliptic curves." In Annual International Conference on the Theory and Applications of Cryptographic Techniques (pp. 403-428). Springer, Berlin, Heidelberg.](https://eprint.iacr.org/2015/1060)
