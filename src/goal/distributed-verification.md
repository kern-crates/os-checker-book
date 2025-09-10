# 分布式和资源节约型验证

## 背景：验证标准库

验证标准库是一项又 Rust 基金会和亚马逊发起的开源项目，通过应用各种形式化验证工具，对 Rust
标准库进行形式化验证，并为整个 Rust 生态的形式化验证奠定基础。

* verify-rust-std：[仓库][verify-rust-std] | [文档](https://model-checking.github.io/verify-rust-std/intro.html)
  * 现有验证工具：Kani | GOTO-Transcoder (ESBMC) | VeriFast | Flux
  * 对此感兴趣的验证工具：Hax | Verus | KMIR | RAPx
* Rust 项目目标：
  * 2024H2: [Contracts and Invariants](https://rust-lang.github.io/rust-project-goals/2024h2/Contracts-and-invariants.html)
  * 2024H2: [Survey tools suitability for Std safety verification](https://rust-lang.github.io/rust-project-goals/2024h2/std-verification.html)
  * 2025H1: [Instrument the Rust standard library with safety contracts](https://rust-lang.github.io/rust-project-goals/2025h1/std-contracts.html)
* 相关链接：
  * (2024.11) Rust 基金会公告：《[与 AWS Initiative 合作验证 Rust 标准库](https://foundation.rust-lang.org/news/rust-foundation-collaborates-with-aws-initiative-to-verify-rust-standard-libraries/)》
  * (2024.11) AWS 公告：《[验证 Rust 标准库的安全性](https://aws.amazon.com/cn/blogs/opensource/verify-the-safety-of-the-rust-standard-library/)》
  * (2024.10) SPLASH HATRA '24《[Surveying the Rust Verification Landscape](https://2024.splashcon.org/details/hatra-2024-papers/5/Surveying-the-Rust-Verification-Landscape)》
  * (2025.07) Linux 基金会的 Open Source Summit 演讲《[Verifying the Rust Standard Library](https://youtu.be/8_lzVNs1uPk)》
  * (2025.07) Rust 基金会博客：《[扩展 Rust 形式验证生态系统：欢迎 ESBMC](https://rustfoundation.org/media/expanding-the-rust-formal-verification-ecosystem-welcoming-esbmc/)》

[verify-rust-std]: https://github.com/model-checking/verify-rust-std

## 背景：GSoC Rust 2025

> 实现 [Distributed and resource-efficient verification](https://github.com/rust-lang/google-summer-of-code/tree/45141d74c28d91e114cf621d2d56aea6c3f82547?tab=readme-ov-file#distributed-and-resource-efficient-verification),
> GSoC Rust 2025 （谷歌开源之夏项目中，在 verify-rust-std 项目中的一个提议）

（以下是该提议的译文）

我们的目标是使用安全契约来检测 Rust 标准库。通过这种方法，我们将从指定 unsafe 函数的安全要求的非正式注释转变为可执行的 Rust 代码。
为了证明这些安全要求在所有可能的执行中都成立，我们需要运行静态分析工具。

我们进一步想证明这些安全要求仍然有效 尽管代码发生了变化。因此，我们需要运行静态分析工具来证明这些安全性持续集成的要求。
我们已经在分叉的 verify-rust-std 仓库中开始这样的努力。

静态分析工具通常需要计算工作量，每个需要证明的函数可能需要长达数分钟。因此，随着合约数量的增长，我们需要将证明工作量分散到多个节点。

然而，如果我们能够确定任何给定的证明必然持续成立，因为合约所管辖的函数的传递闭包自上次证明完成以来从未被修改过，那么我们也可以减少证明工作量。

我们正在寻找以下领域的贡献：
1. 可达性分析或影响分析，在代码更改的情况下，确定哪些证明需要重新验证。这将避免因不必要的重新验证而浪费计算资源。
2. 一个将证明分片到多个节点并重新组合其结果的系统。该系统可能涉及多种验证工具，并且这些工具的性能可能各不相同。
   负载平衡可以以动态方式实现（需要节点间通信，但这对 GitHub 运行器来说可能难以实现），或者按照预先计算的分布方式实现（将预定义的验证任务集映射到运行器）。
   在后一种情况下（可能更可行），由于我们预计验证任务集会随时间变化，因此分布应该易于通过配置文件进行调整。

## distributed-verification

> 仓库：[distributed-verification](https://github.com/os-checker/distributed-verification)
> |
> UI：[os-checker.github.io/distributed-verification](https://os-checker.github.io/distributed-verification/)

distributed-verification （简称 dv）是我参加谷歌开源之夏、实现上述提议的一个工具。

该工具提供两部分功能：
1. 将 Kani 的形式化证明 (harnesses) 放到多个 Github CI 机器上分布式和并行地执行，而且提供一个缓存功能：
   当验证的代码以及涉及的所有调用在源代码级别没有发生变化，则无需重新验证。
2. 提供 UI 界面：展示这些证明的信息和验证情况，以及被验证函数的信息。

![](https://github.com/user-attachments/assets/21d9d8e1-969f-4aad-89b2-810905024ce5)

当前进展：
1. 编写了 rustc 包装器，并从 Kani 中复制了可达性分析 (reachability) 代码，从而得到调用关系图，计算函数和证明的哈希值（用于比较是否变化）；
2. 编写了前端静态网页，聚合 Kani 和 dv 的数据，在网页中展示证明和函数的信息。

下一步：
* 编写 CI/Workflow 脚本，把 dv 集成进 verify-rust-std，进行 Kani 验证并比较节省的验证时间。
* 清理 dv 的 [issues](https://github.com/os-checker/distributed-verification/issues)，打磨细节。

（该项目我延期了 10 周，从 8 月底结束延长到 11 月结束。）
