# 组件化目标

## `os-checker.github.io` 当前构成

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
  * [ ] 测例名称：因为大部分仓库无法正常编译，仅有部分仓库能获取测例名称；
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


