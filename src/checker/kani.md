# Kani

## 源码安装

文档：<https://model-checking.github.io/kani/build-from-source.html>

```bash
# 下载 Kani 仓库及其子模块（含 Charon）
git clone https://github.com/model-checking/kani.git
cd kani
git submodule update --init

# 安装依赖（针对 Ubuntu；macOS 使用另一个脚本）
./scripts/setup/ubuntu/install_deps.sh

# 本地编译 Kani 的完整代码
# 注意 cargo build 并不会构建完整的代码
# Kani 有自己包装程序进行安装和源码构建
cargo build-dev

# 可选：把本地源码构建的 Kani 应用到任何地方
# export PATH=$(pwd)/scripts:$PATH
```

## 使用方式

文档：<https://model-checking.github.io/kani/usage.html>

* 命令行
  * `kani filename.rs [OPTIONS]` 分析单个文件中的所有 proofs
  * `cargo kani [OPTIONS]` 分析 package 内的所有 proofs
* Cargo.toml 配置选项
  * `[workspace.metadata.kani.flags]` 或 `[package.metadata.kani.flags]`

源码：把 proofs 放到受 `cfg(kani)` 条件编译控制的模块；即使是 tests 目录下的测试文件，也应该采用这种模式

```rust
#[cfg(kani)]
mod verification {
    use super::*;

    #[kani::proof]
    pub fn check_something() {
        // ....
    }
}
```

## 一些有趣的设计

### 验证 Rust 标准库

Kani 验证 Rust 标准库的 CStr API 的安全性的详细思路和步骤，它来自 Kani 博客，非常值得一读：

* (2024.12.03)：[Verifying Safety of Rust's CStr](https://model-checking.github.io/kani-verifier-blog/2024/12/03/safety-of-cstr.html)》

### 结合模糊测试框架

`#[kani::proof]` ：

* (2022.10.27)：[From Fuzzing to Proof: Using Kani with the Bolero Property-Testing Framework](https://model-checking.github.io/kani-verifier-blog/2022/10/27/using-kani-with-the-bolero-property-testing-framework.html)

其中 [Bolero] 是 AWS 的 [s2n-quic] 作者编写的测试框架，该工具提供统一的接口，轻松在 LibFuzzer、AFL、Honggfuzz 和 Kani 之间切换，只需更改
`cargo bolero test` 的 `--engine` 参数。

[Bolero]: https://github.com/camshaft/bolero/
[s2n-quic]: https://github.com/aws/s2n-quic


### 提供失败的反例

文档：<https://model-checking.github.io/kani/reference/experimental/concrete-playback.html>

`-Z concrete-playback --concrete-playback=[print|inplace]` 参数可以在失败的 proof 上知道具体的输入值，其中
* `print` 打印单元测试
* `inplace` 直接在源码中插入这个重现失败的单元测试，从而使用者可以分析甚至编译它来对单元测试的二进制文件进行进一步调试。

示例：对于以下 `a.rs` 文件

```rust
#[kani::proof]
fn f() {
  let a: u8 = kani::any();
  assert_eq!(a, 1, "a ≠ 1");
}
```

运行 `kani -Z concrete-playback --concrete-playback=print a.rs` 命令：

```text
RESULTS:
Check 1: f.assertion.1
         - Status: FAILURE
         - Description: ""a ≠ 1""                                                                                                                                        - Location: a.rs:8:3 in function f


SUMMARY:
 ** 1 of 1 failed
Failed Checks: "a ≠ 1"
 File: "a.rs", line 8, in f

VERIFICATION:- FAILED
Verification Time: 0.09984675s

Concrete playback unit test for `f`:
---
/// Test generated for harness `f`
///
/// Check for `assertion`: ""a ≠ 1""

#[test]
fn kani_concrete_playback_f_2469314071636892245() {
    let concrete_vals: Vec<Vec<u8>> = vec![
        // 255
        vec![255],
    ];
    kani::concrete_playback_run(concrete_vals, f);
}
---
INFO: To automatically add the concrete playback unit test(s) to the src code, run Kani with `--concrete-playback=inplace`.
Manual Harness Summary:
Verification failed for - f
Complete - 0 successfully verified harnesses, 1 failures, 1 total.
```

### 动态库替换技巧

Kani 使用动态库技巧，直接传递编译器参数来注入依赖，无需使用者在自己的项目中声明 `kani` 依赖。


```
[
  ..
  "--cfg=kani",
  "-Z",
  "crate-attr=feature(register_tool)",
  "-Z",
  "crate-attr=register_tool(kanitool)",
  "--sysroot",
  "/home/zjp/rust/kani/target/kani",
  "-L",
  "/home/zjp/rust/kani/target/kani/lib",
  "--extern",
  "kani",
  "--extern",
  "noprelude:std=/home/zjp/rust/kani/target/kani/lib/libstd.rlib",
  ..
]
```

安装 Kani 将自动下载所需的预编译的动态库（包括 core 和 std），执行时直接替换为 Kani 的 sysroot（默认在 `~/.kani/{version}` 目录下），而不是本地的 sysroot[^1] ([源码][setup])
—— 在调试模式下（需使用 `export PATH=/path/to/kani/scripts:$PATH` 替换全局安装的 Kani），它指向源码目录下的 `target/kani` 目录。

[setup]: https://github.com/model-checking/kani/blob/7a126c2b2e16b843d94bcd850a17375c3299b826/src/setup.rs

[^1]: 但这貌似造成某些 RUSTFLAGS 环境变量传递失效。

### 评估是否值得证明

文档：<https://model-checking.github.io/kani/dev-assess.html>

Kani 的 `cargo kani assess` 的相关命令可以输出一些评估数据，来帮助包的作者决定在当前的测例情况上编写证明，以及了解
kani 的哪些功能不支持你的测例。

示例输出：

```text
======================================================
 Unsupported feature           |   Crates | Instances
                               | impacted |    of use
-------------------------------+----------+-----------
 caller_location               |       71 |       239
 simd_bitmask                  |       39 |       160
...

================================================
 Reason for failure           | Number of tests
------------------------------+-----------------
 unwind                       |              61
 none (success)               |               6
 assertion + overflow         |               2
...
```

## GSoC Proposal

Rust 项目中的 GSoC 中有两项与 verify-rust-std 直接相关。

我决定参加 “[Distributed and resource-efficient verification][GSoC]”。

[GSoC]: https://github.com/rust-lang/google-summer-of-code?tab=readme-ov-file#distributed-and-resource-efficient-verification

该任务的主要内容是让验证标准库的 CI 运行地更快一些，通过把验证的清单拆分到多个 Github Runner 机器上，并发地执行验证，并且通过可达性分析，跳过未改动的证明，只验证代码改动影响的证明。

虽然 verify-rust-std 还聚集了其他几个验证工具，但是采用最多、耗时最长的是 kani。因此主要聚焦于缩短 kani 的验证时间，尤其当 proofs 的数量逐渐增加时。

现状：当前 kani 在 verify-rust-std 仓库总共需要大约 10 小时机器时间，在 Linux 和 macOS 上分别运行验证代码，并最终生成验证清单；通过将验证清单分批分散到 4 个独立的 runner 
机器上并行执行，从而 CI 时间取决于最耗时的那个 runner，从而需要大约 2 小时人类时间。

本节定义：
* proofs：标注了 `#[kani::proof]` 和 `#[kani::proof_for_contract]`，甚至 `-Zautoharness`（虽然目前尚未在 verify-rust-std 中使用）的证明函数。

提议实施以下步骤来完成该任务：
1. 获取所有 proofs 的基本信息，尤其是名称和对应的 hash 值（下文介绍）
2. 下载上次运行结果，尤其获取所有 proofs 名称、hash 值以及实际执行耗时
3. 通过对比 hash 值的差异，计算改动的 proofs，得到待运行 proofs 清单
4. 在多个 runner 上并行地完成改动的 proofs，并记录耗时
5. 上传所有 proofs 的信息：未改动的 proofs 直接使用上一次的运行结果，新运行的 proofs 使用这次的运行结果

细节：
* 每个 proof 携带一个 hash 值，它基于 kani 的可达性分析，通过提取以下信息计算
  * proof 自身的信息（整个函数及其属性）
  * 以及所有内部函数调用的信息（被调用函数及其属性）
* 下载和上传数据
  * 使用 npm 的 [artifact API 接口](https://github.com/actions/toolkit/tree/main/packages/artifact)
  * 需要维护一个列表记录完成的缓存信息，以便寻找上一次的运行结果；上传所有分区数据之后才汇总并更新这个列表
  * 工件的名称格式为 `kani-commit_hash-{0,1,2,...}`，其中 0 表示汇总数据，其他对应 Runner 分区数据
* 分布式更新数据
  * 方案一（推荐）：提前计算每个 Runner 执行的所有 proofs；这不需要同步 runner 之间的状态；将来可以通过历史耗时实现负载均衡
  * 方案二：提前计算每个 Runner 执行的一部分 proofs，当 runner 完成一批之后，获取下一批 proofs 运行；这需要同步状态；由于每次改动的涉及的 proofs 并不多，暂时不考虑动态清单策略
* 其他
  * 需要处理失败：如果验证期间因为失败而提前退出，应该仍然产生以及执行过的 proofs 的结果，并把结果上传（未执行的 proofs 应该有额外的状态标记）
  * 多个 PR 的 CI 同时触发，将造成重复验证，但不应该相互影响，因为每个 PR 获取的上一次运行结果在其各自的 CI 期间不会发生变化
  * 提供强制运行所有 proofs 的配置选项
