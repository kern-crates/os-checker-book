# KernMiri


## 讨论记录

### 目标

* 让 KernMiri 作为单独的工具，可被其他 OS 使用：
  * 进一步地完善实现和文档，明确它需要的 shims，以及增加一些用户友好的命令行逻辑之类的
  * 扩展检查功能，让其可以检查一些 OS specific 的问题
* 检测 ArceOS
  * 如何修改 ArceOS 代码来应用 KernMiri 
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
* [中期报告](https://github.com/billlosw/kern_miri/blob/40be3e5141bebecc20e5da84a6fa1aa5c5cba8df/os_training/os_train_midterm.pptx)。
* 修改过 [axplat](https://github.com/billlosw/axplat_crates/tree/axplat-kernmiri)，简易的笔记在 doc 目录。
  * axplat 对应的 [KernMiri 分支](https://github.com/billlosw/kern_miri/tree/KernMiri-2025-05-20)

KernMiri 面临三个工具链的版本：20241104（原先的），20250201（星绽目前的工具链版本），20250520（ArceOS在用的版本）。

## 杂记

<details>

<summary>旧</summary>

miri_asterinas：

分支：<https://github.com/asterinas/atc25-artifact-evaluation/tree/miri_asterinas>

修改内容：
* 添加 miri 等价物的函数调用（或符号？）
* 添加 miri arch 模块来模拟平台代码
  * 有些代码是原地修改的，尤其把 x64 的代码改成 miri 的版本，而未采用条件编译
* osdk 添加 miri 子命令，生成模板代码
* 把指针运算改为数值运算（以规避 miri 的严格的指针检查？）
* 移动测试到单独的模块

kern_miri：

分支：<https://github.com/asterinas/atc25-artifact-evaluation/tree/kern_miri>

修改内容 (shims)：
* 任务管理支持：添加 CPU 和 thread 相关的字段和方法
* 内存管理支持：页表操作、内核内存、物理内存


</details>

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

## KernMiri 源码阅读

### 对 Miri 的改进

1. 内存分类 (memory type)：支持内核内存、static 等内存类型的处理
2. 内存分页管理：物理内存模拟、增加页表映射
3. 线程管理中增加内核栈内存
4. CPU 模拟支持：CPU 局部变量、CPU 绑定线程（比如线程随机化转化为 CPU 随机化）

来自陈工程师的 [overview] 资料。

[overview]: https://hackmd.io/bLRtlCH0T9udi9OLGrrXuQ

### 命令行代码

`src/bin/miri.rs` 是对 rustc 进行包装的程序，编译二进制项目的文件，从入口函数开始分析和检查代码。

包装程序通过编译器驱动程序的方式调用编译器内部的 API 来触发正常的编译，并且注入额外的分析逻辑。

执行编译是通过调用 `rustc_driver::RunCompiler` 来实现的。

注入的方式是实现 `rustc_driver::Callbacks` trait，覆盖其中的一些方法来在某些固定的阶段执行我们编写的回调的代码。
Miri 有两个结构实现了它：
* `MiriCompilerCalls` 是 Miri 分析代码的回调逻辑，根据传递的 Miri 参数进行复杂的代码检查，但不生成二进制文件。
  * `miri::eval_entry` 是进行 Miri 分析的入口；分析代码存放在 `src/lib.rs` 中。
* `MiriBeRustCompilerCalls` 也是 Miri 的分析回调逻辑，但只是在直接转发给编译器时做了一点处理，生成二进制文件。

`cargo-miri/src/main.rs` 是对 cargo 命令的包装，以及一些初始化、与 Rustdoc 交互的逻辑。

### `eval_entry`

核心组成（除了处理错误）：
* `init_miri_physical_mem()`
* `create_ecx(tcx, entry_id, entry_type, &config)`
* `init_boot_pt(&mut ecx)`
* `ecx.set_page_table(page_table)`
* `ecx.run_threads()`

## rustc API

### `rustc_const_eval::interpret`

> An interpreter for MIR used in <abbr title="Compile-Time Function Evaluation">CTFE</abbr> and by miri.

常量解释器是执行 MIR 的虚拟机，不编译到机器码，被编译器和 Miri 共享使用代码。

### MIR 的常量

常量计算的典型场景：
* static item 的 initializer （也就是等号右边的结构）
* 数组长度
* enum variant discriminants （避免不同的变体具有相同的值）
* patterns （避免重叠）
* 更多见 [Reference#const-eval](https://doc.rust-lang.org/reference/const_eval.html)

MIR 中的 constants 分为两类

|            | MIR constants             | Type system constants                        |
|------------|---------------------------|----------------------------------------------|
| 未计算常量 | [mir::Const]              | [ty::Const]                                  |
| 已计算常量 | [mir::ConstValue]         | [ty::ValTree]                                |
| 计算函数   | [op_to_const]             |                                              |
| 含义或用途 | 用作 MIR 的一种 [Operand] | 类型系统中的常量，尤其是数组的长度、常量泛型 |

两者交互：每当常量泛型参数作为 Operand 的时候，[mir::Const::Ty] 把任意的 Type system constants 转化为 MIR constants。

[mir::Const]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/mir/enum.Const.html
[mir::ConstValue]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/mir/enum.ConstValue.html
[op_to_const]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_const_eval/const_eval/eval_queries/fn.op_to_const.html
[ty::Const]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/ty/struct.Const.html
[ty::ValTree]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/ty/struct.ValTree.html
[Operand]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/mir/enum.Operand.html
[mir::Const::Ty]: https://doc.rust-lang.org/nightly/nightly-rustc/rustc_middle/mir/enum.Const.html#variant.Ty

Ref:
* [`rustc_const_eval::interpret`](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_const_eval/interpret/index.html) 模块
* [rustc-dev-guide: const-eval](https://rustc-dev-guide.rust-lang.org/const-eval.html)
* [rustc-dev-guide: const-eval#interpret](https://rustc-dev-guide.rust-lang.org/const-eval/interpret.html)

### Pointer Provenance

Pointer provenance 或者 provenance 描述的是指针携带的分配信息。

该名词翻译为“指针来源”，也就是是指针来自的分配的信息。但 provenance 本身的英文含义是”来源+用于证明的依据“，因此“来源”这个翻译有失偏颇，
它在这个语境下强调这个信息用来证明指针来自该分配。

虽然最终的二进制产物中，指针只有整数（地址信息）一种形式，并没不带有分配的信息，但编译器和优化会考虑 provenance，
将不同的分配产生的指针视为不同的指针。这意味着，源码使用整数（也就是地址）来比较指针，但编译器还使用 provenance 比较指针。

不同的分配可以指以下情况
* 不同的分配器产生的分配
* 相同的分配器在相同的时间段、不同的地址上分配
* 相同的分配器在不同的时间段、对相同地址的分配

[地址][address] 是一个整数，表明进程内存中有数据存放。

[分配][allocation] 表示可从 Rust 中进行寻址的一块内存。
* 可寻址 (addressable) 表示可以通过内存直接定位、进行访问或者操作 —— 也就是通过地址读取值、或者赋给值。
* 这块内存目前 Rust 假设是连续的地址范围、一次全部释放，但将来可能允许有洞、增加或缩小分配范围。
* 创建分配，可发生在堆、栈、globals (statics 和 consts) 等对象上，也可发生在 Rust 无法感知的函数、虚表上。
* 即使是空分配（范围为空，即长度为零），也有基地址 (base address)；如果两个不同的分配有相同的基地址，那么其中一个必须为空分配。
* 分配不会跟踪存放的数据的类型；被分配的数据是按照 [abstract bytes] 存放的。

[provenance]: https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#abstract-byte
[address]: https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#memory-address
[allocation]: https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#allocation
[abstract bytes]: https://rust-lang.github.io/unsafe-code-guidelines/glossary.html#abstract-byte

### Priroda

仓库：<https://github.com/oli-obk/priroda>

由 Oli 编写的图形界面调试器，已不再更新。

但某些编译器 API 或者 Miri 代码注释提及了它。

### 视频

* [Unsafe Rust and Miri by Ralf Jung - Rust Zürisee June 2023](https://www.youtube.com/watch?v=svR0p6fSUYY)
* [oli-obk on miri and constant evaluation](https://www.youtube.com/watch?v=5Pm2C1YXrvM) (2019)
