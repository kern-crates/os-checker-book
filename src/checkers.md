# Checkers

正如该工具的名字 **os-checker**，它分为两部分， os 相关和 checker 相关。

其中 os 表示操作系统相关，主要针对操作系统组件库而设计。这意味，考虑的使用场景将比惯常的 Rust 
项目更奇特，尤其在编译条件上，因为最底层的操作系统库必须严格地面向特定的机器。

而另一部分 checker，可能不仅仅局限于纯粹的程序检查。在 Rust 成熟的库中，可以观察到它们倾向于设置额外的
**检查步骤**，可能直接检查代码，也可能间接检查代码，或者甚至不检查代码。

因此 **checker** 一词在该工具中，具有非常宽泛的定义，它表示执行某些检查，帮助使用者从各种角度改进代码。

具体来说，**checkers** 包含以下几类：


| checker 类别 |    子类别    | 工具           | 重要程度 | 亮点/论文                  | issue  | 说明                                    |
|:------------:|:------------:|----------------|----------|----------------------------|--------|-----------------------------------------|
| 程序分析工具 |              |                |          |                            |        |                                         |
|      👉      | 静态检查工具 |                |          |                            |        |                                         |
|              |              | [clippy]       | ⭐⭐⭐   | 社区实践标准               |        | 捕获常见的编码错误，并使代码更加地道    |
|              |              | [mirai]        | ⭐⭐     | [论文][mirai-paper]        | [#36]  | 检查 panic；支持属性标注                |
|              |              | [rapx]         | ⭐⭐     | [RAPx book][rapx-book]     | [#138] | 检查 UAF 和内存泄露                     |
|              |              | [lockbud]      | ⭐⭐     | [论文][lockbud-paper]      | [#34]  | 检查 deadlock 等并发问题                |
|              |              | [atomvchecker] | ⭐⭐     | [论文][atomvchecker-paper] |        | 检查 memory ordering misuse             |
|              |              | [rudra]        | ⭐⭐     | [论文][rudra-paper]        | [#161] | 检查 panic safety 和 Send/Sync Variance |
|      👉      | 动态检查工具 |                |          |                            |        |                                         |
|              |              | 测试           | ⭐⭐⭐   | 工程实践标准               |        | `cargo test` 或者自定义测试?            |
|              |              | [miri]         | ⭐⭐⭐   | 社区实践标准               | [#12]  | 最高质量的 UB 检查结果                  |
| 辅助检查工具 |              |                |          |                            |        |                                         |
|      👉      |  格式化检查  | [fmt]          | ⭐⭐⭐   | 社区实践标准               | [#4]   | 检查未格式化的代码                      |
|      👉      |  供应链审查  |                |          |                            |        |                                         |
|              |              | [audit]        | ⭐⭐⭐   | 社区实践标准               | [#42]  | 检查是否存在已报告安全漏洞的依赖版本    |
|              |              | [udeps]        | ⭐⭐     |                            |        | 尽可能消除无用的依赖                    |
|              |              | [outdated]     | ⭐       |                            | [#131] | 尽可能使用最新的依赖                    |
|      👉      |   代码统计   | [geiger]       | ⭐       |                            | [#154] | 尽可能警惕不安全代码                    |
|      👉      | 版本语义检查 | [semver]       | ⭐⭐     | 社区实践标准               |        | 一个严肃的发版应该遵循语义化版本控制    |

注意：`?` 表示尚未实施，但计划一定会集成到 os-checker。

[fmt]: https://github.com/rust-lang/rustfmt
[#4]: https://github.com/os-checker/os-checker/issues/4

[audit]: https://github.com/RustSec/rustsec/tree/main/cargo-audit
[#42]: https://github.com/os-checker/os-checker/issues/42

[outdated]: https://github.com/kbknapp/cargo-outdated
[#131]: https://github.com/os-checker/os-checker/issues/131

[udeps]: https://github.com/est31/cargo-udeps

[geiger]: https://github.com/geiger-rs/cargo-geiger
[#154]: https://github.com/os-checker/os-checker/issues/154

[clippy]: https://github.com/rust-lang/rust-clippy

[mirai]: https://github.com/endorlabs/MIRAI
[mirai-paper]: https://alastairreid.github.io/papers/hatra2020.pdf
[#36]: https://github.com/os-checker/os-checker/issues/36

[lockbud]: https://github.com/BurtonQin/lockbud
[lockbud-paper]: https://burtonqin.github.io/publication/2020-03-11-rustdetector-tse-8
[#34]: https://github.com/os-checker/os-checker/issues/34

[atomvchecker]: https://github.com/AtomVChecker/rust-atomic-study
[atomvchecker-paper]: https://ieeexplore.ieee.org/document/10771495

[rapx]: https://github.com/Artisan-Lab/RAP
[rapx-book]: https://artisan-lab.github.io/RAP-Book
[#138]: https://github.com/os-checker/os-checker/issues/138

[rudra]: https://github.com/sslab-gatech/Rudra
[rudra-paper]: https://github.com/sslab-gatech/Rudra/blob/master/rudra-sosp21.pdf
[#161]: https://github.com/os-checker/os-checker/issues/161

[miri]: https://github.com/rust-lang/miri
[#12]: https://github.com/os-checker/os-checker/issues/12

[semver]: https://github.com/obi1kenobi/cargo-semver-checks
[checker-list]: https://burtonqin.github.io/posts/2024/07/rustcheckers/

此外，os-checker 还应包括基础信息：
* Cargo.toml：Package 维度；由许多工具读取和使用，应该正确维护
* Github API：仓库维度
