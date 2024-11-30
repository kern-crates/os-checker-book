# 自动化部署

自动化部署是编写 os-checker 的主要动机，具体来说，部署内容有：

* 安装检查工具，以及检查工具所需的环境：
  * 使用者无需关注工具的安装和使用等各自问题：比如代码静态检查工具需要与 Rust 工具链版本号绑定
  * 因此使用者只需关注检查结果，而不是检查流程。
* 安装和应用 os-checker 工具集，**自动化的工作不限于**：
  1. 从 Github 下载被检查的仓库；
  1. 分析仓库的项目组织结构，生成检查命令；
  1. 执行检查命令，尽可能让检查完成，并搜集检查结果；
  1. 缓存检查结果：重新检查所未有新提交的仓库是不必要的；
  1. 维护数据仓库：检查结果以 JSON 格式推送并存储在指定的 Github 仓库；
  1. 通过静态网页展示检查结果：从统计数字到检查详情；
  1. 基于仓库最新的提交构建库的 rustdoc 文档；
  1. 搜集仓库和 package 的各种基本信息。


宗旨：运行检查工具的目的是帮助提高代码质量，而搜集基础信息的目的是帮助提高项目质量。

部署方式：
* Docker 容器：[zjpzjp/os-checker](https://hub.docker.com/repository/docker/zjpzjp/os-checker)
* Github Action：[os-checker/os-checker-action](https://github.com/os-checker/os-checker-action)
