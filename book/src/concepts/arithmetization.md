# PLONKish 算法

halo2 使用的运算方法 来自于 [PLONK](https://eprint.iacr.org/2019/953), 
或者更准确地说，它的扩展UltraPLONK支持自定义门和查找arguments.
我们称之为 [***PLONKish***](https://twitter.com/feministPLT/status/1413815927704014850).

***PLONKish电路***是用矩形矩阵值定义的. 
我们称呼矩阵的***行***，***列***，***单元格*** 用常规的称呼。

PLONKish电路依赖***配置*** 

* 一个有限域$\mathbb{F}$，其中单元格的值(对于给定的statement和witness)将是$\mathbb{F}$的元素.

* The number of columns in the matrix, and a specification of each column as being
  ***fixed***, ***advice***, or ***instance***. Fixed columns are fixed by the circuit;
  advice columns correspond to witness values; and instance columns are normally used for
  public inputs (technically, they can be used for any elements shared between the prover
  and verifier).

* 矩阵中列的数量, 以及每个列的参数是***固定***、***通知***还是***实例***.
  固定列是固定在电路中的. 通知列和witness 值一致. 实例列通常用于公开输入(
  确切来说, 它们可以用于prover和verifier之间共享的任何元素).


* A subset of the columns that can participate in equality constraints.

* 可以参与相等约束的列的子集。

* A ***polynomial degree bound***.

* 多项式次数约束

* A sequence of ***polynomial constraints***. These are multivariate polynomials over
  $\mathbb{F}$ that must evaluate to zero *for each row*. The variables in a polynomial
  constraint may refer to a cell in a given column of the current row, or a given column of
  another row relative to this one (with wrap-around, i.e. taken modulo $n$). The maximum
  degree of each polynomial is given by the polynomial degree bound.

* 一个***多项式约束***的序列. 这是在 $\mathbb{F}$ 上的*每行*必须为0的多元多项式.
  多项式约束中的变量可以指当前行的给定列中的单元格,也可以指与这行(即取模n)相关的另一行的给定列
  每个多项式的最高次由这个多项式次数给出.

* A sequence of ***lookup arguments*** defined over tuples of ***input expressions***
  (which are multivariate polynomials as above) and ***table columns***.

* 一个***查找 arguments***在***输入表达式***(多元多项式)的元组和表列上定义

A PLONKish circuit also defines:

PLONKish电路还定义:

* The number of rows $n$ in the matrix. $n$ must correspond to the size of a multiplicative
  subgroup of $\mathbb{F}^\times$; typically a power of two.

* 矩阵的行数为 $n$. $n$ 必须对应于 $\mathbb{F}^\times$ 的可乘子群的大小;通常是 2的幂次方.

* A sequence of ***equality constraints***, which specify that two given cells must have equal
  values.

* 一个由***相等约束***组成的数组，数组中指定两个给定单元格必须具有相等的值

* The values of the fixed columns at each row.

* 每行中固定列的值

From a circuit description we can generate a ***proving key*** and a ***verification key***,
which are needed for the operations of proving and verification for that circuit.

从电路描述中我们可以生成 电路的证明和验证操作所需要的 一个 ***证明key*** 和 一个 ***验证key***

> Note that we specify the ordering of columns, polynomial constraints, lookup arguments, and
> equality constraints, even though these do not affect the meaning of the circuit. This makes
> it easier to define the generation of proving and verification keys as a deterministic
> process.

> 请注意，我们指定了列的顺序、多项式约束、查找参数和等式约束，即使这些并不影响电路的含义。
> 这使得将证明和验证密钥的生成定义为确定性过程变得更加容易。


Typically, a configuration will define polynomial constraints that are switched off and on by
***selectors*** defined in fixed columns. For example, a constraint $q_i \cdot p(...) = 0$ can
be switched off for a particular row $i$ by setting $q_i = 0$. In this case we sometimes refer
to a set of constraints controlled by a set of selector columns that are designed to be used
together, as a ***gate***. Typically there will be a ***standard gate*** that supports generic
operations like field multiplication and division, and possibly also ***custom gates*** that
support more specialized operations.

通常，配置将定义多项式约束，这些约束由定义在固定列中的***选择器***关闭和开启。
例如，约束 $q_i \cdot p(...) = 0$ 可以通过在第i行设置$q_i = 0$来关闭。
在这种情况下，我们有时引用一组由一组选择器列控制的约束，这些选择器列被设计为一起使用，就像一个***gate***。
通常会有一个标准门， 它支持像字段乘法和除法这样的通用操作，也可能有一个支持更专用操作的自定义门.
