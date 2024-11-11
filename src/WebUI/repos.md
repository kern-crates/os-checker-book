# Repos

<video width="100%"  controls>
  <source src="https://github.com/user-attachments/assets/92fad475-a012-46ae-a777-7e6ed7e8c5b6" type="video/mp4">
</video>

<div style="text-align: center;">
  地址： <a href="https://os-checker.github.io/repos" target="_blank">https://os-checker.github.io/repos</a>
</div>

## 字段解释

repos 页面展示 96 个仓库的信息，它们是从 Github API 上查询的，共有 19 列数据：

1. 许可证 (License)
2. 主页链接 (Home)
3. 状态为 open 的 issues (Open Issues)：并链接到 issues 页面
4. 仓库描述 (Description)
5. 创建时间 (Created)
6. 最后推送的时间 (Updated)：对于 forked 仓库，该时间可能早于创建时间
7. 活跃天数 (Active Days)：对于非 forked 仓库，由 `最后推送时间 - 创建时间` 计算；对于 forked 仓库，由另一个更新时间计算
8. 贡献数量 (Contributions)：贡献者总计的提交数
9. 贡献者数量 (Contributors)：作者自己也为贡献者，因此为至少 1 人；由于 REST API 限制，最多只查询到 30 人
10. 仓库大小 (Sized)：小于 1KB 的数值为灰色
11. 默认分支 (Default Branch)
12. 该仓库是否是被 forked 的仓库 (Is This Forked)：大多数 kern-crates 仓库这列为 true
13. 该仓库是否被归档 （Is This Archived)：有 1 个仓库归档
14. Star 人数 (Stargazers)
15. 订阅人数 (Subscibers)：也就是 watcher 数量
16. 该仓库被 forked 的数量 (Forks)
17. 网络节点数 (Net Work)：大概是从源仓库的 forks 情况，它会组成一个网络，这个网络可以视为被贡献或者受欢迎的程度；一个查询组合是  [Contributions + Contributors + Forks + NetWork + Is This Forked](https://os-checker.github.io/repos?columns=contributions%252Ccontributors%252Cforks%252Cnetwork%252Cfork&sorts=network%253D-1) ，按照 Net Work 降序排列，可以看到第一位 embassy 有 767 个节点，第二位 arceos 有 258 个节点：

<details><summary>💡 Contributions + Contributors + Forks + NetWork + Is This Forked 查询结果</summary>
<p>

![](https://github.com/user-attachments/assets/4e868b48-4e11-481e-b877-73b39639a4e5)

</p>
</details> 


18. 仓库的讨论区链接 (Discussions)：有的话才会显示
19. 仓库主题/分类 (Topics)：只有 1 个仓库设置了

<details><summary>💡 在每个仓库主页 About 区域，就可以输入 Topics</summary>
<p>

![](https://github.com/user-attachments/assets/915730cf-c909-422e-ae2c-1969ccf4fc73)

</p>
</details>


## 路由查询

> **当更新筛选条件时，浏览器的地址栏的 URL query params 会自动更新，从而使用者可以分享这个 URL 链接；**
>
> **获得上述 URL 链接的访问者，则无需重复输入相同的查询条件，来获得使用者所看到的查询结果。**

示例：上面介绍网络节点数的组合查询链接 <https://os-checker.github.io/repos?columns=contributions%252Ccontributors%252Cforks%252Cnetwork%252Cfork&sorts=network%253D-1>

查询条件指：

* 显示列：
  * Full：显示所有列
  * Default (Slimmed)：默认的、部分列
  * 19 个列名
* 对于某列的筛选条件：
  * License 
  * Topics
* 针对所有文本列的文本筛选条件：忽略大小写，只支持严格匹配（不支持模糊匹配）
* 每个查询条件都是可以多选，并且不同的查询条件可以组合
