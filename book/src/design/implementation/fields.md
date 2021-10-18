# 域

`halo2` 中使用的 [Pasta曲线](https://electriccoin.co/blog/the-pasta-curves-for-halo-2-and-beyond/) 是两条在有限域上均有阶为 $2^S$ 的大乘法子群的曲线。这意味着我们可以将有限域的阶写为 $p-1 \equiv 2^S \cdot T$ ，其中 $T$ 为奇数。对于 Pallas 和 Vesta 曲线，我们都有 $S=32$ 。

## Sarkar 平方根算法（基于查表的变式）

在 `halo2` 中，我们使用 [Sarkar2020](https://eprint.iacr.org/2020/1407.pdf) 论文中的技术计算平方根。该算法的思想是，我们可以将计算平方根的任务在每个乘法子群中进行拆分。

假设我们想要求 $u$ 模某个 Pasta 素数阶 $p$ 的平方根，（ $u$ 是 $\mathbb{Z}_p^\times$ 上的一个非零完全平方数），我们可以定义一个 $2^S$ 阶的 [单位根](../../background/fields.md#roots-of-unity) $g=z^T$ ，（ $z$ 是 $\mathbb{Z}_p^\times$ 中的非平方数），并计算如下辅助表格：

$$
gtab = \begin{bmatrix}
g^0 & g^1 & ... & g^{2^8 - 1} \\
(g^{2^8})^0 & (g^{2^8})^1 & ... & (g^{2^8})^{2^8 - 1} \\
(g^{2^{16}})^0 & (g^{2^{16}})^1 & ... & (g^{2^{16}})^{2^8 - 1} \\
(g^{2^{24}})^0 & (g^{2^{24}})^1 & ... & (g^{2^{24}})^{2^8 - 1}
\end{bmatrix}
$$

$$
invtab = \begin{bmatrix}
(g^{-2^{24}})^0 & (g^{-2^{24}})^1 & ... & (g^{-2^{24}})^{2^8 - 1}
\end{bmatrix}
$$

令 $v = u^{(T-1)/2}$ 。我们可以定义 $x = uv \cdot v = u^T$ 作为 $2^S$ 阶乘法子群上的一个元素。

令 $x_3 = x, x_2 = x_3^{2^8}, x_1 = x_2^{2^8}, x_0 = x_1^{2^8}.$

### 当 i = 0, 1 时
使用 $invtab$ 表，我们查找 $t_0$ 使得
$$
x_0 = (g^{-2^{24}})^{t_0} \implies x_0 \cdot g^{t_0 \cdot 2^{24}} = 1.
$$

定义 $\alpha_1 = x_1 \cdot (g^{2^{16}})^{t_0}.$

### 当 i = 2 时
查找 $t_1$ 使得
$$
\begin{array}{ll}
\alpha_1 = (g^{-2^{24}})^{t_1} &\implies x_1 \cdot (g^{2^{16}})^{t_0} = (g^{-2^{24}})^{t_1} \\
&\implies
x_1 \cdot g^{(t_0 + 2^8 \cdot t_1) \cdot 2^{16}} = 1.
\end{array}
$$

定义 $\alpha_2 = x_2 \cdot (g^{2^8})^{t_0 + 2^8 \cdot t_1}.$
         
### 当 i = 3 时
查找 $t_2$ 使得

$$
\begin{array}{ll}
\alpha_2 = (g^{-2^{24}})^{t_2} &\implies x_2 \cdot (g^{2^8})^{t_0 + 2^8\cdot {t_1}} = (g^{-2^{24}})^{t_2} \\
&\implies x_2 \cdot g^{(t_0 + 2^8 \cdot t_1 + 2^{16} \cdot t_2) \cdot 2^8} = 1.
\end{array}
$$

定义 $\alpha_3 = x_3 \cdot g^{t_0 + 2^8 \cdot t_1 + 2^{16} \cdot t_2}.$

### 最终结果
查找 $t_3$ 使得

$$
\begin{array}{ll}
\alpha_3 = (g^{-2^{24}})^{t_3} &\implies x_3 \cdot g^{t_0 + 2^8\cdot {t_1} + 2^{16} \cdot t_2} = (g^{-2^{24}})^{t_3} \\
&\implies x_3 \cdot g^{t_0 + 2^8 \cdot t_1 + 2^{16} \cdot t_2 + 2^{24} \cdot t_3} = 1.
\end{array}
$$

令 $t = t_0 + 2^8 \cdot t_1 + 2^{16} \cdot t_2 + 2^{24} \cdot t_3$.

我们写成

$$
\begin{array}{lclcl}
x_3 \cdot g^{t} = 1 &\implies& x_3 &=& g^{-t} \\
&\implies& uv^2 &=& g^{-t} \\
&\implies& uv &=& v^{-1} \cdot g^{-t} \\
&\implies& uv \cdot g^{t / 2} &=& v^{-1} \cdot g^{-t / 2}.
\end{array}
$$

将等式右边进行平方，我们得到 $(v^{-1} g^{-t / 2})^2 = v^{-2}g^{-t} = u$ 。因此， $u$ 的平方即为 $uv \cdot g^{t / 2}$ ；第一个部分在之前已有计算，第二部分可以通过在 $gtab$ 中查表并计算得到。
