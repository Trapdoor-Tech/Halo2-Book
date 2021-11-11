# 证明系统

Halo2的证明系统可以分解成五个部分：

1. 对电路中的主要模块多项式承诺：
   - 单元格的赋值
   - 查找表证明中的置换和乘积
   - 相等约束的置换
2. 电路中和零相关的部分，构造消退证明：
   - 标准的和自定义门
   - 查找表规则
   - 相等约束规则
3. 在所有需要的点上，获取上述多项式的取值：
   - 所有自定义门上的偏移信息
   - 消退证明的各个部分
4. 构建多点打开证明，检查所有的多项式取值和相应的承诺是否一致
5. 采用内积证明为多点打开证明生成证明

这些部分依次在本章节讲解。

## 举个例子

为了辅助我们的解释，我们时不时地用如下的约束系统：

- 四个advice列 $a, b, c, d$
- 一个fixed列 $f$
- 三个自定义门：
  - $a \cdot b \cdot c_{-1} - d = 0$
  - $f_{-1} \cdot c = 0$
  - $f \cdot d \cdot a = 0$

## 太长不读

Halo2协议的简洁描述整理在如下的表格中。这部分会被Halo2的论文和安全性证明替代，目前作为后面子章节的总结：

| 证明者                                                       |         | 验证者                             |
| ------------------------------------------------------------ | ------- | ---------------------------------- |
|                                                              | $\larr$ | $t(X) = (X^n - 1)$                 |
|                                                              | $\larr$ | $F = [F_0, F_1, \dots, F_{m - 1}]$ |
| $\mathbf{A} = [A_0, A_1, \dots, A_{m - 1}]$                  | $\rarr$ |                                    |
|                                                              | $\larr$ | $\theta$                           |
| $\mathbf{L} = [(A'_0, S'_0), \dots, (A'_{m - 1}, S'_{m - 1})]$ | $\rarr$ |                                    |
|                                                              | $\larr$ | $\beta, \gamma$                    |
| $\mathbf{Z_P} = [Z_{P,0}, Z_{P,1}, \ldots]$                  | $\rarr$ |                                    |
| $\mathbf{Z_L} = [Z_{L,0}, Z_{L,1}, \ldots]$                  | $\rarr$ |                                    |
|                                                              | $\larr$ | $y$                                |
| $h(X) = \frac{\text{gate}_0(X) + \dots + y^i \cdot \text{gate}_i(X)}{t(X)}$ |         |                                    |
| $h(X) = h_0(X) + \dots + X^{n(d-1)} h_{d-1}(X)$              |         |                                    |
| $\mathbf{H} = [H_0, H_1, \dots, H_{d-1}]$                    | $\rarr$ |                                    |
|                                                              | $\larr$ | $x$                                |
| $evals = [A_0(x), \dots, H_{d - 1}(x)]$                      | $\rarr$ |                                    |
|                                                              |         | 检查 $h(x)$                        |
|                                                              | $\larr$ | $x_1, x_2$                         |
| 构造 $h'(X)$ 多点打开多项式                                  |         |                                    |
| $U = \text{Commit}(h'(X))$                                   | $\rarr$ |                                    |
|                                                              | $\larr$ | $x_3$                              |
| $\mathbf{q}_\text{evals} = [Q_0(x_3), Q_1(x_3), \dots]$      | $\rarr$ |                                    |
| $u_\text{eval} = U(x_3)$                                     | $\rarr$ |                                    |
|                                                              | $\larr$ | $x_4$                              |

Then the prover and verifier:

- Construct $\text{finalPoly}(X)$ as a linear combination of $\mathbf{Q}$ and $U$ using
  powers of $x_4$;
- Construct $\text{finalPolyEval}$ as the equivalent linear combination of
  $\mathbf{q}_\text{evals}$ and $u_\text{eval}$; and
- Perform $\text{InnerProduct}(\text{finalPoly}(X), x_3, \text{finalPolyEval}).$

> TODO: Write up protocol components that provide zero-knowledge.
