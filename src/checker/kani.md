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
\```
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
\```
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
