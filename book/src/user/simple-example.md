## 一个简单例子

我们以一个简单电路为例，给大家介绍 `halo2` 所提供的应用编程接口，以及如何使用它们。示例电路有一个公开输入$c$, 可证明知道两个隐私输入$a$和$b$，使得下式成立。 $$ a^2 \cdot b^2 = c$$

## 定义指令

首先，我们需要定义我们的电路所依赖的指令，指令介于高层的工具（gadgets）和底层的电路操作之间。指令既可以细粒度也可以粗粒度，但在实践中，指令的功能应当足够小，这样可以重复使用；但又要足够大，这样可以优化它的实现。设计者应当在这两者之间取得平衡。

在我们的样例电路中，我们将使用如下三种指令：

- 加载一个隐私数到电路中；
- 计算两个数的乘积；
- 将一个数设置为电路的公开输入。

我们还需要一个类型来表示其值为数的变量。

```rust
trait NumericInstructions<F: FieldExt>: Chip<F> {
    /// 用于表示一个数的变量
    type Num;

    /// 将一个数加载到电路中，用作隐私输入
    fn load_private(&self, layouter: impl Layouter<F>, a: Option<F>) -> Result<Self::Num, Error>;

    /// 将一个数加载到电路中，用作固定常数
    fn load_constant(&self, layouter: impl Layouter<F>, constant: F) -> Result<Self::Num, Error>;

    /// 返回 `c = a * b`.
    fn mul(
        &self,
        layouter: impl Layouter<F>,
        a: Self::Num,
        b: Self::Num,
    ) -> Result<Self::Num, Error>;

    /// 将一个数置为电路的公开输入
    fn expose_public(
        &self,
        layouter: impl Layouter<F>,
        num: Self::Num,
        row: usize,
    ) -> Result<(), Error>;
}
```

### 定义芯片的实现

在示例电路中，我还将构造一个芯片，其功能包含上述的有限域上的数值指令。

```rust
/// 这块芯片将实现我们的指令集！芯片存储它们自己的配置，
///  必要情况下也要包含 type markers
struct FieldChip<F: FieldExt> {
    config: FieldConfig,
    _marker: PhantomData<F>,
}
```

每一个"芯片"类型都要实现`Chip`接口。`Chip`接口定义了`Layouter`在做电路综合时可能需要的关于电路的某些性质，以及若将该芯片加载到电路所需要设置的任何初始状态。

```rust
impl<F: FieldExt> Chip<F> for FieldChip<F> {
    type Config = FieldConfig;
    type Loaded = ();

    fn config(&self) -> &Self::Config {
        &self.config
    }

    fn loaded(&self) -> &Self::Loaded {
        &()
    }
}
```

### 配置芯片

需要为芯片配置好实现我们想要的功能所需要的那些列、置换、门。

```rust
/// 芯片的状态被存储在一个 config 结构体中，它是在配置过程中由芯片生成，
/// 并且存储在芯片内部。
#[derive(Clone, Debug)]
struct FieldConfig {
    /// 对于这块芯片，我们将用到两个 advice 列来实现我们的指令集。
    /// 它们也是我们与电路的其他部分通信所需要用到列。
    advice: [Column<Advice>; 2],

    /// 这是公开输入（instance）列
    instance: Column<Instance>,

    // 我们需要一个 selector 来激活乘法门，从而在用不到`NumericInstructions::mul`指令的
    //  cells 上不设置任何约束。这非常重要，尤其在构建更大型的电路的情况下，列会被多条指令集用到
    s_mul: Selector,

    /// 用来加载常数的 fixed 列
    constant: Column<Fixed>,
}

impl<F: FieldExt> FieldChip<F> {
    fn construct(config: <Self as Chip<F>>::Config) -> Self {
        Self {
            config,
            _marker: PhantomData,
        }
    }

    fn configure(
        meta: &mut ConstraintSystem<F>,
        advice: [Column<Advice>; 2],
        instance: Column<Instance>,
        constant: Column<Fixed>,
    ) -> <Self as Chip<F>>::Config {
        meta.enable_equality(instance.into());
        meta.enable_constant(constant);
        for column in &advice {
            meta.enable_equality((*column).into());
        }
        let s_mul = meta.selector();

        // 定义我们的乘法门
        meta.create_gate("mul", |meta| {
            // 我们需要3个 advice cells 和 1个 selector cell 来实现乘法
            // 我们把他们按下表来排列：
            //
            // | a0  | a1  | s_mul |
            // |-----|-----|-------|
            // | lhs | rhs | s_mul |
            // | out |     |       |
            //
            // 门可以用任一相对偏移，但每一个不同的偏移都会对证明增加开销。
            // 最常见的偏移值是 0 (当前行), 1(下一行), -1(上一行)。
            // 针对这三种情况，有特定的构造函数来构造`Rotation` 结构。
            let lhs = meta.query_advice(advice[0], Rotation::cur());
            let rhs = meta.query_advice(advice[1], Rotation::cur());
            let out = meta.query_advice(advice[0], Rotation::next());
            let s_mul = meta.query_selector(s_mul);

            // 最终，我们将约束门的多项式表达式返回。
            // 对于我们的乘法门，我们仅需要一个多项式约束。
            //
            // `create_gate` 函数返回的多项式表达式，在证明系统中一定等于0。
            // 我们的表达式有以下性质：
            // - 当 s_mul = 0 时，lhs, rhs, out 可以是任意值。
            // - 当 s_mul != 0 时，lhs, rhs, out 将满足 lhs * rhs = out 这条约束。 
            vec![s_mul * (lhs * rhs - out)]
        });

        FieldConfig {
            advice,
            instance,
            s_mul,
            constant,
        }
    }
}
```

### 实现芯片功能
```rust
/// 用于表示数的变量
#[derive(Clone)]
struct Number<F: FieldExt> {
    cell: Cell,
    value: Option<F>,
}

impl<F: FieldExt> NumericInstructions<F> for FieldChip<F> {
    type Num = Number<F>;

    fn load_private(
        &self,
        mut layouter: impl Layouter<F>,
        value: Option<F>,
    ) -> Result<Self::Num, Error> {
        let config = self.config();

        let mut num = None;
        layouter.assign_region(
            || "load private",
            |mut region| {
                let cell = region.assign_advice(
                    || "private input",
                    config.advice[0],
                    0,
                    || value.ok_or(Error::SynthesisError),
                )?;
                num = Some(Number { cell, value });
                Ok(())
            },
        )?;
        Ok(num.unwrap())
    }

    fn load_constant(
        &self,
        mut layouter: impl Layouter<F>,
        constant: F,
    ) -> Result<Self::Num, Error> {
        let config = self.config();

        let mut num = None;
        layouter.assign_region(
            || "load constant",
            |mut region| {
                let cell = region.assign_advice_from_constant(
                    || "constant value",
                    config.advice[0],
                    0,
                    constant,
                )?;
                num = Some(Number {
                    cell,
                    value: Some(constant),
                });
                Ok(())
            },
        )?;
        Ok(num.unwrap())
    }

    fn mul(
        &self,
        mut layouter: impl Layouter<F>,
        a: Self::Num,
        b: Self::Num,
    ) -> Result<Self::Num, Error> {
        let config = self.config();

        let mut out = None;
        layouter.assign_region(
            || "mul",
            |mut region: Region<'_, F>| {
                // 在这个 region 中，我们只想用一个乘法门，所以我们在 region 偏移 0 处，
                //  激活它；这意味着它将对 0偏移 和 1偏移处的两个 cells 进行约束。
                config.s_mul.enable(&mut region, 0)?;

                // 给我们的输入有可能在电路的任一位置，但在当前 region 中，我们仅可以用
                // 相对偏移。所以，我们在 region 内分配新的 cells 并限定他们的值与输入 cells 的值相等。
                let lhs = region.assign_advice(
                    || "lhs",
                    config.advice[0],
                    0,
                    || a.value.ok_or(Error::SynthesisError),
                )?;
                let rhs = region.assign_advice(
                    || "rhs",
                    config.advice[1],
                    0,
                    || b.value.ok_or(Error::SynthesisError),
                )?;
                region.constrain_equal(a.cell, lhs)?;
                region.constrain_equal(b.cell, rhs)?;

                // 现在我们可以把乘积放到输出的位置了。
                let value = a.value.and_then(|a| b.value.map(|b| a * b));
                let cell = region.assign_advice(
                    || "lhs * rhs",
                    config.advice[0],
                    1,
                    || value.ok_or(Error::SynthesisError),
                )?;

                // 最后，我们返回一个用来表示输出的变量，它会被用在电路的另一个部分。
                out = Some(Number { cell, value });
                Ok(())
            },
        )?;

        Ok(out.unwrap())
    }

    fn expose_public(
        &self,
        mut layouter: impl Layouter<F>,
        num: Self::Num,
        row: usize,
    ) -> Result<(), Error> {
        let config = self.config();

        layouter.constrain_instance(num.cell, config.instance, row)
    }
}
```

### 构造电路
既然我们已经有了所需要的指令，以及一块实现了这些指令的芯片，我们终于可以构造示例电路啦!
```rust
/// 完整的电路实现
///
/// 在这个结构体中，我们保存隐私输入变量。我们使用 `Option<F>` 类型是因为，
/// 在生成密钥阶段，他们不需要有任何的值。在证明阶段中，如果它们任一为 `None` 的话，
/// 我们将得到一个错误。
#[derive(Default)]
struct MyCircuit<F: FieldExt> {
    constant: F,
    a: Option<F>,
    b: Option<F>,
}

impl<F: FieldExt> Circuit<F> for MyCircuit<F> {
    // 因为我们在任一地方值用了一个芯片，所以我们可以重用它的配置。
    type Config = FieldConfig;
    type FloorPlanner = SimpleFloorPlanner;

    fn without_witnesses(&self) -> Self {
        Self::default()
    }

    fn configure(meta: &mut ConstraintSystem<F>) -> Self::Config {
        // 我们创建两个 advice 列，作为 FieldChip 的输入。
        let advice = [meta.advice_column(), meta.advice_column()];

        // 我们还需要一个 instance 列来存储公开输入。
        let instance = meta.instance_column();

        // 创建一个 fixed 列来加载常数
        let constant = meta.fixed_column();

        FieldChip::configure(meta, advice, instance, constant)
    }

    fn synthesize(
        &self,
        config: Self::Config,
        mut layouter: impl Layouter<F>,
    ) -> Result<(), Error> {
        let field_chip = FieldChip::<F>::construct(config);

        // 将我们的隐私值加载到电路中。
        let a = field_chip.load_private(layouter.namespace(|| "load a"), self.a)?;
        let b = field_chip.load_private(layouter.namespace(|| "load b"), self.b)?;

        // 将常数因子加载到电路中
        let constant =
            field_chip.load_constant(layouter.namespace(|| "load constant"), self.constant)?;

        // 我们仅有乘法可用，因此我们按以下方法实现电路：
        //     asq  = a*a
        //     bsq  = b*b
        //     absq = asq*bsq
        //     c    = constant*asq*bsq
        //
        // 但是，按下面的方法实现，更加高效: 
        //     ab   = a*b
        //     absq = ab^2
        //     c    = constant*absq
        let ab = field_chip.mul(layouter.namespace(|| "a * b"), a, b)?;
        let absq = field_chip.mul(layouter.namespace(|| "ab * ab"), ab.clone(), ab)?;
        let c = field_chip.mul(layouter.namespace(|| "constant * absq"), constant, absq)?;

        // 将结果作为电路的公开输入进行公开
        field_chip.expose_public(layouter.namespace(|| "expose c"), c, 0)
    }
}
```

### 测试电路的功能

可以用 `halo2::dev::MockProver` 对象来测试一个电路是否正常工作。构造电路的一组隐私输入和公开输入，这组输入可直接用来计算合法证明，但我们把这组输入传入到`MockProver::run`函数中之后，就能得到一个可用于检验电路中每一条约束是否满足的对象。而且电路验证不过，这个对象还能输出那条不满足的约束。

```rust
    // 我们电路的行数不能超过 2^k. 因为我们的示例电路很小，我们选择一个较小的值
    let k = 4;

    // 准备好电路的隐私输入和公开输入
    let constant = Fp::from(7);
    let a = Fp::from(2);
    let b = Fp::from(3);
    let c = constant * a.square() * b.square();

    // 用隐私输入来实例化电路
    let circuit = MyCircuit {
        constant,
        a: Some(a),
        b: Some(b),
    };

    // 将公开输入进行排列。乘法的结果被我们放置在 instance 列的第0行，
    // 所以我们把它放在公开输入的对应位置。
    let mut public_inputs = vec![c];

    // 给定正确的公开输入，我们的电路能验证通过
    let prover = MockProver::run(k, &circuit, vec![public_inputs.clone()]).unwrap();
    assert_eq!(prover.verify(), Ok(()));

    // 如果我们尝试用其他的公开输入，证明将失败！
    public_inputs[0] += Fp::one();
    let prover = MockProver::run(k, &circuit, vec![public_inputs]).unwrap();
    assert!(prover.verify().is_err());
```

### 完整例子
 [这里](https://github.com/zcash/halo2/tree/main/examples/simple-example.rs) 有示例的所有源代码。
