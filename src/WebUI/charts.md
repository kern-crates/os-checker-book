# 统计图

![截图_20241009175251](https://github.com/user-attachments/assets/4d0f2771-17d5-4179-8857-710dc933a921)

地址：[os-checker.github.io/charts](https://os-checker.github.io/charts)

目前仅为仓库诊断情况计数，也就是主页进度条在编译目标维度上的可视化：
* 横坐标为仓库数量：具体划分为 pass （诊断数量为 0） 和 defect （诊断数量不为 0）两种；每个条形图末端有一个标签，显示了合计和通过率。
* 纵坐标为编译目标：目前有 18 个编译目标（除 All-Targets），数量最多的为 x86_64-unknown-linux-gnu，主要因为它是默认值。

该图是可交互的，可以不显示某类指标（单击图例），支持鼠标悬浮信息。


