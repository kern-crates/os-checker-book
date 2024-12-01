# 诊断数量表

![截图_20241009161254](https://github.com/user-attachments/assets/b2c47a5a-6f6d-41e2-a951-d730da800276)

地址：[os-checker.github.io](https://os-checker.github.io)

按照标注的顺序介绍功能
1. 仓库计数：pass 表示诊断数量为 0 的仓库数量；total 表示所有被检查的仓库数量；还有一个进度条，它显示 pass/total 计算得到的百分数；
2. 列选择：当表格过宽时，或者你只对某些列感兴趣，那么从这个组件中去掉不想展示的列；
3. 搜索框：在 user、repo、package 三列中搜索内容，并筛选掉不符合搜索条件的行；
4. 排序按钮：你可以单击列头进行升序、降序或取消排序；单击时按 Ctrl 或者 Meta 键，可以[多列排序](https://github.com/os-checker/os-checker.github.io/issues/6)；
5. 仓库详情链接：链接到 `https://os-checker.github.io/{user}/{repo}` 网址，目前它展示仓库级别的问题文件树[^1]；
6. 折叠按钮：当仓库只有一个 pkg 时，每行为完整的 user/repo#package 数据，因此无折叠按钮；但当仓库包含多个 pkg 时，Package 列折叠起来，并在折叠行汇总，此时你可以点击该按钮查看具体 pkg 的诊断数量。

[^1]: os-checker 的 WebUI 采用 SPA （单页应用程序），也就是说，点击链接实际在当前页面切换视图，而不刷新或者打开新的网页。


该表格列介绍
* 序号：由于默认按照 `报告数量` 降序，因此序号为 1 的仓库就是报告数量最多的仓库；最大序号 N 表示有 N 个非 0 诊断数量的仓库（pass + N = total）。
* User：Github ID，比如个人账户或者组织账户（os-checker 目前假设只对 github 仓库进行检查）。
* Repo：仓库名称。
* Package：os-checker 搜索该仓库内的所有 [Packages](https://doc.rust-lang.org/cargo/appendix/glossary.html#package) （os-checker 目前假设同一个仓库内不存在同名的 Packages）。
* 报告数量：所有检查工具报告数量的总计。
* Cargo 列直至最后：每列都是一个检查工具的检查结果数量统计（检查结果在 os-checker 中，简称诊断）。

