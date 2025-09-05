# 安全属性标注

## 背景

当前审计 Rust 不安全代码，依赖于安全注释 (safety comments)。该注释以人类可读的文本形式放置在函数的定义和使用处。

但这存在以下问题：
* 不安全调用处的安全注释不全：审计人员无法确认不安全函数的安全要求是否全部满足以及如何满足
* 不安全函数声明的安全注释不全：当封装不安全操作的时候，其内部的安全要求被委托至不安全函数，代码作者可能因此遗漏某些安全要求

此外，复旦大学的徐辉老师团队对此正在展开研究，见论文《[Annotating and Auditing the Safety Properties of Unsafe Rust](https://arxiv.org/abs/2504.21312)》。

## 基本思路

我们提供几个[工具宏](https://doc.rust-lang.org/reference/attributes.html#tool-attributes)，将当前的安全注释变成结构化的安全属性标签：
* 在定义处，将每个安全要求归纳成一个标识符，并携带可选的参数来对应适用的对象
  * 原本的安全注释（或者安全文档）将附加到相应的标签上
  * 另一种做法是，我们提取了一组通用的安全属性，这些属性在定义的时候具有标识符、参数、含义等一系列信息，使用者只需要应用这些安全属性到不安全函数上
  * 由于安全标签和安全注释在文本解释上完全相同，我们只需要保留安全标签，统一生成安全文档
* 在调用处，必须完整地列举安全标签，并且说明这些标签代表的安全要求如何满足
  * 有静态分析工具对定义和声明的安全标签进行分析和检查
  * 如果调用不安全函数的时候，缺少函数所需的安全标签，那么该工具将发出诊断，以提醒缺失的安全属性应该检查是否满足

<details>

<summary>示例</summary>

![](https://github.com/user-attachments/assets/48ec3740-5a49-4afd-b17d-64bfc8b7e8e3)

</details>


## 适用的项目

我编写了安全属性标注的检查工具 [safety-tool] 和 [Pre-RFC: Safety Property System]，而徐老师团队的同学正将这个工具用于以下项目

* Std | Verify-Rust-Std
  * 目前通过验证标准库项目将安全属性放入标准库，并联合 [RAPx] 进行检查和验证；
  * 我们也向 Rust 官方提出了一个略微不同的 [RFC: Safety Tags]，将上述思路集成到标准库、 Clippy 和 Rust-Analyer，以提供一流的安全属性支持
* Rust for Linux
  * Rfl 去年提出了一个 [Rust Safety Standard](https://kangrejos.com/2024/Rust%20Safety%20Standard.pdf)，表达了对标准化安全注释的要求，而我们的思路符合这一要求
  * 我们有 [issue#3](https://github.com/Artisan-Lab/tag-std/issues/3) 跟踪这一点，并且它被 Rfl 人员引用
* Asterinas
  * 六月研讨会前一天，与星绽开发人员（田洪亮博士等）、徐老师的学生们面对面地讨论了在星绽中如何实验这项工作
  * 在星绽仓库打开了 [discussion](https://github.com/asterinas/asterinas/discussions/2336)
  * 田博士邀请我们参加每周五下午的星绽社区会议：第一次参加的时候，我分享了思路；如果对标注有任何疑问，可以在周会上即时答疑

[safety-tool] 的 CI 都成功初步集成到了这些项目。而同学们也在各自负责的项目在逐一分析安全注释，对不安全函数进行标注。

[safety-tool]: https://github.com/Artisan-Lab/tag-std
[sp-core]: https://github.com/Artisan-Lab/tag-std/blob/main/safety-tool/assets/sp-core.toml
[RFC: Safety Tags]: https://github.com/rust-lang/rfcs/pull/3842
[Pre-RFC: Safety Property System]: https://os-checker.github.io/slides/Safety-Property-System

## 与 RAPx 集成验证实现

[safety-tool] 只有声明安全标签和检查标签缺失的功能，由程序员保证安全标签的含义与函数实现相匹配。

徐老师的饶子豪博士生正将安全属性标签进一步分析，在 [RAPx] 中进行一些验证的工作，保证标签和实现相符。

[RAPx]: https://github.com/Artisan-Lab/RAPx

## 与 Clippy 和 RA 集成

我向 Rust 社区提出了 [RFC: Safety Tags]，希望在 Clippy 中支持安全标签的声明和检查，在 Rust-Analyer 
中支持安全标签的查看、跳转和补全，并对标准库的公开的不安全函数应用安全标签。

![](https://github.com/user-attachments/assets/9956fa78-afe6-45e5-83af-0d6c1947cf29)

该 RFC 引发了一些社区的关注和热烈的讨论，但尚未得到 Rust 团队的明确支持。
