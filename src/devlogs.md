# 开发日志

真正的日志记录在 issues 页面，那里记录了所有思考、问题以及解决结果：

* <https://github.com/os-checker/os-checker/issues>
* <https://github.com/os-checker/os-checker.github.io/issues>


每周开发总结：

- [第 1-2 周]：设计和解析配置文件；实现 fmt 和 clippy 检查
- [第 3 周]：实现主页诊断数量表
- [第 4 周]：实现问题文件树，展示所有仓库的原始检查结果
- [第 5 周]：重构 JSON 输出；把数据处理迁移到 database 仓库
- [第 6 周]：支持编译目标搜索；WebUI 添加编译目标下拉框
- [第 7 周]：集成 lockbud； 添加 Cargo 检查报告；WebUI 增加 All-Targets 统计
- [第 8 周]：成功检查 88 个 kern-crates 相关的仓库；使用 Rust 重构 jq 数据处理；实现仓库级别的问题文件树
- [第 9 周]：处理遗留出错的 18 个仓库；尝试集成 mirai
- [第 10-11 周]：WebUI 增加统计图；实现数据库缓存
- [第 12 周]：缓存和展示编译目标明细信息
- [第 13 周]：编写 WebUI 使用说明；创建 os-checker book 文档；集成 cargo-audit
- [第 14 周]：集成 RAP 和 cargo-outdated
- [第 15 周]：WebUI 增加 Github Actions 页面；给 RAP 提交重构 PR；集成 cargo-geiger；解析 package 和测例信息
- [第 16 周]：WebUI 增加 info 页面；集成 Rudra；部署在线文档
- [第 17 周]：WebUI 添加 repos 页面、实现路由查询；在 kern-crates 组织部署 OS 组件库的 rustdoc 文档和 os-checker 的 WebUI；编写 os-checker 组件化目标文档
- [第 18 周]：调整 info 页面 package 的范围；给 RAP 提交修复和新功能；更新 os-checker book 文档；查看 Rust Formal Methods WG/IG 相关资料；WebUI 细微调整
- [第 19 周]：实现 os-checker 工具集的 Docker 镜像和 Github Action Workflows；在 package 信息表上新增字段；发布所有插件 crates 到 crates.io
- [第 20 周]：重写 kern-crates/.github；WebUI 主页与 info 页面互换；集成 cargo-semver-checks；编写自动化部署文档；阅读/翻译
- [第 22 周]：初步集成 Miri；WebUI 增加 testcases 页面，展示测试和 Miri 详情
- [第 23 周]：查看全部有诊断的 143 个仓库的诊断信息，分析和记录产生诊断的原因，并修正部分仓库的 targets 配置
- [第 24 周]：os-checker 发布 v0.5.0；一些琐事和未完成的事情；年终总结/展望
- [第 25 周]：给 OS 组件库提交诊断修复；修改 kern-crates/.github 的模板
- [第 26 周]：给 OS 组件库提交诊断修复；os-checker 更新检查工具和工具链
- [第 27 周]：kern-crates 组织添加和更新仓库的策略、处理重命名的外部仓库；给 OS 组件库提交诊断修复
- [第 28 周]：编写 os-checker PPT（面向使用者）；os-checker 在 JSON 配置文件中支持指定 features
- [第 29-30 周]：初步重构诊断详情页面；PPT 演讲准备完毕
- [第 31 周]：file-tree 页面添加筛选项来交互式展示诊断详情
- [第 32 周]：修改 PPT；file-tree 页面支持路由参数查询；testcases 页面添加筛选下拉框查询
- [第 33 周]：进行 os-checker 报告；plugin-cargo 支持缓存；WebUI 改进；文档添加《工作原理》一章
- [第 34 周]: 研究 Charon、Rudra 和 Charon-Rudra；os-checker 更新检查工具及其工具链
- [第 35 周]: 给 Charon 提交重构 CLI & 修复 PRs；研究 Rudra 源代码
- [第 36 周]: 给 Charon 提交 PRs；使用 Charon 数据源复现 Rudra 的 Unsafe Destructor 分析；阅读和记录 SendSyncVariance 分析
- [第 37 周]: Charon-Rudra 使用 Charon 数据源复现 Rudra 的 Send / Sync 分析；一些计划
- [第 38 周]: 阅读 Kani 代码和文档；os-checker 小修复
- [第 39 周]: 创建 distributed-verification 仓库，从每个 `#[kani::proof]` 入口获取所有调用，计算稳定的 hash 值
- [第 40 周]: distributed-verification 测试；Rust Workshop 提纲；term-rustdoc 更新
- [第 41 周]: os-checker 许可从 MIT 改为 GPL-3.0 OR MulanPubL；distributed-verification 处理 kani-list.json 和 libcore proofs
- [第 42 周]: os-checker 与 Chain-Fox 合作；os-checker 修复和添加新功能；distributed-verification 支持新参数


[第 1-2 周]: https://github.com/os-checker/os-checker/blob/3fdf88db57403949f95c3034608481d64db80764/assets/development-logs.md
[第 3 周]: https://github.com/os-checker/os-checker/discussions/15
[第 4 周]: https://github.com/os-checker/os-checker/discussions/20
[第 5 周]: https://github.com/os-checker/os-checker/discussions/24
[第 6 周]: https://github.com/os-checker/os-checker/discussions/32
[第 7 周]: https://github.com/os-checker/os-checker/discussions/41
[第 8 周]: https://github.com/os-checker/os-checker/discussions/66
[第 9 周]: https://github.com/os-checker/os-checker/discussions/90
[第 10-11 周]: https://github.com/os-checker/os-checker/discussions/104
[第 12 周]: https://github.com/os-checker/os-checker/discussions/121
[第 13 周]: https://github.com/os-checker/os-checker/discussions/136
[第 14 周]: https://github.com/os-checker/os-checker/discussions/145
[第 15 周]: https://github.com/os-checker/os-checker/discussions/159
[第 16 周]: https://github.com/os-checker/os-checker/discussions/163
[第 17 周]: https://github.com/os-checker/os-checker/discussions/164
[第 18 周]: https://github.com/os-checker/os-checker/discussions/170
[第 19 周]: https://github.com/os-checker/os-checker/discussions/185
[第 20 周]: https://github.com/os-checker/os-checker/discussions/189
[第 22 周]: https://github.com/os-checker/os-checker/discussions/193
[第 23 周]: https://github.com/os-checker/os-checker/discussions/225
[第 24 周]: https://github.com/os-checker/os-checker/discussions/249
[第 25 周]: https://github.com/os-checker/os-checker/discussions/255
[第 26 周]: https://github.com/os-checker/os-checker/discussions/263
[第 27 周]: https://github.com/os-checker/os-checker/discussions/265
[第 28 周]: https://github.com/os-checker/os-checker/discussions/270
[第 29-30 周]: https://github.com/os-checker/os-checker/discussions/278
[第 31 周]: https://github.com/os-checker/os-checker/discussions/284
[第 32 周]: https://github.com/os-checker/os-checker/discussions/287
[第 33 周]: https://github.com/os-checker/os-checker/discussions/291
[第 34 周]: https://github.com/os-checker/os-checker/discussions/301
[第 35 周]: https://github.com/os-checker/os-checker/discussions/302
[第 36 周]: https://github.com/os-checker/os-checker/discussions/303
[第 37 周]: https://github.com/os-checker/os-checker/discussions/304
[第 38 周]: https://github.com/os-checker/os-checker/discussions/308
[第 39 周]: https://github.com/os-checker/os-checker/discussions/309
[第 40 周]: https://github.com/os-checker/os-checker/discussions/310
[第 41 周]: https://github.com/os-checker/os-checker/discussions/316
[第 42 周]: https://github.com/os-checker/os-checker/discussions/339

