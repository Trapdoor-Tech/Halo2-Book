# Cryptographic groups

# 密码学的群

In the section [Inverses and groups](fields.md#inverses-and-groups) we introduced the
concept of *groups*. A group has an identity and a group operation. In this section we
will write groups additively, i.e. the identity is $\mathcal{O}$ and the group operation
is $+$.

在章节 [群](fields.md#groups) 中，我们引入了 *群* 的概念。
群有一个单位元和一个群运算。本节，我们将写一个加法群，也就是说，它的单位元是 $\mathcal{O}$，群运算是 $+$。

Some groups can be used as *cryptographic groups*. At the risk of oversimplifying, this
means that the problem of finding a discrete logarithm of a group element $P$ to a given
base $G$, i.e. finding $x$ such that $P = [x] G$, is hard in general.

有些群可以被用于密码学，是为 *密码学的群*。这意味着，通常来讲，找到一个在基 $G$ 下群元素 $P$ 的离散对数
是困难的。找到离散对数，就是找到一个 $x$ 使它满足 $P = [x] G$。


## Pedersen commitment

## Pedersen承诺

The Pedersen commitment [[P99]] is a way to commit to a secret message in a verifiable
way. It uses two random public generators $G, H \in \mathbb{G},$ where $\mathbb{G}$ is a
cryptographic group of order $p$. A random secret $r$ is chosen in $\mathbb{Z}_q$, and the
message to commit to $m$ is from any subset of $\mathbb{Z}_q$. The commitment is 

Pedersen承诺 [[P99]] 是一种以可验证的方式对一个秘密消息进行承诺的方法。
它使用两个公开的、随机的生成元 $G, H \in \mathbb{G},$，$\mathbb{G}$ 是一个 $p$ 阶的密码群。
在 $\mathbb{Z}_q$ 上随机选取一个秘密 $r$，而欲承诺的消息
$m$ 来自于 $\mathbb{Z}_q$ 的任意一个子集。那么该消息的承诺就是： 

$$c = \text{Commit}(m,r)=[m]G + [r]H.$$ 

To open the commitment, the committer reveals $m$ and $r,$ thus allowing anyone to verify
that $c$ is indeed a commitment to $m.$

为了打开这个承诺，承诺者必须曝露 $m$ 和 $r$，以使任何人都能验证 $c$ 确实是 $m$ 的承诺。

[P99]: https://link.springer.com/content/pdf/10.1007%2F3-540-46766-1_9.pdf#page=3

Notice that the Pedersen commitment scheme is homomorphic:

注意，Pedersen承诺机制是同态的：

$$
\begin{aligned}
\text{Commit}(m,r) + \text{Commit}(m',r') &= [m]G + [r]H + [m']G + [r']H \\
&= [m + m']G + [r + r']H \\
&= \text{Commit}(m + m',r + r').
\end{aligned}
$$

Assuming the discrete log assumption holds, Pedersen commitments are also perfectly hiding
and computationally binding:

假设离散对数假设能够保持，那Pedersen承诺就是完美的隐藏(hiding)，计算意义上的绑定(binding)

* **hiding**: the adversary chooses messages $m_0, m_1.$ The committer commits to one of
  these messages $c = \text{Commit}(m_b;r), b \in \{0,1\}.$ Given $c,$ the probability of
  the adversary guessing the correct $b$ is no more than $\frac{1}{2}$.

* **隐藏**: 对方选择两个消息 $m_0, m_1$。承诺者承诺其中一条消息
  $c = \text{Commit}(m_b;r), b \in \{0,1\}$。 给定 $c$，则对手猜出
  $b$ 的正确值得概率不超过 $\frac{1}{2}$。

* **binding**: the adversary cannot pick two different messages $m_0 \neq m_1,$ and
  randomness $r_0, r_1,$ such that $\text{Commit}(m_0,r_0) = \text{Commit}(m_1,r_1).$

* **绑定**: 对方不可能选择两个不同得消息 $m_0 \neq m_1,$ 与随机数
  $r_0, r_1,$ 而能使下列等式成立 $\text{Commit}(m_0,r_0) = \text{Commit}(m_1,r_1).$

### Vector Pedersen commitment

### 向量Pedersen承诺
We can use a variant of the Pedersen commitment scheme to commit to multiple messages at
once, $\mathbf{m} = (m_1, \cdots, m_n)$. This time, we'll have to sample a corresponding
number of random public generators $\mathbf{G} = (G_0, \cdots, G_{n-1}),$ along with a
single random generator $H$ as before (for use in hiding). Then, our commitment scheme is:

我们可以使用Pedersen承诺的一种变形，来一次性承诺多条消息，
$\mathbf{m} = (m_1, \cdots, m_n)$。此时我们需要选取相应数量的随机、公开生成元 $\mathbf{G} = (G_0, \cdots, G_{n-1})$，以及
一个跟以前一样随机的生成元 $H$ （用于隐藏）。那么该承诺机制就是：

$$
\begin{aligned}
\text{Commit}(\mathbf{m}; r) &= \text{Commit}((m_0, \cdots, m_{n-1}); r) \\
&= [r]H + [m_0]G_0 + \cdots + [m_{n-1}]G_{n-1} \\
&= [r]H + \sum_{i= 0}^{n-1} [m_i]G_i.
\end{aligned}
$$

> TODO: is this positionally binding?

## Diffie--Hellman

An example of a protocol that uses cryptographic groups is Diffie--Hellman key agreement
[[DH1976]]. The Diffie--Hellman protocol is a method for two users, Alice and Bob, to
generate a shared private key. It proceeds as follows:

应用密码群的一个例子就是 Diffie--Hellman 密钥协议
[[DH1976]]。Diffie--Hellman 协议是一种为两个使用者，Alice 和 Bob，
产生一个共享密钥的方法。它的过程如下：

1. Alice and Bob publicly agree on two prime numbers, $p$ and $G,$ where $p$ is large and
   $G$ is a primitive root $\pmod p.$ (Note that $g$ is a generator of the group
   $\mathbb{F}_p^\times.$)

1. Alice 和 Bob 公开协商出两个素数，$p$ 和 $G$，其中 $p$ 是一个大素数
   $G$ 是 $\pmod p$ 本原根（注意 $g$ 是群 $\mathbb{F}_p^\times$ 的生成元）。

2. Alice chooses a large random number $a$ as her private key. She computes her public key
   $A = [a]G \pmod p,$ and sends $A$ to Bob.

2. Alice 选取一个大的随机数 $a$ 作为她的私钥。她计算她的公钥
   $A = [a]G \pmod p$，并把其公钥 $A$ 发送给 Bob。

3. Similarly, Bob chooses a large random number $b$ as his private key. He computes his
   public key $B = [b]G \pmod p,$ and sends $B$ to Alice.

3. 类似地，Bob 也选择一个大的随机数 $b$ 作为他的私钥。然后计算他自己的公钥
   $B = [b]G \pmod p$，并将其公钥 $B$ 发送给 Alice。

4. Now both Alice and Bob compute their shared key $K = [ab]G \pmod p,$ which Alice
   computes as

4. 接下来 Alice 和 Bob 都可以计算他们共享密钥 $K = [ab]G \pmod p$，Alice
   将计算
   $$K = [a]B \pmod p = [a]([b]G) \pmod p,$$
   而 Bob 将计算
   $$K = [b]A \pmod p = [b]([a]G) \pmod p.$$

[DH1976]: https://ee.stanford.edu/~hellman/publications/24.pdf

A potential eavesdropper would need to derive $K = [ab]g \pmod p$ knowing only
$g, p, A = [a]G,$ and $B = [b]G$: in other words, they would need to either get the
discrete logarithm $a$ from $A = [a]G$ or $b$ from $B = [b]G,$ which we assume to be
computationally infeasible in $\mathbb{F}_p^\times.$

潜在的窃密者的目的是在已知 $g, p, A = [a]G,$ and $B = [b]G$ 这些公开信息下，求解出密钥 $K = [ab]g \pmod p$。
换句话说，他们不得不从 $A = [a]G$ 求解离散对数 $a$，或者从 $B = [b]G$ 中求解离散对数 $b$，
而在 $\mathbb{F}_p^\times$ 上求解诸如此类的离散对数问题被认为是计算不可能的。

More generally, protocols that use similar ideas to Diffie--Hellman are used throughout
cryptography. One way of instantiating a cryptographic group is as an
[elliptic curve](curves.md). Before we go into detail on elliptic curves, we'll describe
some algorithms that can be used for any group.

更一般地，密码学中很多协议都运用了与 Diffie--Hellman 相类似的想法。
一个密码群的实例就是
[椭圆曲线](curves.md)。在我们进一步探讨椭圆曲线的诸多细节之前，我们将描述一些群上通用算法。

## Multiscalar multiplication

### TODO: Pippenger's algorithm
Reference: https://jbootle.github.io/Misc/pippenger.pdf
