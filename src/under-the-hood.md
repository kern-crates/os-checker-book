# 工作原理

<style>
img {
  display: block; margin: auto;
  width: 70%; height: auto;
}
</style>

此部分内容是对 os-checker 工具集设计思路的总体概述。

基于 2025-03-01 分享的 20 分钟版本的 [PPT](https://docs.qq.com/slide/DTGlJeFh3S1NDQWlo)。

## 原则

os-checker 的核心想法是 **关注代码库的信息和质量**，并具有以下原则：
1. 自动化：所有数据的生成过程不应该人为干预，使用者从指定一些配置开始，最终只在 WebUI 上关注结果。
2. 流程化：所展示的数据都应该通过执行统一的步骤而得到，使用者可以执行相同的流程重现结果。具体来说：
    * 检查命令应该非常明确，在相同的环境中运行它们可得到相同的诊断；
    * 基础信息的来源应该非常明确，通过访问信息来源，看到相同的信息。
    * 流程化需要记录额外的信息数据，而不仅仅是一个结果；
    * 使用者以 os-checker 的结果为主，他们不必执行流程；
    * 透明的流程有助于发现在某个环节上存在不足或者错误。

## 静态检查流程图


![](https://github.com/user-attachments/assets/6651a227-51c5-4fa6-931e-5c44d5d9ea60)

静态检查由 [os-checker CLI](https://github.com/os-checker/os-checker) 生成，关键点：
1. 输入是一个或者多个配置文件，比如 `--config base.json --config override.json`
    * 如果同一个仓库存在于多个配置文件，那么只使用右侧文件中的该仓库的配置
    * 配置文件说明见 [JSON 配置文件](./config.md) 章节
2. 输出是一些 JSON 文件
    * JSON 的格式结构见 [此文档](https://github.com/os-checker/os-checker/blob/379b4c5f3884500f536ea00ffc0672d2af054861/assets/JSON-data-format.md)
    * 实现见 [`os_checker_types::JsonOutput`](https://github.com/os-checker/os-checker/blob/379b4c5f3884500f536ea00ffc0672d2af054861/os-checker-types/src/lib.rs#L33)
    * 由于这些格式在 2024 年 8 月份的时候设计，我以为只需要 JSONs 就够用，但 9 月份利用嵌入式键值数据库实现了缓存，这意味
        * 实际上这些 JSON 文件其实可以不必存在，而应该统一以二进制格式存在于数据库中
        * 如果将来把 JSON 输出划入数据库，那么格式结构可能也需要适当地调整，因为这个 JSON 格式结构是自描述的，键值数据库可以通过外部索引方便地建立多级映射关系来统一存储和管理
3. 整个过程是单线程执行的，这意味着
    * 串行检查仓库
    * 串行执行检查
    * 曾经尝试过使用多线程/线程池并行检查，但遇到很多问题，比如
        * Rust 编译器已经利用多核 CPU 并行编译，并行检查可能让检查更加缓慢（因为切换进程/线程/争用资源开销？）
        * 并发调用 rustup 存在 [bug](https://github.com/rust-lang/rustup/issues/988)（cargo 代理了 rustup，因此调用 cargo 意味着调用了 rustup）
4. 按检查工具顺序执行：检查时间 vs 磁盘空间
    * 尽可能利用 Cargo 缓存/增量编译来节省时间
    * 每种检查工具执行完后，调用 `cargo clean` 命令清除局部缓存 —— Github Runner 只有 10-20G 的磁盘空间

## 静态检查缓存设计

必要性：
1. 完整检查时间太长/甚至不可行：检查 100+ 个仓库需要好几个小时，尤其随着检查工具增加，全部重新进行完整的检查时间可能 6 超过小时，这是 Github Runner 的运行时间上限。
2. 不必要重复检查：如果仓库没有新的提交，配置文件也没有变化，那么只需使用上一次的检查结果而无需运行任何检查

缓存实现思路：
1. 核心：在仓库特定提交和分支上，若一个包的检查命令存在，则查看数据库中是否存在诊断结果
    * 如果结果不存在，意味着该检查从未运行，因此执行该检查命令，并缓存其诊断缓存
    * 如果结果存在，直接使用该诊断
2. 快速路径：通过 Github API 查询仓库的 HEAD SHA 值和默认分支[^default-branch]，并与该仓库的配置一起作为查询键，如果数据库存在，结果则直接使用诊断结果，无需下载仓库，也无需执行检查。

[^default-branch]: 目前只支持在默认分支上检查，但需要考虑默认分支被修改，那么需要重新检查。

![](https://github.com/user-attachments/assets/aca45a99-bf3f-4fd1-a0a4-164684c55098)

## 静态网页 (WebUI)

### 数据源

![](https://github.com/user-attachments/assets/7fa74824-cd63-4aa6-a151-cf656b616671)

关键点：
1. os-checker CLI 负责生成静态检查诊断，以及相关的统计数据
2. plugin-cargo 负责生成测例和 package 相关的信息
    * 测例：名称、运行的通过结果、Miri 的动态检查结果
    * package：crates.io 版本发布情况、来自 Cargo.toml 的基础信息
3. plugin-github-api 负责生成仓库的基本信息和 CI 的详细信息
4. docs 生成 HEAD commit 的 rustdoc 文档，这是必要的，因为
    * 由于各种原因，不是所有的组件库都发布到了 crates.io 上面，docs.rs 没有组件库的文档
    * 即便发布到了 crates.io 上面，docs.rs 的文档是基于版本的，而不是基于最新提交的

### 页面构成

> 注意：[kern-crates.github.io] 与 [os-checker.github.io] 是一样的，因此这里只以后者为例。

[os-checker.github.io] 是部署在 Github Pages 的静态网页，主体是 SPA (Single Page Application)，但也依赖
Github Pages 的 [project site] 功能。

[kern-crates.github.io]: https://kern-crates.github.io
[os-checker.github.io]: https://os-checker.github.io
[project site]: https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites

| 路径                       | 仓库维度 | Package 维度 | 说明                        | 数据生成的工具      |
|----------------------------|:--------:|:------------:|-----------------------------|---------------------|
| [`/`]                  |          |      ✅      | Package 基础信息            | [plugin-cargo]      |
| [`/repos`]                 |    ✅    |              | 仓库情况（基于 Github API） | [plugin-github-api] |
| [`/testcases`]             |          |              | 测例与 Miri 情况            | [plugin-cargo]      |
| [`/diagnostics`]           |    ✅    |      ✅      | 诊断结果汇总表              | [os-checker]        |
| [`/file-tree`]             |          |      ✅      | 问题文件树（诊断详情）      | [os-checker]        |
| [`/charts`]                |    ✅    |              | 仓库 0 诊断情况条形图       | [os-checker]        |
| [`/target`]                |          |      ✅      | 编译目标明细表              | [os-checker]        |
| [`/workflows`]             |    ✅    |              | Github Actions 运行情况     | [plugin-github-api] |
| [`/docs/user/repo/ws/pkg`] |          |      ✅      | 统一自动部署的 Rustdoc 文档 | [plugin-docs]       |

[`/`]: https://os-checker.github.io
[`/repos`]: https://os-checker.github.io/repos
[`/testcases`]: https://os-checker.github.io/testcases
[`/diagnostics`]: https://os-checker.github.io/diagnostics
[`/file-tree`]: https://os-checker.github.io/file-tree
[`/charts`]: https://os-checker.github.io/charts
[`/target`]: https://os-checker.github.io/target
[`/workflows`]: https://os-checker.github.io/workflows
[`/docs/user/repo/ws/pkg`]: https://os-checker.github.io/docs/docs.json

[os-checker]: https://github.com/os-checker/os-checker
[plugin-github-api]: https://github.com/os-checker/plugin-github-api
[plugin-cargo]: https://github.com/os-checker/plugin-cargo
[plugin-docs]: https://github.com/os-checker/docs

[docs.json]: https://os-checker.github.io/docs/docs.json
[database]: https://github.com/os-checker/database

Rustdoc 生成的文档也是静态的，[plugin-docs] 仓库负责生成所有仓库的文档生成和整合，最终生成 [docs.json]，里面描述了
packages 对应的文档地址。

其余工具都通过自动化构建来生成和推送数据至 [database] 仓库 —— 它们是静态的 JSON 数据，任何人也可获取满足其他用途。

> 注意：从功能分离的角度看
>
> * 数据采集：上述工具，即 os-checker、plugin-github-api、plugin-cargo CLIs；
> * 数据存储：[database] 仓库；
> * 数据展示：[os-checker.github.io] （简称 WebUI）。

<details>

<summary>`/` 页面的组件化思路</summary>


（已完成）

目标：[`/`] 页面，作为操作系统组件库信息的门户，应该简洁美观，不宜有太复杂的交互逻辑，但应该提供链接跳转至详情页面。

Package 维度需要包含的模块如下：

* [x] 来自 Cargo.toml 的基础信息，不限于：版本号、类别、关键字、描述、作者，以及 crates 相关的信息；
* [x] 来自 [docs.json] 的统一自动部署的文档，该文档应根据各 OS 组件仓库的最新提交自动更新构建；如果最新提交导致构建失败，则允许无文档链接；
  * 当前大部分仓库无法正常编译，因此文档无法自动构建；
  * 注意：Cargo.toml 含 documentation 字段，它指向一个最新版本的文档链接（当前为人工填写）；
* [ ] 测试情况：完全基于自动化来获取和执行测试；
  * 基于 `cargo test` 的测试
      * [x] 测例名称：因为大部分仓库无法正常编译，仅有部分仓库能获取测例名称；
      * [x] 测试结果：运行测试并解析测试结果与耗时；
  * 自定义测试命令（系统测试）：直接操作机器的库（boot、时钟、驱动、页表等）通常需要自己的测试脚本；
* [x] 检查工具的诊断结果：
  * [x] 静态检查：目前有 9 个检查工具（12 个检查项），其中真正意义上检查代码内容的工具有 clippy、lockbud、mirai、rap、rudra（5
        个）；目前没有计划再集成更多静态检查工具；
  * [x] 动态检查：主要是 Miri ([#12]) 和 Sanitizer[^Sanitizer] ([#17])，它们具有最高质量的检查效果，但存在一些限制，比如需要合适的 OS 
        或平台、可执行的二进制文件；😐 尚未开始编写相关代码，它可能和测试关系紧密；

[#12]: https://github.com/os-checker/os-checker/issues/12
[#17]: https://github.com/os-checker/os-checker/issues/17

[^Sanitizer]: 尚未实现，暂缓。

仓库维度需要包含的模块：

* [x] Package 所有模块：因为大部分统计信息可以汇总成仓库，比如
  * 统计数字（诊断数量、测试数量）通过加和汇总；
  * 状态通过 bool 汇总：比如只要有至少存在一个诊断或未通过测试，该汇总状态为未通过；
  * 类别、关键字、作者通过取并集汇总；
* 从 Github API 获取的信息：
  * [x] 更新时间、总提交数量、贡献人数、star/fork/watcher 数量；
  * [x] Github Workflows 运行情况（成功或失败状态，以及明细）；

</details>

## 检查工具生态总览

![](https://github.com/user-attachments/assets/30c2f9d0-596a-4392-80f5-32ce2d9485b0)

加粗的工具已经集成到 os-checker 中。

这里有 50 多个链接，在 PPT 中可以点击访问；或者直接在 Github 上搜索它们。

注意：不是所有工具都会集成到 os-checker，因为
* 有些工具需要修改源代码，比如某些测试工具、形式化验证工具，这比较麻烦
* 有些链接并不是工具，比如 Rust 语言/项目中的链接是工作组、某些工具只针对二进制文件、顶层应用只需要独自发挥作用

## 静态检查工具的组件化

![](https://github.com/user-attachments/assets/258f62f4-ad51-421a-b842-9f3d564298d7)

![](https://github.com/user-attachments/assets/f64a814e-73f7-41d9-8e3b-26db52337f26)

补充：
* Rudra 作者欢迎社区维护新的 Rudra，[见 Rudra#55](https://github.com/sslab-gatech/Rudra/pull/55#issuecomment-1938232472)
* 标准输出格式 SARIF 可以与 Github Security/Action 结合：当新的提交改动触发了诊断，PR 中可以看到这些诊断，[见示例](https://github.com/os-checker/os-checker/discussions/11#discussioncomment-12259645)
