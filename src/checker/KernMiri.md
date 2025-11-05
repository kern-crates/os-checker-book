# KernMiri

## miri_asterinas

分支：<https://github.com/asterinas/atc25-artifact-evaluation/tree/miri_asterinas>

修改内容：
* 添加 miri 等价物的函数调用（或符号？）
* 添加 miri arch 模块来模拟平台代码
  * 有些代码是原地修改的，尤其把 x64 的代码改成 miri 的版本，而未采用条件编译
* osdk 添加 miri 子命令，生成模板代码
* 把指针运算改为数值运算（以规避 miri 的严格的指针检查？）
* 移动测试到单独的模块

## kern_miri

分支：<https://github.com/asterinas/atc25-artifact-evaluation/tree/kern_miri>

修改内容 (shims)：
* 任务管理支持：添加 CPU 和 thread 相关的字段和方法
* 内存管理支持：页表操作、内核内存、物理内存


## 讨论记录

### 目标

* 让 KernMiri 作为单独的工具，可被其他 OS 使用：
  * 进一步地完善实现和文档，明确它需要的 shims，以及增加一些用户友好的命令行逻辑之类的
  * 扩展检查功能，让其可以检查一些 OS specific 的问题
* 完善 KernMiri：
  * 让 KernMiri 支持最新的星绽分支
  * 集成 KernMiri 到星绽 CI

<details>

<summary>旧</summary>

背景：在向勇老师的操作系统专题训练课上，罗绍玮同学对 KernMiri 感兴趣。

目标：将 KernMiri 集成到 os-checker。

问题：
* 可行性？：KernMiri 可以作为单独的工具吗？
  * 不特定于具体的操作系统？从星绽中剥离而应用到别的 Rust 操作系统（比如 Arceos）？
  * 或者集成进上游 Miri？
* 可复现？
* 实现思路？

</details>

### 落地步骤

陈澄钧工程师罗列的一些方向：
1. 熟悉现有 KernMiri 代码和架构，理解其相关设计。并优化一些代码组织和实现性能等问题。
2. 在星绽方面的适配，主要让 KernMiri 可以适配 osdk 和 ktest 框架
3. 调研若干 Rust OS，研究移植 KernMiri 所需要的 shims，确定最终 KernMiri 所需要提供的接口。
4. 用户友好性加强，让 KernMiri 的配置性提高，且不影响 miri 其他场景下的使用。同时努力做到让用户 OS 所需要的侵入式改动能容易地被 cfg 滤去。
5. 完善 KernMiri 实现和功能；目前 KernMiri 模拟多核的实现上仍有欠缺；同时可以进一步之前可检查的 OS UB。

罗绍玮同学的课程工作：
* [中期报告](https://github.com/billlosw/kern_miri/blob/40be3e5141bebecc20e5da84a6fa1aa5c5cba8df/os_training/os_train_midterm.pptx)


## 杂记

### 从源码构建 Rust

Miri 使用特定提交的 Rust，但这些提交构建的 Rust 工具链在几个月之后就会被清理而无法下载。

解决措施有两个：
* 采用最近的一个夜间版本。通常该提交之后的第一个夜间版本与这个提交的 Rust 
  代码改动不大，因此可以编译 Miri。在 Miri 脚本中，需要一个叫做 miri 的工具链，因此可以通过以下方式替换
  ```bash
  rustup toolchain install nightly-2024-11-03
  rustup toolchain link miri $(rustc +nightly-2024-11-03 --print sysroot)
  ```

* 源码构建这个特定的 Rust。这可能还需要源码编译 LLVM，因此需要花费很长的时间。如果最近的那个夜间版本不能编译
  Miri，那么才需要这么做：
  ```bash
  # 我们需要完整的工具链，因此采用 stage2
  rust $ ./x build --stage 2 rustfmt clippy cargo
  rust $ rustup toolchain link miri build/aarch64-unknown-linux-gnu/stage2
  ```

  ```toml
  profile = "tools"
  change-id = 132494

  # Use locally built tools, instead of downloading them.
  [build]
  cargo = "/home/gh-zjp-CN/.rustup/toolchains/nightly-2024-11-03-aarch64-unknown-linux-gnu/bin/cargo"
  rustc = "/home/gh-zjp-CN/.rustup/toolchains/nightly-2024-11-03-aarch64-unknown-linux-gnu/bin/rustc"
  rustfmt = "/home/gh-zjp-CN/.rustup/toolchains/nightly-2024-11-03-aarch64-unknown-linux-gnu/bin/rustfmt"
  cargo-clippy = "/home/gh-zjp-CN/.rustup/toolchains/nightly-2024-11-03-aarch64-unknown-linux-gnu/bin/cargo-clippy"

  [rust]
  channel = "nightly" # full toolchain, otherwise dev toolchain won't generate commit-hash or other stuff
  download-rustc = false

  [llvm]
  download-ci-llvm = false
  ```

相关的讨论：
* [Build with nightly Miri (not arbitrary master snapshot)?](https://rust-lang.zulipchat.com/#narrow/channel/269128-miri/topic/Build.20with.20nightly.20Miri.20.28not.20arbitrary.20master.20snapshot.29.3F/with/545808655)
* [How to install an old commit version of rustc?](https://rust-lang.zulipchat.com/#narrow/channel/131828-t-compiler/topic/.E2.9C.94.20How.20to.20install.20an.20old.20commit.20version.20of.20rustc.3F/with/541784962)
