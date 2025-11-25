# 安全属性标注

安全属性 = Safety properties (SPs)

## 星绽

### 属性分类

和晨昊的分类 [Asterinas-safety-properties.md](https://github.com/Artisan-Lab/tag-std/blob/main/Asterinas-safety-properties.md) 有些区别。

标准库的 SP：以变量（指针、数值等）自身的属性为主。
* 指针：对齐、有效、别名等
* 数值：比较（大小、相等）、范围等

操作系统的 SP：以逻辑属性为主。
* 函数调用关系属性：
  * CallOnce
  * (Not)PostToFunc
  * (Not)PreToFunc
  * 封装 (≈[CallThisInstead](https://github.com/Artisan-Lab/tag-asterinas/blob/5d36406b85c4e07605af7bc9293b7fd414a6dddd/ostd/src/cpu/local/cell.rs#L86-L92)) ❔
* 资源（所有权、访问权限）属性：
  * OwnedResource（拥有资源）
  * RefHeld（必须被引用）
  * RefUnheld（必须没被引用）
  * MutAccess（独占访问）
  * Unaccessed（不可访问）
  * Forgetten（[泄露](https://smallcultfollowing.com/babysteps/blog/2025/10/21/move-destruct-leak/)）
  * OriginateFrom（资源来源）
* 上下文属性（时间段）：
  * 链接期间（Section）
  * Context(after, before): "This function should be executed after {after} before {before}."
    * CPU 初始化阶段：Context("BSP starts", "any AP starts")、 Context("boot starts", "boot ends")
  * 内核态（KernelMemorySafe）
  * 用户态（UserSpace）
  * 中断期间（未有单独的属性）preempt 相关？
  * 硬件使用逻辑：
    * NonModifying（寄存器）
  * 锁使用逻辑：（附加问题：如何有助于防止并发问题❔）
    * LockHeld(val): "{val} should have already held the lock."
    * Sync(val): "If the type of {val} is not `Sync`, the caller must ensure that {val} is not accessed concurrently." ❔ 
* 有效性：
  * Valid(val): "{val} should be valid."
  * ValidAccessAddr(addr, access): "{addr} should be valid for {access}."
  * ValidBaseAddr(addr, hardware):  "{addr} should be a valid base address of {hardware}."
  * ValidInstanceAddr(addr, type): "{addr} should point to a valid instance of {type}."
* 与其他属性的关联：
  * ReferTo(func): "This function should meet the safety requirement of {func}."
  * ReferToSP(func, sp_target)：指明某一项安全要求 ❔
  * Ref(entity): 双向引用
* 补充：
  * PanicSafety: 无 panic 应该是一种安全属性。内核应该处理所有异常，而不应该崩溃。[panic 示例](https://github.com/Artisan-Lab/tag-asterinas/blob/5d36406b85c4e07605af7bc9293b7fd414a6dddd/ostd/src/mm/page_table/cursor/mod.rs#L514-L521)

问题：
* 归纳成参数 vs 单独的属性名（泛用/参数化 vs 特指/可读性）
* NotPostToFunc 是晨昊根据 API 自行添加的。需要删除这些标注；最好利用某种数据流分析，以方便审查代码影响。
* 暂不考虑：子属性（compound SP，比如 ValidPtr 是很多安全要求的集合）、条件属性等更复杂的表达式构成的属性。

safety-tool 功能增强：
* 函数调用关系属性检测：该类属性占比很大、比较容易实现和演示。
* （低优先级）LSP 感知属性标注情况：提供缺失的属性补全，而不是提供所有属性补全。

### RFC 大纲

> 基于提交给 Rust 社区的 RFC：<https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md>
>
> 星绽 RFC 模板：<https://asterinas.github.io/book/rfcs/rfc-template.html>
>
> **注意：紧密结合星绽的特点和需求。**

Summary：


Motivation：前两个是重点
* [Safety Invariants: Forgotten by Authors, Hidden from Reviewers](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#safety-invariants-forgotten-by-authors-hidden-from-reviewers) ⭐
* [Safety Invariants Have No Semver](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#safety-invariants-have-no-semver) ⭐
  * ostd 也是版本语义控制的
  * 但更实际的作用：任何一处安全要求发生变化，我们的工具能够根据改动造成的影响，提供不安全要求影响评估信息（变化的内容、波及的函数声明和调用），
    意味着修改不安全要求依然保证审计的完整性（不会留下过时的不安全代码，不会遗漏应该修正的不安全代码）。
* [Granular Unsafe: How Small Is Too Small?](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#safety-invariants-have-no-semver)
* [Formal Contracts, Casual Burden](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#formal-contracts-casual-burden)

Design：(Explain how it works, how it interacts with other parts of the system, and any new APIs or concepts being introduced.)
* [Guide-level explanation](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#guide-level-explanation) ⭐
* [Reference-level explanation](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#reference-level-explanation) （不重要）
* 星绽的最大特点：ostd 是唯一存放不安全代码的库。因此关注于
  * 安全标注的语法
  * 我们的工具能够检查什么问题
  * 如何检查这些问题
  * 集成到星绽的步骤：调研（总体介绍星绽的安全属性）、初期（哪些属性先应用）、评估效果（我们会关注安全属性方面的星绽 PR 审查）、迭代

Drawbacks, Alternatives, and Unknown：
* [Drawbacks](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#drawbacks)
* [Rationale and alternatives](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#rationale-and-alternatives)
* [Unresolved questions](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#unresolved-questions) 不太相关

Prior Art and References：
* [Prior art](https://github.com/Artisan-Lab/rfcs/blob/safety-tags/text/0000-safety-tags.md#prior-art)
* 引用 tag-std 仓库和论文
