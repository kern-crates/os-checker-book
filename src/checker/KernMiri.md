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

