# Charon

<style>
.TP { font-weight: bold; color: gray; background-color: #fff9ed; padding: 0px 2px; }
.TN { font-weight: bold; color: green; background-color: #fff9ed; padding: 0px 2px; }
.FP { font-weight: bold; color: red; background-color: #fff9ed; padding: 0px 2px; }
.FN { font-weight: bold; color: orange; background-color: #fff9ed; padding: 0px 2px; }
</style>

## 工具介绍

仓库：<https://github.com/AeneasVerif/charon>

论文：《 [Charon: An Analysis Framework for Rust][paper] 》

[paper]: https://arxiv.org/abs/2410.18042

<img src="https://github.com/user-attachments/assets/2d7cb20e-0deb-4f2e-af9f-a2145ee3591b" style="width: 70%; display: block; margin: auto;">

Charon 框架
* 目标：Charon 旨在为 Rust 分析工具提供一个通用的、稳定的接口，隐藏底层复杂性，提供一个适合程序分析的数据源。

设计与实现：
* Charon 通过与 Cargo 和 Rust 编译器的深度集成，提供了一个完整的、装饰过的 AST，简化了工具开发者的任务。
* Charon 提供了两种表示形式：ULLBC（无结构低级借用演算）和 LLBC（低级借用演算），分别用于控制流图（CFG）和抽象语法树（AST）的分析。
* Charon 还支持跨语言的输出格式（如 JSON 和 OCaml），便于不同语言的工具开发者使用。

通过多个案例研究验证了 Charon 的适用性，包括：
* 常量时间分析：检测加密代码中的侧信道漏洞。
* Rudra 分析器移植（[charon-rudra]）：将现有的 Rust 静态分析工具 Rudra 重实现到 Charon 上，验证了其有效性。
* [Aeneas] 验证框架：通过将 Rust 程序转换为纯函数式程序进行验证。
* [Eurydice] 编译器：将 Rust 编译为 C 语言，以便在不支持 Rust 的环境中使用。

[charon-artifact]: https://zenodo.org/records/13983686
[charon-rudra]: https://github.com/AeneasVerif/charon-rudra
[Aeneas]: https://github.com/AeneasVerif/aeneas
[Eurydice]: https://github.com/AeneasVerif/eurydice

<details>

<summary>charon-artifact</summary>

[charon-artifact.tar.gz][charon-artifact]

This artifact is the companion to the TACAS 25 tool paper submission: "Charon: An Analysis Framework for Rust".
It contains the different components presented in the paper, namely:
* The Charon framework
* A novel implementation of a taint analysis for cryptographic constant-time
* A reimplementation of Rudra on top of Charon
* The Aeneas verification framework
* The Eurydice compiler, relying on Charon to generate C code

To foster reproducibility, this artifact includes a Dockerfile, as well as instructions to use Docker to set up a development environment to reproduce claims in the paper, as well as experiment with the different tools.

</details>

<details>

<summary>相关术语：ULLBC 和 LLBC 介绍</summary>

在论文中，ULLBC （Unstructured Low-Level Borrow Calculus）和 LLBC （Low-Level Borrow Calculus ）是 Charon 框架为 Rust 语言设计的两种抽象语法树（AST）表示形式，用于简化 Rust 程序的分析。

 ULLBC （ Unstructured  Low - Level  Borrow  Calculus ）
- **定义**： ULLBC 是基于 Rust 编译器中间表示（MIR）的控制流图（CFG）表示形式。它保留了 MIR 的低级语义，如移动（move）、复制（copy）和显式的借用（borrow）与重新借用（reborrow）。
- **特点**：
  - 提供即时的上下文和语义信息（如类型和 trait ），隐藏了实现细节（如常量的表示形式）。
  - 解决了 MIR 中的一些问题，例如 trait 信息的解析、常量的统一表示以及 Steal 抽象的移除。
  - 对 MIR 进行清理和重构，例如将整数溢出和数组越界的断言语义简化为单个操作，将模式匹配从低级表示转换为更高级的形式。
- **用途**： ULLBC 适用于需要控制流图表示的分析工具，提供了更简洁、语义化的 MIR 视图。


 LLBC （Low-Level Borrow Calculus ）
- **定义**：LLBC 是通过控制流重构算法从 ULLBC 生成的结构化 AST 表示形式。
- **特点**：
  - 将 ULLBC 的 CFG 转换为结构化的循环和分支，减少了错误的可能性。
  - 保留了 ULLBC 的语义，但以更高级的 AST 形式呈现，适合需要结构化表示的分析工具。
- **用途**： LLBC 特别适用于需要结构化控制流的分析工具，如编译器（如 Aeneas 和 Eurydice）。

总结：
ULLBC 和 LLBC 是 Charon 框架为 Rust 分析工具提供的两种表示形式，分别适用于不同的分析需求。 ULLBC 提供了清理后的 MIR 视图，适合基于 CFG 的分析；而 LLBC 则进一步重构为结构化的 AST ，适合需要高级表示的工具。

</details>

## 使用说明

基于 `48d01c5` 提交，而不是最新提交；使用 `nightly-2024-08-11` Rust 工具链。

<details>

<summary><code>charon --help</code></summary>

```bash
~ $ charon --help
Usage: charon [OPTIONS]

Options:
      --ullbc                    Extract the unstructured LLBC (i.e., don’t reconstruct the control-flow)
      --lib                      Compile the package’s library
      --bin <BIN>                Compile the specified binary
      --mir_promoted             Extract the promoted MIR instead of the built MIR
      --mir_optimized            Extract the optimized MIR instead of the built MIR
      --crate <CRATE_NAME>       Provide a custom name for the compiled crate (ignore the name computed by Cargo)
      --input <INPUT_FILE>       The input file (the entry point of the crate to extract). This is needed if you want to define a custom entry point (to only extract part of a crate for instance)
      --dest <DEST_DIR>          The destination directory. Files will be generated as `<dest_dir>/<crate_name>.{u}llbc`, unless `dest_file` is set. `dest_dir` defaults to the current directory
      --dest-file <DEST_FILE>    The destination file. By default `<dest_dir>/<crate_name>.llbc`. If this is set we ignore `dest_dir`
      --polonius                 If activated, use Polonius’ non-lexical lifetimes (NLL) analysis. Otherwise, use the standard borrow checker
      --no-code-duplication      Check that no code duplication happens during control-flow reconstruction
                                 of the MIR code.

                                 This is only used to make sure the reconstructed code is of good quality.
                                 For instance, if we have the following CFG in MIR:
                                   ```
                                   b0: switch x [true -> goto b1; false -> goto b2]
                                   b1: y := 0; goto b3
                                   b2: y := 1; goto b3
                                   b3: return y
                                   ```

                                 We want to reconstruct the control-flow as:
                                   ```
                                   if x then { y := 0; } else { y := 1 };
                                   return y;
                                   ```

                                 But if we don’t do this reconstruction correctly, we might duplicate
                                 the code starting at b3:
                                   ```
                                   if x then { y := 0; return y; } else { y := 1; return y; }
                                   ```

                                 When activating this flag, we check that no such things happen.

                                 Also note that it is sometimes not possible to prevent code duplication,
                                 if the original Rust looks like this for instance:
                                   ```
                                   match x with
                                   | E1(y,_) | E2(_,y) => { ... } // Some branches are "fused"
                                   | E3 => { ... }
                                   ```

                                 The reason is that assignments are introduced when desugaring the pattern
                                 matching, and those assignments are specific to the variant on which we pattern
                                 match (the `E1` branch performs: `y := (x as E1).0`, while the `E2` branch
                                 performs: `y := (x as E2).1`). Producing a better reconstruction is non-trivial.

      --extract-opaque-bodies    Usually we skip the bodies of foreign methods and structs with private fields. When this flag is on, we don’t
      --include <INCLUDE>        Whitelist of items to translate. These use the name-matcher syntax (note: this differs
                                 a bit from the ocaml NameMatcher).

                                 Note: This is very rough at the moment. E.g. this parses `u64` as a path instead of the
                                 built-in type. It is also not possible to filter a trait impl (this will only filter
                                 its methods). Please report bugs or missing features.

                                 Examples:
                                   - `crate::module1::module2::item`: refers to this item and all its subitems (e.g.
                                       submodules or trait methods);
                                   - `crate::module1::module2::item::_`: refers only to the subitems of this item;
                                   - `core::convert::{impl core::convert::Into<_> for _}`: retrieve the body of this
                                       very useful impl;

                                 When multiple patterns in the `--include` and `--opaque` options match the same item,
                                 the most precise pattern wins. E.g.: `charon --opaque crate::module --include
                                 crate::module::_` makes the `module` opaque (we won’t explore its contents), but the
                                 items in it transparent (we will translate them if we encounter them.)

      --opaque <OPAQUE>          Blacklist of items to keep opaque. Works just like `--include`, see the doc there.
      --exclude <EXCLUDE>        Blacklist of items to not translate at all. Works just like `--include`, see the doc there.
      --hide-marker-traits       Whether to hide the `Sized`, `Sync`, `Send` and `Unpin` marker traits anywhere they show up
      --no-cargo                 Do not run cargo; instead, run the driver directly
      --rustc-flag <RUSTC_ARGS>  Extra flags to pass to rustc
      --cargo-arg <CARGO_ARGS>   Extra flags to pass to cargo. Incompatible with `--no-cargo`
      --abort-on-error           Panic on the first error. This is useful for debugging
      --error-on-warnings        Consider any warnings as errors
      --no-serialize             Don’t serialize the final (U)LLBC to a file.
      --print-original-ullbc     Print the ULLBC immediately after extraction from MIR.
      --print-ullbc              Print the ULLBC after applying the micro-passes (before serialization/control-flow reconstruction).
      --print-built-llbc         Print the LLBC just after we built it (i.e., immediately after loop reconstruction).
      --print-llbc               Print the final LLBC (after all the cleaning micro-passes).
      --no-merge-goto-chains     Do not merge the chains of gotos in the ULLBC control-flow graph.

  -h, --help                     Print help
```

</details>

部分参数介绍：
* `--print-llbc`：终端打印人类可读的 llbc 格式
* `--no-cargo --input xx.rs`：只分析单个文件（适合于测试）

# Charon-Rudra

原仓库：<https://github.com/AeneasVerif/charon-rudra>

原 charon-rudra 相比于 rudra 的区别：
* 数据源从编译器查询 MIR 改为使用 charon 的 ULLBC 文件，这意味着没有与编译器内部 API 交互的代码（比如不需要调用 cargo，也无需驱动 rustc）
* 仅支持 UnsafeDataflow 分析，其他分析尚未实现（unsafe_destructor、send_sync_variance）
* 支持了一个额外的测例
    <details>

    <summary>详情</summary>

    原 Rudra 在 `tests/unsafe_destructor/copy_filter.rs` 测例上未检测出问题，也预期没有分析，但 charon-rudra 报告了问题

    ![](https://github.com/user-attachments/assets/b1f8e258-7488-4864-af33-77e018f09d21)

    </details>


---

我修改的仓库：<https://github.com/os-checker/charon-rudra>

改动：
* 复制 rudra 的测例（tests 目录）
* 将 charon 作为 git 子模块（根据 Cargo.lock，固定了提交）
* 添加 CI 来进行 [API 文档](https://os-checker.github.io/charon-rudra) 部署（方便查看和引用 charon 的数据结构）

## 安装

以下内容基于我修改的仓库，而不是 charon 或者原始的 charon-rudra 仓库。

```bash
cd charon/charon
# Install charon / charon-driver / generate-ml
cargo install --path . --locked

cd ../..
# Install cargo-charon-rudra
cargo install --path . --locked
```

```bash
# Linux
export LD_LIBRARY_PATH="/root/.rustup/toolchains/nightly-2021-10-21-x86_64-unknown-linux-gnu/lib:$LD_LIBRARY_PATH"

# Windows: 将下面的路径添加到系统环境变量（需要重启终端生效）
# C:\Users\Administrator\.rustup\toolchains\nightly-2024-08-11-x86_64-pc-windows-msvc\lib\rustlib\x86_64-pc-windows-msvc\lib
```

## 运行测试

```bash
# Generate insertion_sort.ullbc
charon --ullbc --no-merge-goto-chains --no-cargo --input tests/panic_safety/insertion_sort.rs

# Analyze with rudra
LOG=trace cargo-charon-rudra --file insertion_sort.ullbc
```

所生成的 ullbc 文件示例： [insertion_sort.json](https://github.com/user-attachments/files/19103374/insertion_sort.json) 文件（后缀名请自行改回 ullbc）。

`LOG=trace` 环境变量让分析过程更加清楚：

```rust
INFO | [rudra-progress] UnsafeDataflow analysis started
TRACE| [analysis::unsafe_dataflow] Analyzing fun call: core::slice::{Slice<T>}::len

TRACE| [analysis::unsafe_dataflow] Found call with unresolvable generic parts: core::slice::{Slice<T>}::len (block: 0)
TRACE| [analysis::unsafe_dataflow] Found unresolvable call to trait method: into_iter (block: 1)
TRACE| [analysis::unsafe_dataflow] Found unresolvable call to trait method: next (block: 4)
TRACE| [analysis::unsafe_dataflow] Analyzing fun call: core::ptr::read

TRACE| [analysis::unsafe_dataflow] Found potential strong lifetime bypass: core::ptr::read (block: 10)
TRACE| [analysis::unsafe_dataflow] Found strong lifetime bypass: core::ptr::read (block: 10)
TRACE| [analysis::unsafe_dataflow] Found unresolvable call to trait method: gt (block: 16)
TRACE| [analysis::unsafe_dataflow] Analyzing fun call: core::intrinsics::copy

TRACE| [analysis::unsafe_dataflow] Found potential strong lifetime bypass: core::intrinsics::copy (block: 22)
TRACE| [analysis::unsafe_dataflow] Found strong lifetime bypass: core::intrinsics::copy (block: 22)
TRACE| [analysis::unsafe_dataflow] Analyzing fun call: core::ptr::write

TRACE| [analysis::unsafe_dataflow] Found weak lifetime bypass: core::ptr::write (block: 24)
INFO | [rudra-progress] UnsafeDataflow analysis finished
Warning (UnsafeDataflow:/ReadFlow/CopyFlow/WriteFlow): Potential unsafe dataflow issue in `insertion_sort::insertion_sort_unsafe`
-> tests/panic_safety/insertion_sort.rs:10:0-22:1
// 颜色输出如下图
```

![rudra-report](https://github.com/user-attachments/assets/2b6c04b6-3c5f-4e02-a230-dca6633f2f5b)

颜色解释：
* Red: lifetime bypass of high precision
  * `Vec::set_len`
  * `Vec::from_raw_parts`
* Yellow: lifetime bypass of median precision 
  * Read Flow: `ptr::read`、 `ptr::const_ptr::_::read`
  * Copy Flow: `intrinsics::copy`、 `intrinsics::copy_nonoverlapping`
  * Write Flow: `ptr::write`、 `ptr::mut_ptr::_::write`
* Cyan: lifetime bypass of low precision
  * TRANSMUTE: `intrinsics::transmute`
  * PTR_AS_REF: `ptr::const_ptr::_::as_ref`、 `ptr::non_nul::_::as_ref`
  * SLICE_UNCHECKED
  * SLICE_FROM_RAW

此外，内部还区分 strong_bypass_spans、weak_bypass_spans、unresolvable_generic_function_spans。

## 测例支持程度

Rudra 实现了三种分析
* UnsafeDataflow：基于 taint analysis 
* UnsafeDestructor
* SendSyncVariance

我分析了 Charon-Rudra 源代码，发现它相当不完善，只是移植了数据源，但没有重建 Rudra 的算法/功能：
* 它只是把单个函数内部的调用标注了污点，但没有沿着调用往下分析
* 它只是实现了 UnsafeDataflow 分析的基础部分（算法未完全实现，但应该可以补全），而且没有实现另外两个分析（尚不知道重现 SendSyncVariance 分析是否可行）

| 编号 |            类别            | 测例 (tests 目录)                      | Rudra            | Charon-Rudra       | 对照 | New Charon-Rudra | 对照 |
|------|:--------------------------:|----------------------------------------|------------------|--------------------|:----:|------------------|------|
| 1    | <span class="TP">TP</span> | panic_safety/insertion_sort.rs         | UnsafeDataflow   | UnsafeDataflow     |      | UnsafeDataflow   |      |
| 2    | <span class="TN">TN</span> | panic_safety/order_safe_if.rs          |                  |                    |      |                  |      |
| 3    | <span class="TN">TN</span> | panic_safety/order_safe_loop.rs        |                  |                    |      |                  |      |
| 4    | <span class="TN">TN</span> | panic_safety/order_safe.rs             |                  |                    |      |                  |      |
| 5    | <span class="TP">TP</span> | panic_safety/order_unsafe_loop.rs      | UnsafeDataflow   | UnsafeDataflow     |      | UnsafeDataflow   |      |
| 7    | <span class="TP">TP</span> | panic_safety/order_unsafe_transmute.rs | UnsafeDataflow   | UnsafeDataflow     |      | UnsafeDataflow   |      |
| 6    | <span class="TP">TP</span> | panic_safety/order_unsafe.rs           | UnsafeDataflow   | UnsafeDataflow     |      | UnsafeDataflow   |      |
| 8    | <span class="FN">FN</span> | panic_safety/pointer_to_ref.rs         |                  |                    |      |                  |      |
| 9    | <span class="TP">TP</span> | panic_safety/vec_push_all.rs           | UnsafeDataflow   | UnsafeDataflow     |      | UnsafeDataflow   |      |
| 10   | <span class="TN">TN</span> | send_sync/no_generic.rs                |                  |                    |      |                  |      |
| 11   | <span class="FP">FP</span> | send_sync/okay_channel.rs              | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 12   | <span class="TN">TN</span> | send_sync/okay_imm.rs                  |                  |                    |      |                  |      |
| 13   | <span class="TN">TN</span> | send_sync/okay_negative.rs             |                  |                    |      |                  |      |
| 14   | <span class="TP">TP</span> | send_sync/okay_phantom.rs              | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 15   | <span class="TN">TN</span> | send_sync/okay_ptr_like.rs             |                  |                    |      |                  |      |
| 16   | <span class="TN">TN</span> | send_sync/okay_transitive.rs           |                  |                    |      |                  |      |
| 17   | <span class="TN">TN</span> | send_sync/okay_where.rs                |                  |                    |      |                  |      |
| 18   | <span class="FP">FP</span> | send_sync/sync_over_send_fp.rs         | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 19   | <span class="TP">TP</span> | send_sync/wild_channel.rs              | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 20   | <span class="TP">TP</span> | send_sync/wild_phantom.rs              | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 21   | <span class="TP">TP</span> | send_sync/wild_send.rs                 | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 22   | <span class="TP">TP</span> | send_sync/wild_sync.rs                 | SendSyncVariance |                    |  ❌  | SendSyncVariance |      |
| 23   | <span class="TN">TN</span> | unsafe_destructor/copy_filter.rs       |                  | ~~UnsafeDataflow~~ |      |                  |      |
| 24   | <span class="TN">TN</span> | unsafe_destructor/ffi.rs               |                  |                    |      | UnsafeDestructor | 😀   |
| 25   | <span class="FP">FP</span> | unsafe_destructor/fp1.rs               | UnsafeDestructor |                    |  ❌  | UnsafeDestructor |      |
| 26   | <span class="TN">TN</span> | unsafe_destructor/normal1.rs           |                  |                    |      |                  |      |
| 27   | <span class="TP">TP</span> | unsafe_destructor/normal2.rs           | UnsafeDestructor |                    |  ❌  | UnsafeDestructor |      |

<p style="text-align: center;">对照含义：Charon-Rudra 不支持 = ❌；Charon-Rudra 分析结果不同 = 😀</p>

|            类别            | 全称           | 含义                    | Rudra 测例数量 |
|:--------------------------:|----------------|-------------------------|----------------|
| <span class="TP">TP</span> | True Positive  | 有问题 - 且报告         | 11             |
| <span class="TN">TN</span> | True Negative  | 无问题 - 不报告         | 12             |
| <span class="FP">FP</span> | False Positive | 无问题 - 但报告（误报） | 3              |
| <span class="FN">FN</span> | False Negative | 有问题 - 不报告（漏报） | 1              |

<p style="text-align: center;">Positive = 报告；Negative = 不报告</p>


注：
1. `copy_filter.rs` 测例由我在 [dcda8f7](https://github.com/os-checker/charon-rudra/commit/dcda8f74bbba25446da936b08f75b917ce8c0f97)
提交中修复，原因是原 Charon-Rudra 对泛型尚未检查 Copy bound。
2. `tests/utility` 目录下面的测例并不在 Rudra 的测例分析范围内，因为它们缺少 meta 头。我补充了 utility 测例的情况，
   `generic_param_ctxts.rs` 和 `report_handle_macro` 是在上面表格之外的具有 SendSyncVariance 诊断的测例。

## 细节解释

[`CrateData`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/export/struct.CrateData.html

数据结构 [`CrateData`] 对应于 `charon` CLI 在 [crate](https://doc.rust-lang.org/cargo/appendix/glossary.html#crate)
上生成的 .ullbc 或者 .llbc 文件，它实际上是一个 JSON 文件，内容为：

```rust
pub struct CrateData {
    pub charon_version: String,
    pub translated: TranslatedCrate,
    pub has_errors: bool,
}
```

该 JSON 文件作为通用的数据，可被不同工具和编程语言所使用，比如 [Aeneas] 使用 OCaml 语言进行验证，但数据源通过 charon-ml 解析该 JSON 文件。

### `TranslatedCrate`

[`TranslatedCrate`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/krate/struct.TranslatedCrate.html

[`TranslatedCrate`] 是一个把 Crate 从 MIR 翻译到 LLBC 或者 LLBC 的结果，它是程序分析的基础数据。

| Field              | Type                               | 说明                                                                                                                               |
|--------------------|------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| crate_name         | `String`                           | 使用者指定的 crate 名称；可能与 rustc 报告的名称不一致                                                                             |
| real_crate_name    | `String`                           | rustc 报告的 crate 名称                                                                                                            |
| id_to_file         | `Vector<FileId, FileName>`         | 一组文件路径；根据 id 查找文件；Virtual 映射标准库路径；Local 本地路径；NotReal 适用于宏、query                                    |
| file_to_id         | `HashMap<FileName, FileId>`        | 根据文件查找 id                                                                                                                    |
| file_id_to_content | `HashMap<FileId, String>`          | 文件对应的源码；NotReal 没有源码                                                                                                   |
| all_ids            | `LinkedHashSet<AnyTransId>`        | 所有 ids；按出现的顺序                                                                                                             |
| item_names         | `HashMap<AnyTransId, Name>`        | 所有注册 id 的 [`Name`]；Name 由多个 PathElem 组成；PathElem 是 Ident 或者 Impl，并携带 `Disambiguator`（为了区分哪个 impl block） |
| type_decls         | `Vector<TypeDeclId, TypeDecl>`     | 一组类型声明；见 [`TypeDecl`] 和 [`TypeDeclKind`]                                                                                  |
| fun_decls          | `Vector<FunDeclId, FunDecl>`       | 一组函数声明；见 [`FunDecl`] 和 [`ItemKind`]                                                                                       |
| global_decls       | `Vector<GlobalDeclId, GlobalDecl>` | 一组全局变量定义；见 [`GlobalDecl`]                                                                                                |
| bodies             | `Vector<BodyId, Body>`             | 一组函数主体或者常量主体；见 [`Body`]；ULLBC （CFG) 或者 LLBC （AST）                                                              |
| trait_decls        | `Vector<TraitDeclId, TraitDecl>`   | 一组 trait 声明；见 [`TraitDecl`]                                                                                                  |
| trait_impl         | `Vector<TraitImplId, TraitImpl>`   | 一组 trait impl 声明；见 [`TraitImpl`]                                                                                             |
| ordered_decls      | `Option<Vec<DeclarationGroup>>`    | 一组顶级声明，并重新排序；见 [`DeclarationGroup`]                                                                                  |


[`Name`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/names/struct.Name.html
[`TypeDecl`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/types/enum.TypeDeclKind.html
[`TypeDeclKind`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/types/enum.TypeDeclKind.html
[`FunDecl`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/struct.FunDecl.html
[`ItemKind`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/enum.ItemKind.html
[`GlobalDecl`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/struct.GlobalDecl.html
[`Body`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/enum.Body.html
[`TraitDecl`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/struct.TraitDecl.html
[`TraitImpl`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/gast/struct.TraitImpl.html
[`DeclarationGroup`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/transform/reorder_decls/enum.DeclarationGroup.html

### `Vector<I, T>`

[`Vector<I, T>`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ids/vector/struct.Vector.html

```rust
pub struct Vector<I, T>
where
    I: Idx,
{
    vector: IndexVec<I, Option<T>>,
    real_len: usize,
}
```

其实就是 `Vec<Option<T>>`，但特殊在
* 索引是包装了 usize 的类型，因为同一个 usize 在 `Vec<A>` 和 `Vec<B>` 上含义不同，因此包装索引来防止误用
* 每个元素都是 `Option`，会保留 slot 以供将来填充数据；删除元素将把该位置的值设置为 None，但不会调整长度
* real_len 记录了非 None 元素的数量

使用者通常只保留 I，I 与 usize 大小相同，并且是 Copy 的，然后从 `Vector<I, T>` 中查找真正的数据。

### `Body`

[`Body`] 内部的主要结构如下：

```rust
pub enum Body {
    Unstructured(ullbc_ast::ExprBody), // CFG / ULLBC
    Structured(llbc_ast::ExprBody),    // AST / LLBC
}

pub struct GExprBody<T> {
    pub span: Span,
    pub arg_count: usize,
    pub locals: Vector<VarId, Var>,
    pub comments: Vec<(usize, Vec<String>)>,
    pub body: T,
}

// CFG / ULLBC
pub type ullbc_ast::ExprBody = GExprBody<BodyContents>; 
pub type BodyContents = Vector<BlockId, BlockData>
pub struct BlockData {
    pub statements: Vec<ullbc_ast::RawStatement>,
    pub terminator: ullbc_ast::Terminator,
}

// AST / LLBC
pub type llbc_ast::ExprBody = GExprBody<Block>;
pub struct Block {
    pub span: Span,
    pub statements: Vec<llbc_ast::RawStatement>,
}
```

```rust
// CFG / ULLBC
pub enum ullbc_ast::RawStatement {
    Assign(Place, Rvalue),
    Call(Call),
    FakeRead(Place),
    SetDiscriminant(Place, VariantId),
    StorageDead(VarId),
    Deinit(Place),
    Drop(Place),
    Assert(Assert),
    Nop,
    Error(String),
}
pub enum ullbc_ast::RawTerminator {
    Goto { target: BlockId, },
    Switch { discr: Operand, targets: SwitchTargets, },
    Abort(AbortKind),
    Return,
}

// AST / LLBC
pub enum llbc::RawStatement {
    Assign(Place, Rvalue),
    FakeRead(Place),
    SetDiscriminant(Place, VariantId),
    Drop(Place),
    Assert(Assert),
    Call(Call),
    Abort(AbortKind),
    Return,
    Break(usize),
    Continue(usize),
    Nop,
    Switch(Switch),
    Loop(Block),
    Error(String),
}
```

## UnsafeDataflow

如果我用白话概括 Rudra 的 UnsafeDataflow 分析的话，它是这样的：
1. 查看函数体的 CFG （控制流图）
2. 对某些固定的“危险”调用标记为“入口” (source)
3. 对 drop_in_place 和泛型函数标记为“出口” (sink)
4. 从入口点的污染状态开始，沿着执行流标记污染状态，直至函数返回
5. 如果出口具有污染状态，那么认为该函数具有 UnsafeDataflow

一些细节补充：
* 只有入口或者只有出口的函数不被认为具有 UnsafeDataflow
* 我描述的“执行流”与代码实现并不相同，代码实现中以 basic block 作为标记对象
  * basic block（缩写 bb）是 [MIR] 中的节点，只在末尾可能多个 successor，但内部每个语句只有一个 successor
  * 这样做的潜在风险是，由于标记的原因是某个函数调用语句，但标记的对象扩大到一个 bb，那么当 source 和 sink 处于同一个
    bb，但 sink 在 source 之前执行，那么这会导致一种误报（见下面的差异）

[MIR]: https://rustc-dev-guide.rust-lang.org/mir/

### 与 Rudra 的差异

Charon-Rudra 和 Rudra 在 UnsafeDataflow 在算法上没有区别，因此数据源决定分析结果的差异。

在污点分析中，如何定义 unresolvable function 决定了在哪些函数上标记 sink。

通过阅读 Rudra 在 [`analyze`] 函数的代码，可知识别 unresolvable 函数的关键是 rustc 内部的 [`Instance::resolve`] 函数的返回结果：

[`analyze`]: https://github.com/sslab-gatech/Rudra/blob/6ca6c6218e1a0126ab61f1825f5060d4d10c8cf3/src/analysis/unsafe_dataflow.rs#L233-L235
[`Instance::resolve`]: https://os-checker.github.io/Rudra/rustc/rustc_middle/ty/instance/struct.Instance.html#method.resolve

* `Ok(None)` 表示无法解析到具体的实例，例如 `fn foo<T: Debug>(t: T)`
* `Ok(Some(instance))` 表示在通过 coherence 和 type-check 之后，在单态化上下文中调用，则保证返回它
* `Err(_)` 则表示在其他地方出现了错误，而无法完成实例解析

但是，对于以下函数调用，Rudra 未能标识 f 具有 UnsafeDataflow，因为 [`Instance::resolve`] 函数对 `inner` 调用的解析结果为
`Ok(Some(_))`，这与 rustc API 自己记录的内容不符。

而 Charon 识别 unresolvable 函数的方式是查看函数携带泛型类型参数，认为 `inner` 函数是 unresolvable 的，从而标识 f 具有 UnsafeDataflow。

```rust
// Warning (UnsafeDataflow:/WriteFlow): Potential unsafe dataflow issue in `f`
fn f<T: Debug>(a: T, b: T, f: &mut Formatter<'_>) {
    unsafe { std::ptr::write(0 as _, a) }; // 👈 Write Flow
    inner(b, f); // 👈 Unresolvable Generic Function: 但 Rudra 调用 rustc API 的方式认为该泛型函数已解析实例
}

fn inner<T: Debug>(b: T, f: &mut Formatter<'_>) {
    b.fmt(f);
}
```

为了对比 Rudra 在这个例子上的奇怪之处，当你把 inner 函数手动内联，会发现 Rudra “成功”标识 f 具有 UnsafeDataflow。

```rust
fn f<T: Debug>(a: T, b: T, f: &mut Formatter<'_>) {
    unsafe { std::ptr::write(0 as _, a) }; // 👈 Write Flow
    b.fmt(f); // 👈 Unresolvable Generic Function
}
```

此外，通常不影响分析结果，但可能导致一种误报的数据差异是，Rudra 在 basic block (bb) 上切分地很细，可能一个函数调用就是一个
bb，而 Charon 把正常的执行流划分到一个 bb 内，在 switch、goto、return 之类的控制流上产生另一个 bb。不确定这是由于
Rust 编译器版本导致的差异，还是 charon 自己合并和清理导致的差异（虽然其论文确实明确提到这一点）。但最终我们会看到分析结果不同：

```rust
// Charon-Rudra 报告 UnsafeDataflow:/WriteFlow，但 Rudra 并没报告
fn f<T: Debug>(a: T, f: &mut Formatter<'_>) {
    a.fmt(f); // Sink: Unresolvable
    unsafe { std::ptr::write(0 as _, a) }; // Source: Write Flow
}
```

## UnsafeDestructor

算法相当简单和直观，Rudra 只做了两件事：
1. 查找当前 crate 中定义的所有 Drop impls
    * 主要通过 rustc 的 [`lang_items`]、 [`all_local_trait_impls`] 等 API
2. 查看这些 drop 函数是否存在 unsafe block，然后报告它
    * 主要在 HIR 层通过 rustc 的 [`Visitor::visit_block`] hook 遍历查找 unsafe block
    * 此外，Rudra 通过 [`is_foreign_item`] API 排除了 extern unsafe function

[`lang_items`]: https://os-checker.github.io/Rudra/rustc/rustc_middle/ty/context/struct.TyCtxt.html#method.lang_items
[`all_local_trait_impls`]: https://os-checker.github.io/Rudra/rustc/rustc_middle/ty/context/struct.TyCtxt.html#method.all_local_trait_impls
[`Visitor::visit_block`]: https://os-checker.github.io/Rudra/rustc/rustc_hir/intravisit/trait.Visitor.html#method.visit_block
[`is_foreign_item`]: https://os-checker.github.io/Rudra/rustc/rustc_middle/ty/context/struct.TyCtxt.html#method.is_foreign_item

Charon-Rudra 尚未实现 UnsafeDestructor 功能，但我认为可以实现类似的结果：
1. 遍历 [`trait_impls`] 筛选出 Drop impls，并由此找到它们的 drop 函数体
2. 在每个函数体内遍历基本块，找到所有函数调用
3. 如果函数调用的签名信息在 [`is_unsafe`] 字段上为 true，则报告这个 Drop impl
    * 目前函数签名还没有 is_extern，所以无法排除 extern unsafe functions

[`trait_impls`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/krate/struct.TranslatedCrate.html#structfield.trait_impls
[`is_unsafe`]: https://os-checker.github.io/charon-rudra/charon/charon_lib/ast/types/struct.FunSig.html#structfield.is_unsafe

## SendSyncVariance

首先解释这个检查名称中的 Variance。根据 Rudra 的论文，基本想法是，当且仅当 `T: Send` 满足，则 `Vec<T>: Send`
满足，这种由容器类型引发的推理关系与子类型关系的 type variance 类似，所以称 Send 和 Sync 的 trait bound 检查为 SendSyncVariance。

> The Send and Sync rules become complex as the implementation bound becomes conditional when generic types
> are involved (see Table 1). One simple example is a container type, `Vec<T>`, that is Send only if the inner type T is Send and
> is Sync only if the inner type T is Sync. The logic quickly becomes non-intuitive and error-prone for types that provide
> non-trivial sharing like Mutex and RwLock (see Table 1). Inspired by type variance in subtyping relations, we call this
> subtle relation between the Send/Sync of a generic type and the Send/Sync of the inner types the Send/Sync variance.

需要说明的是，这个 Send/Sync Variance 与 Rust 语言认为的 [Variance](https://doc.rust-lang.org/reference/subtyping.html) 
并不一致（后者只发生在生命周期泛型上）。

然后，阅读完 Rudra 的这部分的源代码之后，我发现实际的实现与论文的算法描述有些出入。

因此，无论如何我都会自己再描述 Rudra 的算法，就像描述以上两种算法一样。

Send 类型安全检测的核心算法：
1. 查找该 crate 内所有 Send impl block
2. 查看 impl 块上，ADT 的泛型类型参数是否具有 Send 约束；如果没有，则报告诊断

看起来似乎很简单和直观，但有一系列细节问题：
* 由于编译器会自动对所有泛型类型参数插入 Send 约束来严格保证并发和类型安全，不需要人为做额外的事情。因此手工实现的
  Send impl，通常意味着代码作者有自己的理由。因此严格的 Send 检测只是一种起到提醒和初筛的作用。
* Rudra 考虑到了泛型类型位于 `PhantomData` 内，从论文算法伪代码来看，当发生这种情况，不应该报告检查 Send
  的检查结果。但实际代码并未做到这一点，无论泛型类型在哪，都会进行相同的处理和报告。
* 如果泛型类型在 Send impl 块上具有 Copy 或 Sync 约束，那么视为具有 Send 约束，认为安全，因此不报告。

Sync 类型安全检测算法描述如下：
1. 查找该 crate 内所有 Sync impl block
2. 查看 impl 块的 ADT 的泛型类型，在 API（函数、方法）上具有哪些行为
    * 如果泛型类型通过 **所有权** 使用，则检查该泛型在 Sync impl 块上是否具有 Send / Copy 约束；如果没有，则报告诊断
    * 如果泛型类型通过 **借用** 来使用，则检查该泛型在 Sync impl 块上是否具有 Sync 约束；如果没有，则报告诊断

在实际检查 Sync 安全的代码中，我发现还有很多细节和可能导致错报的地方：
* 只通过 API 的签名来近似泛型的类型实际使用
  * 更精确的方式是进入函数体，并且进入该函数体的所有调用
  * 还要更精确地话，需要跟踪类型擦除（这可能非常困难）
* ADT 使用泛型类型的方式可能非常复杂，比如
  * `ADT<T>` 对 T 的使用是 `Wrapper<T>`，那么需要通过遍历才能真正确定最终是使用 `T` 还是 `&T`
  * `ADT<T>` 对 T 的使用发生在 trait bounds 上，比如 `fn f<T, U: Trait<T>>(..)`，那么通过所有权还是借用使用 T 需要查看 Trait
* 一些复杂并且我怀疑是否必要的做法
  * 单独处理 `PhantomData`，并且二元化地认为泛型 `T` 要么只在 `PhantomData<T>` 中，要么不在 `PhantomData<T>` 中，实际上这不太必要？
  * Rudra 区分构造 `Self` 函数和 `&Self` 方法，我认为也不必要。因为这是查看泛型以所有权 vs 借用方式使用的间接方式，Self 
    的使用方式和泛型使用方式可能完全无关，从而造成错报。
