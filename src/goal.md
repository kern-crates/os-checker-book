# 组件化目标

## WebUI 当前构成

> 注意：[kern-crates.github.io] 与 [os-checker.github.io] 是一样的，因此这里只以后者为例。

[os-checker.github.io] 是部署在 Github Pages 的静态网页，主体是 SPA (Single Page Application)，但也依赖
Github Pages 的 [project site] 功能。

[kern-crates.github.io]: https://kern-crates.github.io
[os-checker.github.io]: https://os-checker.github.io
[project site]: https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites

| 路径                       | 仓库维度 | Package 维度 | 说明                        | 数据生成的工具      |
|----------------------------|:--------:|:------------:|-----------------------------|---------------------|
| [`/`]                      |    ✅    |      ✅      | 诊断结果汇总表              | [os-checker]        |
| [`/file-tree`]             |          |      ✅      | 问题文件树（诊断详情）      | [os-checker]        |
| [`/charts`]                |    ✅    |              | 仓库通过情况条形图          | [os-checker]        |
| [`/target`]                |          |      ✅      | 编译目标明细表              | [os-checker]        |
| [`/workflows`]             |    ✅    |              | Github Actions 运行情况     | [plugin-github-api] |
| [`/info`]                  |          |      ✅      | Package 基础信息与测例      | [plugin-cargo]      |
| [`/docs/user/repo/ws/pkg`] |          |      ✅      | 统一自动部署的 Rustdoc 文档 | [docs]      |

[`/`]: https://os-checker.github.io
[`/file-tree`]: https://os-checker.github.io/file-tree
[`/charts`]: https://os-checker.github.io/charts
[`/target`]: https://os-checker.github.io/target
[`/workflows`]: https://os-checker.github.io/workflows
[`/info`]: https://os-checker.github.io/info
[`/docs/user/repo/ws/pkg`]: https://os-checker.github.io/docs/docs.json

[os-checker]: https://github.com/os-checker/os-checker
[plugin-github-api]: https://github.com/os-checker/plugin-github-api
[plugin-cargo]: https://github.com/os-checker/plugin-cargo
[docs]: https://github.com/os-checker/docs

[docs.json]: https://os-checker.github.io/docs/docs.json
[database]: https://github.com/os-checker/database

Rustdoc 生成的文档也是静态的，[docs] 仓库负责生成所有仓库的文档生成和整合，最终生成 [docs.json]，里面描述了
packages 对应的文档地址。

其余工具都通过自动化构建来生成和推送数据至 [database] 仓库 —— 它们是静态的 JSON 数据，任何人也可获取满足其他用途。

> 注意：从功能分离的角度看
>
> * 数据采集：上述工具，即 os-checker、plugin-github-api、plugin-cargo CLIs；
> * 数据存储：[database] 仓库；
> * 数据展示：[os-checker.github.io] （简称 WebUI）。

## `/info` 页面的组件化思路

目标：[`/info`] 页面需要取代 [`/`] 页面，作为操作系统组件库信息的门户，应该简洁美观，不宜有太复杂的交互逻辑，但应该提供链接跳转至详情页面。

Package 维度需要包含的模块如下：

* [x] 来自 Cargo.toml 的基础信息，不限于：版本号、类别、关键字、描述、作者，以及 crates 相关的信息；
* [x] 来自 [docs.json] 的统一自动部署的文档，该文档应根据各 OS 组件仓库的最新提交自动更新构建；如果最新提交导致构建失败，则允许无文档链接；
  * 当前大部分仓库无法正常编译，因此文档无法自动构建；
  * 注意：Cargo.toml 含 documentation 字段，它指向一个最新版本的文档链接（当前为人工填写）；
* [ ] 测试情况：完全基于自动化来获取和执行测试；但需要考虑实在无法模拟的机器；
  * [x] 测例名称：因为大部分仓库无法正常编译，仅有部分仓库能获取测例名称；
  * [ ] 测试结果：😐 尚未开始编写相关代码来运行测试并解析结果；
* [ ] 检查工具的诊断结果：
  * [x] 静态检查：目前有 9 个检查工具（12 个检查项），其中真正意义上检查代码内容的工具有 clippy、lockbud、mirai、rap、rudra（5
        个）；目前没有计划再集成更多静态检查工具；
  * [ ] 动态检查：主要是 Miri ([#12]) 和 Sanitizer ([#17])，它们具有最高质量的检查效果，但存在一些限制，比如需要合适的 OS 
        或平台、可执行的二进制文件；😐 尚未开始编写相关代码，它可能和测试关系紧密；

[#12]: https://github.com/os-checker/os-checker/issues/12
[#17]: https://github.com/os-checker/os-checker/issues/17

仓库维度需要包含的模块：

* [ ] Package 所有模块：因为大部分统计信息可以汇总成仓库，比如
  * 统计数字（诊断数量、测试数量）通过加和汇总；
  * 状态通过 bool 汇总：比如只要有至少存在一个诊断或未通过测试，该汇总状态为未通过；
  * 类别、关键字、作者通过取并集汇总；
* 从 Github API 获取的信息：
  * [ ] 更新时间、总提交数量、贡献人数、star/fork/watcher 数量；
  * [x] Github Workflows 运行情况（成功或失败状态，以及明细）；

## 静态分析工具的组件化

这部分并不是当前的工作重点，只是一些可能的想法。

目标：实现统一的静态分析框架，**让新的检查工具编写者专注于编写算法库**，而不是重复编写琐碎的工具逻辑。

背景：
1. Rust 静态分析工具大多采用 [MIR] 作为基础的分析对象，通过一系列算法，实现某些问题模式的检测来识别代码 bug，帮助编写无 UB 的高质量 Rust 代码。
2. 获取 MIR 的主要途径是直接与 Rust 编译器的夜间接口交互，这会导致一些麻烦：
    * 检查工具的编写者需要从复杂的、无良好文档记录的接口中找到程序分析所需的部分，有时数据来自多个地方，需要自行查找多处；
    * 夜间接口是不稳定的，需要固定一个版本号，这意味着升级版本的障碍会随着时间而增加，API 更改将导致重新查找接口；
    * 检查工具的编写者需要同时了解 Rust 语言、Rust 编译器和 Cargo，这需要额外的时间，并且在短期内无法做到；
3. 写一个实用和好用的检查工具，除了程序分析知识和上述 Rust 相关知识，还需要经验的积累和反复的迭代 —— 而检查工具编写者大多以发论文为目的，
   它们在工程能力上缺乏经验[^1]，并且只关注眼前的问题，不了解应用到实际项目上的需求。

[MIR]: https://blog.rust-lang.org/2016/04/19/MIR.html

[^1]: 比如很少有静态检查工具提供 JSON 格式输出，它们大多只是日志打印；也很少正确处理 Cargo 项目编译，尤其是条件编译；甚至其代码库，都不做到应用 fmt 和 clippy。

组件构成：

* 友好和稳定的数据来源：[charon] [^charon] 向程序验证提供精简了 MIR 和语义的、与编程语言无关的、**稳定的** JSON 数据格式，支持 CFG 和 AST 两种数据结构。 
  * 主要作者之一是 Nadrieril，他是 Rust 项目成员，资深 Rust 编译器开发者；
  * 贡献点：charon 尚不支持一些复杂类型（如关联类型）的转换；缺少示例；
* 程序分析算法库：
    * 贡献点：
        * 《[Charon: An Analysis Framework for Rust][charon-thesis]》论文中实现了 [charon-rudra]，成功利用 charon 重写 rudra，因此需要总结一下移植的步骤和经验；
        * 移植一些现有的检查工具：lockbud、rap；
        * 直接利用 charon 编写新的工具：参考上述论文有实现一个 taint-checker；
* 标准化输出格式：基于 JSON 或 SARIF 或诊断 UI 库的数据结构；
  * 贡献点：对现有检查工具的内部输出格式或相关数据类型进行总结；
* os-checker：运行并汇集多种检查工具的检查结果；提供 Github Workflows 和 Docker 进行打包、部署和应用。

[^charon]: 概括地说，charon 与 stable MIR 是一个级别的，试图通过稳定的数据结构降低与 rustc 内部的数据结构之间交互的复杂性。但 charon
的定位是向静态分析工具之类的目标一次提供所有必要的信息（所以 charon 编写了 rustc driver，从而工具编写者无需写这部分，也不必了解
Rust 编译器内部才关心的细枝末节，从而专注于编写工具本身），而 stable MIR 的目标偏向于作为编写 rustc driver 的工具（写 rustc driver 的人会更需要它）。

[charon]: https://github.com/AeneasVerif/charon
[charon-rudra]: https://github.com/AeneasVerif/charon-rudra
[charon-thesis]: https://zenodo.org/records/13983686


## 其他想法

我持开放态度，因为一开始我并不把 os-checker 局限在检查代码这件事上，而是让它有用这件事情上，所以我希望看到很多有用的信息，并且简化一些流程。

当然，最重要的是使用它，并且持续地完善。

一些基本问题尚未解决：OS 组件库的编译问题，这是长期以来的就认识到的问题，也是当前的主要障碍。完全解决它需要一个个分析组件库，把它调整正确，然后持续地保持它正确。

嗯... 如果在企业中，os-checker 会类似于质量报告和监测系统。


