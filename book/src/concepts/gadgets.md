# 工具


当实现一个电路时，我们可以直接使用我们选择的芯片的特性。 通常, 我们会通过 ***工具*** 使用他们. 
这种间接方式是很有用的，因为由于效率和UPA的限制，芯片接口通常依赖于底层的实现细节.
工具接口可以提供更加便捷和稳定的, 从无关细节抽象化的 api

举个例子, 想一个哈希函数, 比如说 SHA-256. 芯片的接口支持 SHA-256 可能是依赖于内部的哈希函数设计, 
例如消息调度和压缩功能之间的分离功能. 一致的工具接口可以提供更加便捷和熟悉的 `更新`/`完成` API, 
也能够在不需要芯片支持的情况下处理hash函数的一部分, 比如说 填充 功能.
这类似于通常通过软件库而不是直接访问cpu上加密原语的[加速](https://software.intel.com/content/www/us/en/develop/articles/intel-sha-extensions.html)
[指令](https://developer.arm.com/documentation/ddi0514/g/introduction/about-the-cortex-a57-processor-cryptography-engine)。

工具能在高层次上为电路程序提供一个模块化,可重用的抽象概念, 类似于在像 
[libsnark](https://github.com/christianlundkvist/libsnark-tutorial),
[bellman](https://electriccoin.co/blog/bellman-zksnarks-in-rust/)
这样的库中使用. 不但需要 抽象 *函数*, 同样也需要抽象 *类型*, 
例如特定大小的椭圆曲线上的点或整数类型.


