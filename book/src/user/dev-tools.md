## 开发者工具

Rust库 `halo2` 中包含了数个工具，为你设计并开发自己的电路提供帮助。

### Mock prover

`halo2::dev::MockProver` 是一个用来调试电路用的工具。除此之外，它还可以用来在单元测试中快速检验电路正确性。

隐私输入和公开输入的计算过程，是和生成一个真正的证明一样。但是，`MockProver::run` 函数只会生成一个对象，它直接检验电路中每一条约束是否满足。如果电路中某个约束不满足，那这个对象就会返回关于该约束的一个细粒度的错误信息。

*译注*: MockProver 能检验电路是否满足，这比创建证明之后，发现证明证不过快多了。

### 耗时估算器

程序 `cost-model` 可以根据电路设计所采用的不同参数，估算出，验证一个证明的时长，以及对应的证明大小。

```shell
用法: cargo run --example cost-model -- [OPTIONS] k

Positional arguments:
  k                       2^k 总行数的上界.

Optional arguments:
  -h, --help              打印此用法信息.
  -a, --advice R[,R..]    An advice column with the given rotations. May be repeated.
  -i, --instance R[,R..]  An instance column with the given rotations. May be repeated.
  -f, --fixed R[,R..]     A fixed column with the given rotations. May be repeated.
  -g, --gate-degree D     Maximum degree of the custom gates.
  -l, --lookup N,I,T      A lookup over N columns with max input degree I and max table degree T. May be repeated.
  -p, --permutation N     A permutation over N columns. May be repeated.

```


比如，估算有三个 advice column 和一个 fixed column 以及 门的最高次数为 4 的电路: 

```plaintext
cargo run --example cost-model -- -a 0,1 -a 0 -a 0,-1,1 -f 0 -g 4 11
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s
     Running `target/debug/examples/cost-model -a 0,1 -a 0 -a 0,-1,1 -f 0 -g 4 11`
Circuit {
    k: 11,
    max_deg: 4,
    advice_columns: 3,
    lookups: 0,
    permutations: [],
    column_queries: 7,
    point_sets: 3,
    estimator: Estimator,
}
Proof size: 1440 bytes
Verification: at least 81.689ms
```