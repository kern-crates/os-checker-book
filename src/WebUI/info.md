# Info

<video width="100%"  controls>
  <source src="https://github.com/user-attachments/assets/c7119225-ff27-45d5-959b-c5de0f463489" type="video/mp4">
</video>

<div style="text-align: center;">
  地址： <a href="https://os-checker.github.io/info" target="_blank">https://os-checker.github.io/info</a>
</div>

## Package 信息

info 页面为 package 维度信息（大部分来自 Cargo.toml），共 15 列：

* 版本号 (version)
* 是否为 lib
* 是否为 bin
* 直接依赖的数量
* 测例数量（由于很多仓库无法直接编译，仅有部分仓库内的 packages 才有这个信息）：
  * 当所有测例运行成功时，该值为绿色；
  * 当存在至少一个运行失败的测例时，该值为红色；
* tests/examples/benches 文件数量
* 文档链接（来自 documentation 字段）
* 最新文档链接（来自 os-checker 组织对所有仓库统一生成的在线 rustdoc 文档，基于仓库的最新提交；部分仓库无法编译而暂时无法生成文档；详细列表见 https://os-checker.github.io/docs/docs.json ）
* 主页链接
* 类别
* 关键字
* 描述
* 作者


## 测例信息

单击一行，则弹出该 package 的文本信息和测例信息。

测例信息包含：
* 测例来自哪个二进制文件 (Binary Name)
* 该二进制文件对应于哪种 crate (Kind)
  * lib 表示来自 lib.rs
  * bin 表示来自 bin crate，比如 main.rs 
  * test 表示来自 tests 文件夹内的测试文件
* 测例名称 (Test Case)
* 测例运行结果 (Status)
* 测试运行耗时，单位为毫秒 (Duration ms)

示例：

![](https://github.com/user-attachments/assets/ec7d25b7-b1d1-4730-9dce-c127e039f3ad)


<details>

<summary>带 Categories 和 Keywords 标签的测例框</summary>

![](https://github.com/user-attachments/assets/a6f293f3-b911-4256-9afc-a1eae84bab3f)

</details>

## 路由查询

> **当更新筛选条件时，浏览器的地址栏的 URL query params 会自动更新，从而使用者可以分享这个 URL 链接；**
>
> **获得上述 URL 链接的访问者，则无需重复输入相同的查询条件，来获得使用者所看到的查询结果。**

查询条件指：

* 显示列：按照列宽和相似的类型划分
  * 所有数值型列 (digits)
  * 所有链接列 (links)：文档、主页
  * 所有文本信息列(texts)：类别、关键字、描述、作者
* 类别
* 关键字
* 作者
* crate kind：
  * 对于每项，表示该 package 是否存在某 crate kind
  * 对于多选，表示该 package 是否同时存在这些 crate kinds
* 每个查询条件都是可以多选，并且不同的查询条件可以组合；注意：有些多选或者组合并不表示并集，比如
  * 同时选择多个查询条件，那么取的是它们的交集：也就是同时满足这个组合
  * 在单个查询条件内多选，大多数情况下取的是并集（例外是 crate kind，多选时取的是交集）
  * 这个交集和并集的取舍是主观的，如果对此有什么想法，欢迎和我交流
