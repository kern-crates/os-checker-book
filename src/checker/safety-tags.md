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

