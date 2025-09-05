<style>
.contributions {
  color: red;
  font-weight: bold;
}
</style>

# 静态分析工具的组件化

> 静态分析工具：这里指通过与 Rust 编译器进行交互来分析代码，并检查不良模式。

橙色字体的组件是已经存在的，黑色字体的组件是尚未进展的。

![](https://github.com/user-attachments/assets/95ecc275-b52a-4832-a53b-3280efa9bf7a)

## 目标

实现统一的静态分析框架，**让新的检查工具编写者专注于编写算法库**，而不是重复编写琐碎的工具逻辑，并将分析结果轻松地集成到 os-checker。

## 背景

1. Rust 静态分析工具大多采用 [MIR] 作为基础的分析对象，通过一系列算法，实现某些问题模式的检测来识别代码 bug，帮助编写无 UB 的高质量 Rust 代码。
2. 获取 MIR 的主要途径是直接与 Rust 编译器的夜间接口交互，这会导致一些麻烦：
    * 检查工具的编写者需要从复杂的、无良好文档记录的接口中找到程序分析所需的部分，有时数据来自多个地方，需要自行查找多处；
    * 夜间接口是不稳定的，需要固定一个版本号，这意味着升级版本的障碍会随着时间而增加，API 更改将导致重新查找接口；
    * 检查工具的编写者需要同时了解 Rust 语言、Rust 编译器和 Cargo，这需要额外的时间，并且在短期内无法做到；
3. 写一个实用和好用的检查工具，除了程序分析知识和上述 Rust 相关知识，还需要经验的积累和反复的迭代 —— 而检查工具编写者大多以发论文为目的，
   它们在工程能力上缺乏经验，并且只关注眼前的问题，不了解应用到实际项目上的需求。
    * 比如很少有静态检查工具提供 JSON 格式输出，它们大多只是日志打印
    * 很少正确处理 Cargo 项目编译，尤其是条件编译
    * 甚至其代码库，都不做到应用 fmt 和 clippy

[MIR]: https://blog.rust-lang.org/2016/04/19/MIR.html

## 组件构成

### 稳定的数据源

| 项目        | 核心功能                                       | 缺点                                | 使用者           |
|-------------|------------------------------------------------|-------------------------------------|------------------|
| [charon]    | 提供精简 MIR 的通用 JSON 数据，支持 CFG 和 AST | 数据不够完整                        | [Aeneas]、[Kani] |
| [StableMir] | 提供稳定的、版本语义控制的编译器 API 接口      | 需回退到内部 API 以获取不支持的数据 | [Kani]           |

[Aeneas]: https://github.com/AeneasVerif/aeneas
[Kani]: https://github.com/model-checking/kani

#### Charon

[charon] 向程序分析提供精简了 MIR 和语义的、与编程语言无关的、稳定的 JSON 数据格式，支持 CFG 和 AST 两种数据结构。 

它主要作者是 Nadrieril —— 法国 Inria （信息与自动化研究所）工程师、Rust 项目成员、资深 Rust 编译器开发者。

Charon 主要是为了给 [Aeneas] 形式化验证工具和 Eurycice （Rust 到 C 的翻译）使用。但 Kani 也采用了 Charon。

它通过编译 Rust 代码，抽取程序数据，并在 Aeneas 中验证放到 Ocaml 等验证后端或者 Eurycice 分析该数据以提取 Rust 语义。

Charon 一次提供所有必要的信息，编写了 rustc driver，从而工具编写者无需写这部分，也不必了解 Rust 
编译器内部才关心的细枝末节，从而专注于编写工具本身。因此 Charon 不仅将 **数据逻辑与验证逻辑进行分离**，而且适合跨语言分析。

《[Charon: An Analysis Framework for Rust][charon-thesis]》论文中实现了 [charon-rudra]，成功利用 Charon 重写 Rudra 
的一个算法。我在 3 月的时候把剩下两个 Rudra 算法 [补齐了](./checker/charon.md)，并总结了一下移植的步骤和经验。

<span class="contributions">贡献点：</span>
* 移植一些现有的检查工具的算法，比如 lockbud、rap；
* 直接利用 charon 编写新的工具：参考上述论文有实现一个 taint-checker ([charon-artifacts])

[charon]: https://github.com/AeneasVerif/charon
[charon-rudra]: https://github.com/AeneasVerif/charon-rudra
[charon-thesis]: https://link.springer.com/chapter/10.1007/978-3-031-98685-7_18
[charon-artifacts]: https://zenodo.org/records/13983686

#### StableMir

[StableMir] 是 Rust 官方的一个子项目和持续推进的 [项目目标][project-goal-StableMir]。

它提供 [rustc_public] 这个稳定且版本语义化的编译器 API 接口库，具有类似 MIR 的中间表示和非常易用的编译器入口功能。

当某些程序信息尚未支持，那么可通过 [internal][rustc_internal] 接口将 DefId 与内部的 DefId 相互转换，从而访问更多信息。

其维护者主要是 Celina —— [Kani] 作者、AWS 工程师、Rust 项目成员。Kani 是 StableMir 的主要使用者。

我在 [distributed-verification] 和 [safety-tool] 也使用了 StableMir，也可以参考它们。

<span class="contributions">贡献点：</span>
* StableMir 适合作为 Rust 程序分析的入口，因此可以制作使用教程：我预计 10 月会和 Rust 编译器训练营合作一节课，来讲解这部分

[StableMir]: https://github.com/rust-lang/project-stable-mir
[project-goal-StableMir]: https://rust-lang.github.io/rust-project-goals/2025h1/stable-mir.html
[rustc_public]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_public/
[rustc_internal]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_public/rustc_internal/index.html

[distributed-verification]: https://github.com/os-checker/distributed-verification
[safety-tool]: https://github.com/Artisan-Lab/tag-std

### 算法库

| checker   | 功能                                                             | 重写                                                       |
|-----------|------------------------------------------------------------------|------------------------------------------------------------|
| [Rudra]   | 查找 panic 导致的内存安全问题；缺乏 Send/Sync 约束导致的并发问题 | 已完成 [charon-rudra]，但需支持完整的[测试套件][rudra-poc] |
| [RAPx]    | 分析指针别名、内存释放和泄露、API 依赖                           | 有意切换到 StableMir                                       |
| [Lockbud] | 检测死锁等并发问题                                               | 正在切换到 StableMir                                       |

[Rudra]: https://github.com/sslab-gatech/Rudra
[rudra-poc]: https://github.com/sslab-gatech/Rudra-PoC
[RAPx]: https://github.com/Artisan-Lab/RAPx
[Lockbud]: https://github.com/BurtonQin/lockbud

### 工具类

#### 参数解析

基于 Rust 编译器的静态分析工具应当支持与 `cargo build` 相同的参数，但也允许自己的命令行参数。

当前分析工具对此没有统一的解析方式，并且不会主动考虑支持 `cargo build` 的所有参数。

工具的使用者从而面临以下问题：
* 了解每个工具各异的参数规则，增加使用的认知成本
* 无法确保这些工具对 `cargo build` 参数具有一流的支持，尤其在 `--taget` 和 `--feature` 参数上

此外，参数解析还涉及设置 cargo 的 rustc_wrapper：通常一个静态分析程序是纯粹的 rustc 包装器（或者称为编译器驱动程序），但还需要一个
cargo 子命令程序以利用 cargo 构建系统和你的程序中额外的一些编译器参数传递。整个过程是样板代码，但工具的编写者需要重复了解和实现它们。

<span class="contributions">贡献点：</span>
* 参考我在 [distributed-verification]、[safety-tool] 工具中的包装逻辑，提供一个库来封装 cargo 子命令的参数解析并传递参数给 rustc 包装器

#### 测试框架

静态分析工具实质上是一种代码模式查找和匹配的工具，并且采用相同的流程来执行分析。因此统一的测试框架是必要的。

该测试框架应该包含：
* TP / TN / FP / FN 断言
* 诊断输出类别断言
* 诊断 UI 快照测试
* 编译器动态库设置
* 检查命令生成
  * 针对 rs 文件
  * 针对 Cargo 项目
  * 针对真实的 Rust 仓库（固定到 commit）
* 性能基准测试

<span class="contributions">贡献点：</span>
* 参考 [rudra-poc]、[RAPx]、[distributed-verification]、[safety-tool] 等测试代码，提供一个包含上述功能的测试框架库，并应用于现有的检查工具

#### SARIF 输出

[SARIF] (Static Analysis Results Interchange Format) 是一种基于 JSON 文件的标准化诊断输出格式，被 IDE、Github 等多种工具支持。

静态检查工具几乎没有考虑到提供 SARIF 或者只以日志打印的方式不提供格式化输出。

统一的格式化输出既可以将诊断结果在不同的工具中复用，还可以更好地存储和管理。

<span class="contributions">贡献点：</span>
* 对现有检查工具，比如 [Rudra]、[RAPx]、[Lockbud] 等工具的输出进行改造，以支持 SARIF 格式输出
* 对 os-checker 的诊断解析进行改进，以支持读取 SARIF 格式

[SARIF]: https://github.com/os-checker/os-checker/discussions/11

#### IDE 集成

目前有些形式化验证工具（比如 Verus）具有 IDE 集成；Mirai 之前有考虑这一点。

IDE 集成的方式：
* 诊断输出：如果具有 SARIF 输出，那么 IDE 应该可以直接展示其中的诊断，无需额外的工具
* 自动运行检查并内联展示诊断：可能需要一个静态分析的 LSP 实现
  * 把支持的算法库进行静态或者动态加载
  * 提供 fix 功能

<span class="contributions">贡献点：</span>
* 我认为这个组件目前不是特别重要，但如果有人感兴趣实现这个，那也很好。理想情况下，如果前面几个组件化做好，那么添加此功能会更省事。

#### 数据库 | CI | UI | Docker

os-checker 覆盖了这几个组件：
* 运行并汇集多种检查工具的检查结果，统一地存储在 [redb] 嵌入式数据库文件中
* 从数据库生成基于文件目录的 JSON 静态数据文件，以供前端展示
* 提供 Github Workflows 和 Docker 进行打包、部署和应用

<span class="contributions">贡献点：</span>暂无。

[redb]: https://github.com/cberner/redb
