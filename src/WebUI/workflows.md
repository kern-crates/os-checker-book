# Workflows 统计页面

地址：<https://os-checker.github.io/workflows>

## 仓库汇总和明细表

![截图_20241023224448](https://github.com/user-attachments/assets/ff96cb47-b250-467c-892a-3b57cbeea48d)

按顺序依次介绍：
1. 仓库选择/搜索框：支持多选；不支持模糊搜索；无选择和全选都表示全选；
2. 仓库计数描述：总仓库数、运行 Workflows 的次数非 0 和为 0 的仓库数量；
3. 汇总表：每行为一个仓库的 Github Action 总运行数和最近更新的情况；单击一行时，更新明细表；
4. 明细表：Workflows 的详细情况[^details]；单击一行，则弹出具体 Jobs 的运行情况。

[^details]: 目前虽然已经保存了很多 Workflows 字段数据
[(示例)](https://github.com/os-checker/database/blob/main/plugin/github-api/workflows/Starry-OS/Starry.json)，但
WebUI 只展现了最重要的一部分；如果有人想尽快看到其他字段的数据，可以和我交流。

## Jobs 与 Steps 运行情况

![截图_20241023220809](https://github.com/user-attachments/assets/e6d53d66-7e70-4c2b-96b4-b6c99fc7b71e)


按标注顺序依次介绍：

1. 仓库名称：点击跳转[^open]至该仓库的 Github 页面；
2. Workflow 名称：也就是定义在 `.github/workflows/file.yml` 中的 `name: xxx`（通常为第一行）；点击跳转至 Workflow 运行页面；
3. 标题：每个 Workflow 都附带一个与提交信息相同或者缩减的标题；
4. Jobs 统计：每个 Workflow 都直接由 jobs 组成（与 yml 中的 jobs 字段对应），因此可以计算总数；而每个 job 有完成的状态和完成的结果，从而分为 completed 和 success[^success] 维度上的计数；
5. Steps 统计：每个 job 直接由 steps 组成（与 yml 中的 steps 字段对应）；统计逻辑与 jobs 一致；
6. Job 的序号：序号从 1 开始，并且红色表示该 job 包含 failure，绿色表示该 job 不含 failure；
7. Job 的名称：来自 yml 中 jobs 中的 name 字段；点击该名称所在的块元素可以折叠 Job 详情；
8. Job 详情：即 Steps 运行情况，每行从左至右为
    * step 的完成结果图标（结果有成功、跳过、失败），并辅助颜色识别；
    * step 的编号，来自 Github（可能存在不连续）；
    * step 的名称：来自 yml 中的 jobs.steps 中的 name 或者 use 字段；
    * 开始和结束时间：已转换为本地时区（东八区，但由浏览器设置决定）；
    * 耗时：单位为秒；
9. Job 链接：点击跳转至 Github Action 的 Job 页面；
10. 查看日志的命令：通常 Job 页面有按钮可以下载日志（运行结束才可下载），但也可以通过 Github 官方 gh 命令行工具下载到本地，并利用 `less -R` 命令，在终端方便地查看带颜色的长日志。


[^open]: 以上跳转操作都打开新标签页。

[^success]: 具体来说，skipped 之类的非 failure 状态也算作了 success。
