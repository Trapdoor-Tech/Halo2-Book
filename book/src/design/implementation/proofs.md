# Halo2 证明

## 透明字节流形式的证明

在 `bellman` 这样的证明系统实现中，存在一个 `Proof` 结构体，用以封装证明数据，由 prover 返回且可以被传递给 verifier 。

但是 `halo2` 的实现中不存在类似的数据结构，理由如下：

- `Proof` 结构体需要包含椭圆曲线点和各标量的向量（以及向量的向量）。这会使证明的序列化/反序列化变得复杂，因为向量的长度实际是由电路的配置决定的。但我们并不希望在证明中将向量的长度进行编码，因为电路在运行时是固定大小的，证明也是。
- 很容易误将一些不属于协议 transcript 的数据放到 `Proof` 结构体里，这对 proving system 的实现会是一个潜在的风险。
- 我们需要同时创建多个 PLONK 证明，而这些证明虽然是给同一个电路的，但它们共享许多不同的子结构体。

因此， `halo2` 将证明数据视作透明的字节流。这些字节流通过 transcript 创建和消费：

- `TranscriptWrite` trait 代表某些我们可以在证明期写入的数据
- `TranscriptRead` trait 代表某些我们可以在验证期读取的数据

重要的是， `TranscriptWrite` 的实现需要满足同时向某个 `std::io::Write` 缓冲写入的能力，这点对于 `TranscriptRead` 和 `std::io::Read` 也是一样。

另外，将证明作为透明字节流对待能让验证过程关注优化反序列化，在进行点的压缩时这一开销是不能忽视的。

## 证明的编码

在曲线 $E(\mathbb{F}_p)$ 上的 Halo2 证明被编码为一些点和标量的字节流：

- 点 $P \in E(\mathbb{F}_p)$) 用于表示多项式的承诺
- 标量 $s \in \mathbb{F}_q$) 用于表示多项式在某个点的取值，以及盲因子等

对于 Pallas 和 Vesta 两条曲线来说，点和标量都用32字节进行编码，因此最后的证明总是32字节的整数倍。

`halo2` 包支持同时证明一个电路的多个 instance ，这样可以共享一些常用的证明组件和协议逻辑。

我们使用如下的电路常量以描述证明的编码：

- $k$ - 电路的大小（ $2^k$ 行）
- $A$ - advice 列的数量
- $F$ - fixed 列的数量
- $I$ - instance 列的数量
- $L$ - lookup argument 的数量
- $P$ - permutation argument 的数量
- $\textsf{Col}_P$ - 在 permutation argument $P$ 中用到的列的数量
- $D$ - 商数多项式的最大度数
- $Q_A$ - advice column 的请求数？？？
- $Q_F$ - fixed column 的请求数？？？
- $Q_I$ - instance column 的请求数？？？
- $M$ - 可以同时证明的电路 instance 数量

因为证明的编码是直接按照 transcript 进行的，我们可以将其按 Halo 2 协议划分成几个部分：

- PLONK 承诺 :
  - $A$ 个点（重复 $M$ 次）
  - $2L$ 个点（重复 $M$ 次）
  - $P$ 个点（重复 $M$ 次）
  - $L$ 个点（重复 $M$ 次）

- Vanishing argument ：
  - $D - 1$ 个点.
  - $Q_I$ 个标量（重复 $M$ 次）
  - $Q_A$ 个标量（重复 $M$ 次）
  - $Q_F$ 个标量
  - $D - 1$ 个标量

- PLONK 求值：
  - $(2 + \textsf{Col}_P) \times P$ 个标量（重复 $M$ 次）
  - $5L$ 个标量（重复 $M$ 次）

- 多点打开 argument ：
  - 1 个点.
  - 在 argument 中的每个点集对应 1 个标量

- 多项式承诺方案：
  - $1 + 2k$ 个点.
  - $2$ 个标量.
