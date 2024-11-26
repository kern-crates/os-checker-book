# 开发日志

真正的日志记录在 issues 页面，那里记录了所有思考、问题以及解决结果：

* <https://github.com/os-checker/os-checker/issues>
* <https://github.com/os-checker/os-checker.github.io/issues>


每周开发总结：

- [第 1-2 周](https://github.com/os-checker/os-checker/blob/3fdf88db57403949f95c3034608481d64db80764/assets/development-logs.md)：设计和解析配置文件；实现 fmt 和 clippy 检查
- [第 3 周](https://github.com/os-checker/os-checker/discussions/15)：实现主页诊断数量表
- [第 4 周](https://github.com/os-checker/os-checker/discussions/20)：实现问题文件树，展示所有仓库的原始检查结果
- [第 5 周](https://github.com/os-checker/os-checker/discussions/24)：重构 JSON 输出；把数据处理迁移到 database 仓库
- [第 6 周](https://github.com/os-checker/os-checker/discussions/32)：支持编译目标搜索；WebUI 添加编译目标下拉框
- [第 7 周](https://github.com/os-checker/os-checker/discussions/41)：集成 lockbud； 添加 Cargo 检查报告；WebUI 增加 All-Targets 统计
- [第 8 周](https://github.com/os-checker/os-checker/discussions/66)：成功检查 88 个 kern-crates 相关的仓库；使用 Rust 重构 jq 数据处理；实现仓库级别的问题文件树
- [第 9 周](https://github.com/os-checker/os-checker/discussions/90)：处理遗留出错的 18 个仓库；尝试集成 mirai
- [第 10-11 周](https://github.com/os-checker/os-checker/discussions/104)：WebUI 增加统计图；实现数据库缓存
- [第 12 周](https://github.com/os-checker/os-checker/discussions/121)：缓存和展示编译目标明细信息
- [第 13 周](https://github.com/os-checker/os-checker/discussions/136)：编写 WebUI 使用说明；创建 os-checker book 文档；集成 cargo-audit
- [第 14 周](https://github.com/os-checker/os-checker/discussions/145)：集成 RAP 和 cargo-outdated
- [第 15 周](https://github.com/os-checker/os-checker/discussions/159)：WebUI 增加 Github Actions 页面；给 RAP 提交重构 PR；集成 cargo-geiger；解析 package 和测例信息
- [第 16 周](https://github.com/os-checker/os-checker/discussions/163)：WebUI 增加 info 页面；集成 Rudra；部署在线文档
- [第 17 周](https://github.com/os-checker/os-checker/discussions/164)：WebUI 添加 repos 页面、实现路由查询；在 kern-crates 组织部署 OS 组件库的 rustdoc 文档和 os-checker 的 WebUI；编写 os-checker 组件化目标文档
- [第 18 周](https://github.com/os-checker/os-checker/discussions/170)：调整 info 页面 package 的范围；给 RAP 提交修复和新功能；更新 os-checker book 文档；查看 Rust Formal Methods WG/IG 相关资料；WebUI 细微调整
- [第 19 周]：实现 os-checker 工具集的 Docker 镜像和 Github Action Workflows；在 package 信息表上新增字段；发布所有插件 crates 到 crates.io

[第 19 周]: https://github.com/os-checker/os-checker/discussions/185
