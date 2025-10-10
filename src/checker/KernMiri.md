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

## 目标

背景：在向勇老师的操作系统专题训练课上，有同学对 KernMiri 感兴趣。

目标：将 KernMiri 集成到 os-checker。

问题：
* 可行性？：KernMiri 可以作为单独的工具吗？
  * 不特定于具体的操作系统？从星绽中剥离而应用到别的 Rust 操作系统（比如 Arceos）？
  * 或者集成进上游 Miri？
* 实现思路？
