# 组件化目标

## `os-checker.github.io` 当前构成

[os-checker.github.io] 是部署在 Github Pages 的静态网页，主体是 SPA (Single Page Application)，但也依赖
Github Pages 的 [project site] 功能。

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
