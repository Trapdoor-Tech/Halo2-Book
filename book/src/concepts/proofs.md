# Proof systems

***证明系统*** 的目标是能够证明有趣的数学或密码学语句.

Typically, in a given protocol we will want to prove families of statements that differ
in their ***public inputs***. The prover will also need to show that they know some
***private inputs*** that make the statement hold.

通常，在给定的协议中，我们想要证明不同于它们的公共输入的语句族。
证明者需要展示他们知道一些隐私输入来make the statement hold

To do this we write down a ***relation***, $\mathcal{R}$, that specifies which
combinations of public and private inputs are valid.

为了做到上面所说的, 我们描绘了一个***关系式***, $\mathcal{R}$, 
这个关系式指定哪些公共和隐私输入组合是有效的。


> The terminology above is intended to be aligned with the
> [ZKProof Community Reference](https://docs.zkproof.org/reference#latest-version).

> 上面的术语和 [ZKProof Community Reference](https://docs.zkproof.org/reference#latest-version) 是一致的

To be precise, we should distinguish between the relation $\mathcal{R}$, and its
implementation to be used in a proof system. We call the latter a ***circuit***.

准确地说，我们应该区分关系式 $\mathcal{R}$ 和它在证明系统中使用的实现。我们称后者为电路。


The language that we use to express circuits for a particular proof system is called an
***arithmetization***. Usually, an arithmetization will define circuits in terms of
polynomial constraints on variables over a field.

我们用来表示特定证明系统的电路的语言叫做***arithmetization***
通常，一个算法会在多项式约束条件下定义电路,其中变量在一个域中 。


> The _process_ of expressing a particular relation as a circuit is also sometimes called
> "arithmetization", but we'll avoid that usage.

表达一个特定关系的过程叫做电路, 有时也叫作 "算法", 但是我们避免这种称呼.

To create a proof of a statement, the prover will need to know the private inputs,
and also intermediate values, called ***advice*** values, that are used by the circuit.

为了去创建一个语句的proof, 证明者需要知道 private input, 还有电路使用的中间值，又称为***通知***值。

We assume that we can compute advice values efficiently from the private and public inputs.
The particular advice values will depend on how we write the circuit, not only on the
high-level statement.

我们假设我们能高效的通过公开输入和隐私输入计算通知值.
具体的通知值依赖于于我们如何编写电路，而不仅仅取决于高级语句。


The private inputs and advice values are collectively called a ***witness***.

隐私输入和通知值共同叫做 ***witness***


> Some authors use "witness" as just a synonym for private inputs. But in our usage,
> a witness includes advice, i.e. it includes all values that the prover supplies to
> the circuit.

> 有些作者在写 隐私输入时使用witness, 但是在我们的使用中, witness还包括 通知值, 
> 即它包括证明者提供给电路的所有值。 

For example, suppose that we want to prove knowledge of a preimage $x$ of a
hash function $H$ for a digest $y$:

举个例子

* The private input would be the preimage $x$.

* 隐私输入是 原像 $x$

* The public input would be the digest $y$.

* 公开输入是 $y$

* The relation would be $\{(x, y) : H(x) = y\}$.

* 关系式为 $\{(x, y) : H(x) = y\}$.

* For a particular public input $Y$, the statement would be: $\{(x) : H(x) = Y\}$.

* 对于特定的公共输入 $Y$, 语句是$\{(x) : H(x) = Y\}$.

* The advice would be all of the intermediate values in the circuit implementing the
  hash function. The witness would be $x$ and the advice.

* advice是实现哈希函数的电路中的所有中间值。witness是$x$和advice。


A ***Non-interactive Argument*** allows a ***prover*** to create a ***proof*** for a
given statement and witness. The proof is data that can be used to convince a ***verifier***
that _there exists_ a witness for which the statement holds. The security property that
such proofs cannot falsely convince a verifier is called ***soundness***.

***非交互式argument***允许prover给给定的语句和witness创建一个proof.
证明是一个可以使验证者确信存在witness 并使数据成立的数据.

A ***Non-interactive Argument of Knowledge*** (***NARK***) further convinces the verifier
that the prover _knew_ a witness for which the statement holds. This security property is
called ***knowledge soundness***, and it implies soundness.

***非交互式证明***(***NARK***)进一步使验证者确信证明者知道使语句成立的witness.
这个安全属性称为***知识可靠性***，它暗示了可靠性。


In practice knowledge soundness is more useful for cryptographic protocols than soundness:
if we are interested in whether Alice holds a secret key in some protocol, say, we need
Alice to prove that _she knows_ the key, not just that it exists.

在实践中，知识的可靠性对于密码协议比可靠性更有用 :
如果我们对Alice是否在某个协议中持有密钥感兴趣，我们需要Alice证明她知道密钥，而不仅仅是密钥存在。

Knowledge soundness is formalized by saying that an ***extractor***, which can observe
precisely how the proof is generated, must be able to compute the witness.

知识的可靠性是通过一个***提取器***来形式化的，它可以精确地观察证明是如何生成的，能够计算witness。

> This property is subtle given that proofs can be ***malleable***. That is, depending on the
> proof system it may be possible to take an existing proof (or set of proofs) and, without
> knowing the witness(es), modify it/them to produce a distinct proof of the same or a related
> statement. Higher-level protocols that use malleable proof systems need to take this into
> account.

> 这个性质是微妙的，因为证明是可塑的。 
> 也就是说，根据证明系统，可以采用现有的证明(或者证明集)
> 在不知道witness的情况下，修改witness 生成一个明显相同的证明或者是一个有关的语句.
> 修改证人的陈述，以提供有关陈述的清晰证明。
> 使用可塑证明系统的高级协议需要考虑到这一点.

>
> Even without malleability, proofs can also potentially be ***replayed***. For instance,
> we would not want Alice in our example to be able to present a proof generated by someone
> else, and have that be taken as a demonstration that she knew the key.

> 即使没有可塑性, 证明也很可能***回放***, 举个例子,
> 我们不希望在我们的例子中Alice能够提交别人生成的证明，并将其作为她知道密钥的证明。

If a proof yields no information about the witness (other than that a witness exists and was
known to the prover), then we say that the proof system is ***zero knowledge***.

如果一个证明中没有关于witness的信息(除此之外, witness存在并且prover 知道witness), 
这时我们说证明系统是 ***零知识的***

If a proof system produces short proofs —i.e. of length polylogarithmic in the circuit
size— then we say that it is ***succinct***. A succinct NARK is called a ***SNARK***
(***Succinct Non-Interactive Argument of Knowledge***).

如果证明系统生成了简短的证明, 即 ? 
那么我们说这时 ***简洁的***
一个简介的***NARK*** 叫做 ***snark***(***简洁的非交互式证明***)

> By this definition, a SNARK need not have verification time polylogarithmic in the circuit
> size. Some papers use the term ***efficient*** to describe a SNARK with that property, but
> we'll avoid that term since it's ambiguous for SNARKs that support amortized or recursive
> verification, which we'll get to later.

> 根据这个定义, 一个SNARK 不需要?
> 有些论文使用术语***efficient***来描述具有这种性质的SNARK，
> 但我们将避免使用这个术语，因为它对于支持平摊或递归验证的SNARKs来说是语义是模糊的，我们稍后会讲到


A ***zk-SNARK*** is a zero-knowledge SNARK.

***zk-SNARK*** 叫做零知识证明

