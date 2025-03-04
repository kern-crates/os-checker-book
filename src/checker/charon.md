# Charon

## 工具介绍

仓库：<https://github.com/AeneasVerif/charon>

论文：《 [Charon: An Analysis Framework for Rust][paper] 》

[paper]: https://arxiv.org/abs/2410.18042

Charon 框架
* 目标：Charon 旨在为 Rust 分析工具提供一个通用的、稳定的接口，隐藏底层复杂性，提供一个适合分析的抽象语法树（AST）。

设计与实现：
* Charon 通过与 Cargo 和 Rust 编译器的深度集成，提供了一个完整的、装饰过的 AST，简化了工具开发者的任务。
* Charon 提供了两种表示形式：ULLBC（无结构低级借用演算）和 LLBC（低级借用演算），分别用于控制流图（CFG）和抽象语法树（AST）的分析。
* Charon 还支持跨语言的输出格式（如 JSON 和 OCaml），便于不同语言的工具开发者使用。

[案例研究](https://zenodo.org/records/13983686)：通过多个案例研究验证了 Charon 的适用性，包括：
* 常量时间分析：检测加密代码中的侧信道漏洞。
* Rudra 分析器移植（[charon-rudra]）：将现有的 Rust 静态分析工具 Rudra 重实现到 Charon 上，验证了其有效性。
* [Aeneas] 验证框架：通过将 Rust 程序转换为纯函数式程序进行验证。
* [Eurydice] 编译器：将 Rust 编译为 C 语言，以便在不支持 Rust 的环境中使用。

[charon-rudra]: https://github.com/AeneasVerif/charon-rudra
[Aeneas]: https://github.com/AeneasVerif/aeneas
[Eurydice]: https://github.com/AeneasVerif/eurydice

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
