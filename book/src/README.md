# halo2 [![Crates.io](https://img.shields.io/crates/v/halo2.svg)](https://crates.io/crates/halo2) #

**重要提示**: 这个库正在积极开发中，不应该在生产软件中使用.

## [文档](https://docs.rs/halo2)

## 最低支持的 Rust 版本

需要 Rust **1.51** 或者更高.

最低支持的 rust 版本 可以在 未来更改, 但是这会带来小版本的改变

## 并行控制方案

`halo2` 当前为并行计算使用 [rayon](https://github.com/rayon-rs/rayon) .
设置 线程数量数量时使用 `RAYON_NUM_THREADS` 环境变量

## 许可证

Copyright 2020-2021 The Electric Coin Company.

你可以在 Bootstrap 开源许可证 1.0 版下使用这个包，
或者您可以选择任意更高版本 . 看这个文件  [`COPYING`](COPYING) 去了解更多详细信息,
 看这个文件 [`LICENSE-BOSL`](LICENSE-BOSL) 了解  Bootstrap 开源许可证 1.0 版条款.

BOSL的目的是允许对包进行商业改进，同时确保所有改进都是开源的 . 看
[这](https://electriccoin.co/blog/introducing-tgppl-a-radically-new-type-of-open-source-license/)
去了解 BOSL 为什么存在.
