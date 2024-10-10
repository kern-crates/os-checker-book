# os-checker

对 Rust 编写的代码运行一系列检查工具，并对结果进行报告和统计，用以督促和提高代码库的质量。

虽然工具名称暗示与操作系统相关，但仅仅是以它为背景而起的名字。也就是说， os-checker 适用于任何 Rust 代码库。

os-checker 由以下部分组成：

| 工具     | 仓库                          | 功能                                            |
|----------|-------------------------------|-------------------------------------------------|
| CLI      | [os-checker]                  | 对目标仓库运行一系列检查工具，最终输出检查结果  |
| WebUI    | [os-checker.github.io][WebUI] | 通过网页应用呈现检查结果，并部署到 Github Pages |
| database | [database]                    | 存储检查结果数据                                |
| 文档     | [book]                        | 介绍 os-checker                                 |

[os-checker]: https://github.com/os-checker/os-checker
[WebUI]: https://github.com/os-checker/os-checker.github.io
[os-checker.github.io]: https://os-checker.github.io
[database]: https://github.com/os-checker/database
[book]: https://github.com/os-checker/book

os-checker 目前设计为检查 Github 的代码，并且采用 Github Action 进行自动化检查[^ga]。

[^ga]: 但尚未推出自己的 Github Action Workflow。

# 如何使用？

**对于 kern-crates 组织来说，只需改动 [repos.json] 文件，等待 CI 检查完，最后前往 [os-checker.github.io] 网页查看检查结果。**

[repos.json]: https://github.com/os-checker/os-checker/blob/main/assets/repos-ui.json

如果你只想知道自己仓库的原始的检查输出，可查看 `os-checker.github.io/user/repo`，其中 user 为 Github 用户/组织名，repo 为仓库名。

如果你对 os-checker 有任何想法，无论关于 WebUI（界面需求） 还是 CLI（集成的检查工具），都可以发起 [讨论][discussions]。

# 如何添加配置？配置示例？

```json
{
  // 描述你的 github 用户名和仓库（可以从浏览器网址栏截取 github.com 后的两部分）
  // 注意：目前只对默认分支进行检查
  "arceos-org/arceos": {},

  // 你的仓库中所有 packages 只针对 riscv64 平台

  "rel4team/sel4_cspace": {
    "targets": "riscv64gc-unknown-none-elf"
  }
}
```

你可以了解该配置文件的 [具体设计](./config.md)。

如果你的仓库比较奇特，需要特定的检查前提，不限于需要额外的 features/targets/cfg-flags/rustc-flags 等等编译条件，那么可以到 
[discussions] 和我进行讨论，以帮助你编写检查配置。

[discussions]: https://github.com/os-checker/os-checker/discussions

