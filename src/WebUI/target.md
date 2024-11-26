# 编译目标明细表

地址：[os-checker.github.io/target](https://os-checker.github.io/target)

编译目标是 os-checker 必须明确的一个核心内容，因为每个检查命令由以下部分组成

* 检查工具的二进制文件名称：os-checker CLI 会检测和安装检测工具以及检查工具所需的工具链环境；
* 检查工具的命令行参数：每个检查工具有自己的命令行参数；
* 编译参数：检查工具可能与 rustc/Cargo 共用编译参数
    * 虽然不是所有检查工具都需要编译，但大部分必须编译代码才能检查，即便对于静态检查工具，可能也必须到达某个编译阶段（比如 MIR）才能开始检查代码。
    * 其中，一类棘手的编译参数是条件编译相关的参数，比如 target[^2]、features 和 rustc flags。后两种尚未支持。
    * 还有 Cargo 的工具链参数，即 `+toolchain`，os-checker 假设一个检查命令只在一个工具链上被运行，并且 CLI 有自己的方式应用和指定工具链参数。

[^2]: 注意：[target](https://doc.rust-lang.org/cargo/appendix/glossary.html#target) 在 Rust/Cargo 中是一个多义词，不同的上下文具有不同的含义。在 os-checker 中，它指编译目标三元组 (Target Triple)。

条件编译参数之所以棘手，是因为它很难由程序自动识别，从而无法确定一个 pkg 支持哪些编译平台或者条件组合。

os-checker 以混合方式处理 target，它会启发式地在 pkg 和仓库的配置或脚本中搜索 target 名称，然后假设它适用于这个 pkg；当我们观察到检查结果只是因为不适用于该 target，那么需要通过正确的配置来执行正确的检查。

由于大部分仓库或者 pkg 都不那么复杂，因此这种天真地搜索是足够的。不过，对于较为复杂项目布局的仓库，搜索的 target 只能作为一种参考。

指定错误的 target （或者编译条件），[可能在不重要的地方报告上千条错误](https://github.com/os-checker/os-checker/discussions/59)，因此，我们需要人为地编写配置文件来让 os-checker 在特定的 target （或者编译条件）上检查。

这种混合的方式天然地应该分成两个表，并且具有一些关联。也就是说，编译目标信息 (target) 分为两个角度：

* [Sources](https://github.com/os-checker/os-checker/blob/9196ccabf686096a45b0985f74aa927dfddad738/os-checker-types/src/layout.rs#L66-L79): 表示启发式搜索 targets 的路径来源，如 config.toml、rust-toolchain.toml、Cargo.toml、脚本等；
* [Resolved](https://github.com/os-checker/os-checker/blob/9196ccabf686096a45b0985f74aa927dfddad738/os-checker-database/src/targets/mod.rs#L10-L16): 表示生成和应用的检查命令，它表示最终在哪个编译目标上如何检查代码，因此包括 target、toolchain、checker、cmd 信息。[^3]

Sources 和 Resolved 中的 target 会存在差异，原因在于 JSON 配置文件可能部分或者完全指定编译目标，从而带来两个指标：
* used：一个 target 是否同时在 Sources 和 Resolved 中
  * true 表示搜索到的 target 应用到了检查命令上；
  * false 表示搜索到的 target 未应用到检查命令上（比如配置文件指定不要在这个 target 上检查）；
* specified：一个 target 是否在配置文件中被指定检查
  * true 意味着 target 一定在 Resovled 中，因为配置文件直接决定生成什么检查命令；
  * false 意味着未在配置文件中指定 target，通常是这个值，因为大部分情况下配置文件没有指定 target。

[^3]: 你可能会问，为什么不展示配置文件的 target，那是因为这个信息可以被压缩成一个字段，而无需一个表；此外更重要的是，我认为知道最终生成的完整命令比展示配置文件更有价值 —— 这个完整命令包含更多信息，而且如果有人想尝试重现检查结果，可以直接使用它。

## 示例：简单情况

![截图_20241009180605](https://github.com/user-attachments/assets/d0b9d8f3-2863-462c-900d-1e72b2de7abf)

上面展示了 `kern-crates/axalloc` 的 target 明细情况，当你下拉选择 user 和 repo 之后，直接得到这个结果。具体解读是：
* 所有下拉框在只有一个值的时候会自动填写，因此我们阅读下拉框，就可以知道该仓库只有一个 pkg，并只在 x86_64-unknown-linux-gnu target 和最近的一个夜间工具链上检查。
* 从上到下，第一个表为 Resolved，它表明在 axalloc pkg 目录下应用了 4 个检查命令（Cmd），并且附上了检查结果数量（Count） 和检查时间（ms，毫秒）。你可能会疑惑为什么有些 target 和 toolchain 与 cmd 内的并不一致。
    * 上面唯一没有指定 target 参数的是 fmt，因为 `rustfmt`/`cargo fmt` 并不编译代码，它在项目级别进行格式化检查，因此无需指定 target 参数，但它是工具链相关的，因为它是 rustup 的一个组件；这里 os-checker 强制调用主机工具链上的 fmt 进行检查。
    * 而在 toolchain 上，该仓库未指定工具链，所以采用主机工具链。但 cmd 是实际运行的命令，如果要完全复现它，安装最新的工具链是不行的，因为夜间工具链是每天更新的，因此我们需要固定工具链到检查日那天，os-checker 转写了主机工具链。对于 lockbud 和 mirai 这类静态检查工具，由于编译器驱动代码非常不稳定，它们需要固定自己的工具链，所以必须在那个工具链上检查，从而与主机工具链不一致。
* 第二个表为 Sources，它表示该仓库没有识别到任何有路径的 target 来源，因此只在默认的主机 target 上（即 x86_64-unknown-linux-gnu）检查，这被应用到了最终的检查命令上，而且 os-checker 的配置文件没有指定任何 target。

## 示例：embassy

embassy 是比较复杂的代码库，包含 90+ 个 pkgs，但 os-checker [只检查库代码](https://github.com/os-checker/os-checker/blob/4cfa7821513cbb1aadf473fdd6ebb03892f42832/assets/repos-embassy.json)，而不检查大部分示例代码。

它是目前唯一一个在 target 种类和来源方面最丰富的仓库，因此具有很多边缘情况，适合解释 Sources 的功能。

选择 embassy 仓库，我们可以看到工具链直接填上了 1.78，说明它的仓库工具链为稳定版的 1.78。

选择第二行 pkg 中的 embassy-basic-example，可以看到如下结果

![截图_20241009211615](https://github.com/user-attachments/assets/5ccad5ac-7de4-40ed-903d-efb66dc2f45f)

首先，Resolved 表没有任何数据，这是因为橙色下拉框虽然只适用于 Sources 表，但在 Pkg 和 Target 字段上，它们被设计作为共同的筛选条件，选择它们将作用于这两个表 —— 这种联动有利于查询数据差异，从而更大地发挥数据的价值。

如前所述，这是预期的结果，因为我们在配置文件中，禁用了所有示例 pkgs 上的检查，它们的 Resolved 表都没有数据 —— 换句话说，这也就是为什么有两个 Pkg 和 Target 筛选条件的原因，数据源头和差异导致它们必须分开选择。Used 一列的 ❌ 也表明，尽管 os-checker 识别到仓库在该 pkg 上指定了这些 targets，但最终没有应用到检查命令上。

其次，我们可以清晰地看到每个 target 来源及其文件路径，比如 ci.sh 和 ci-nightly.sh 两个脚本文件中包含 thumbv6m-none-eabi、在 rust-toolchain.toml 文件中指定了 `thumbv{6,7}m-none-eabi`、在 xxx/.cargo/config.toml[^4] 中指定了 thumbv7m-none-eabi 等等。这表明，os-checker 记录了每个 target 的搜索方式，如果有人想知道为什么在那个 target 上面检查代码，这个表就是很好的回答。

[^4]: 注意：所有来源路径都是仓库根目录的相对路径。


最后，作为 CargoTomlDocsrsInPkgDefault 来源示例，embassy 的 embassy-executor pkg 通过 Cargo.toml 配置的 [`[package.metadata.docs.rs]`](https://docs.rs/about/metadata) 表表达了一个默认 target 为 thumbv7em-none-eabi 的意图。

![截图_20241009220343](https://github.com/user-attachments/assets/6c615b1f-e8d0-458d-9892-cf2170c7e54f)


它带来的效果为 https://docs.rs/embassy-executor/0.6.0/embassy_executor/

![截图_20241009220131](https://github.com/user-attachments/assets/69da74b2-cf8d-4584-acf9-939b5c9bab37)


[这是一个小众但伪标准化的技巧](https://github.com/os-checker/os-checker/issues/26#issuecomment-2302201030)。

