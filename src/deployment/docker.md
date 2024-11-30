# Docker 容器

```bash
docker run -v /check:/check --env-file os-checker.env -e GH_TOKEN=... zjpzjp/os-checker /check/run.sh
```

完整示例： [docker_pull.yml] ，含 Github Pages 部署配置。

[docker_pull.yml]: https://github.com/os-checker/dockerfiles/blob/835cb0b43c839a2d1c4b0762caa461792cbf49b4/.github/workflows/docker_pull.yaml

[`zjpzjp/os-checker`] 将自动指定 latest 标签。

[`zjpzjp/os-checker`]: https://hub.docker.com/r/zjpzjp/os-checker/tags

其余参数拆分讲解：

## `-e GH_TOKEN`

建议将 `GH_TOKEN` 环境变量通过命令行传递，而不是写在文件中。

在 Github Action 中，通常写为 `-e GH_TOKEN=${{ secrets.GH_TOKEN }}`。你需要在仓库或者组织中设置一个名为
`GH_TOKEN` 的 secret，它需要有权限推送到 `DATABASE_REPO`，并访问所有仓库的信息；因此 `github.token` 权限可能不足够。

## `--env-file`

指定容器运行时的环境变量，由运行脚本读取。

`os-checker.env` 示例：

```bash
CHECK_DIR=/check
OS_CHECKER_CONFIGS=repos.json
# OS_CHECKER_CONFIGS=repos-default.json repos-ui.json

# os-checker related
OS_CHECKER_RUST_TOOLCHAIN=nightly-2024-11-21
RUST_LOG=info

GIT_AUTHOR=zjp-CN[bot]
GIT_EMAIL=zjp-CN[bot]@users.noreply.github.com

DATABASE_REPO=os-checker/test-database

# Web UI related
BASE_URL=/dockerfiles/
# This is read by both by plugin-docs and WebUI
DOCS_URL=https://os-checker.github.io/dockerfiles/docs
```

说明：

| env                         | 示例             | 含义                                                                     |
|-----------------------------|------------------|--------------------------------------------------------------------------|
| `CHECK_DIR`                 | `/check`         | 容器启动时映射的卷 `-v /check:/check`，用于存放输入配置文件和输出结果    |
| `OS_CHECKER_CONFIGS`        | `repos.json`[^1] | os-checker CLI 的配置文件；以单个空格分隔；应在容器外存放到 `$CHECK_DIR` |
| `OS_CHECKER_RUST_TOOLCHAIN` | `nightly`        | 默认的工具链；必须是夜间工具链；推荐带具体日期，以便利用检查结果缓存     |
| `RUST_LOG`                  | `info`           | os-checker 所有工具的日志等级；不区分大小写；推荐 `info` 或者 `error`    |
| `GIT_AUTHOR`                | `zjp-CN[bot]`    | 自动推送到数据仓库的提交者（建议不要设置为自己的账户，它会计算提交数）   |
| `GIT_EMAIL`                 | `email@...`      | 上述提交者的邮箱（建议不要设置为自己账户的邮箱，它会计算提交数）         |
| `DATABASE_REPO`             | `user/repo`      | 被自动推送的 Github 仓库，专门存放 WebUI 使用的 JSON 数据                |
| `BASE_URL`                  | `/deploy_repo/`  | 部署的仓库名，由于作为 URL 的一部分，必须携带首尾的 `/`                  |
| `DOCS_URL`                  |                  | 见下文                                                                   |

[^1]: 所有 `--env-file` 指定的文件中，环境变量的值不应包含首尾的引号，因为它会视为字符串的一部分，比如
`OS_CHECKER_CONFIGS=repos.json` 会查找 `repos.json` 的文件，而 `OS_CHECKER_CONFIGS="repos.json"` 会查找
`"repos.json"` 文件（这基本上是错误的） —— 即便是空格分隔，也无需首尾的引号 `OS_CHECKER_CONFIGS=default.json repos.json`。

`DOCS_URL` 指向自动生成的 rustdoc 库文档的路径前缀，具有如下特点：

* `DOCS_URL` 一般为 `https://{user}.github.io/{deploy_repo}/docs`，需要核对两个信息：user 和 deploy_repo
  * user 为 Github Pages 部署的账户名称（个人或者组织）
  * deploy_repo 是部署 Github Pages 的仓库名称
* deploy_repo 和 database repo 可以一样也可以不一样，部署的仓库会嵌入到 URL 中；末尾的 `/docs` 由 os-checker 工具识别，是必须的
* os-checker 会自动生成在该 URL 路径下中生成 docs.json，该文件包含 `DOCS_URL` 前缀的文档访问路径：
<details>

<summary>
访问 `DOCS_URL/docs.json`，将获得类似下面的
JSON 结果。示例：<code>DOCS_URL=https://os-checker.github.io/dockerfiles/docs</code>

</summary>


```json
{
  "os-checker": {
    "os-checker-test-suite": {
      "buildrs-failure": null,
      "lockbud-checks-this": null,
      "mirai-checks-this": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/mirai_checks_this",
      "os-checker-test-suite": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/os_checker_test_suite",
      "rap-checks-this": null,
      "rudra-checks-this": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/rudra_checks_this",
      "ws-lockbud-checks-this1": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_lockbud_checks_this1",
      "ws-lockbud-checks-this2": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_lockbud_checks_this2",
      "ws-rap-checks-test-and-example": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_rap_checks_test_and_example",
      "ws-rap-checks-this1": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_rap_checks_this1",
      "ws-rap-checks-this2": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_rap_checks_this2",
      "ws-rap-checks-this_with-build-script": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_rap_checks_this_with_build_script",
      "ws-rap-checks-this_with-feature-gated": "https://os-checker.github.io/dockerfiles/docs/os-checker/os-checker-test-suite/workspace2/ws_rap_checks_this_with_feature_gated"
    }
  }
}
```

</details>

## `-v`

示例： `/check:/check`；通用：`/host/dir:/${CHECK_DIR}`，其中 `CHECK_DIR` 来自 `--env-file`。

挂载目录，也就是在本机和容器之间共享目录。该目录在本机应包含
* 所有 os-checker 配置文件，与 `OS_CHECKER_CONFIGS` 应保持一致：
  * `OS_CHECKER_CONFIGS=a.json b.json` 意味着此目录应有 a.json 和 b.json 
* [`run.sh`] 运行脚本，与最后的参数 `/check/run.sh` 对应，你可以通过以下命令获得

```bash
wget https://raw.githubusercontent.com/os-checker/dockerfiles/refs/heads/main/run.sh
```

[`run.sh`]: https://github.com/os-checker/dockerfiles/blob/main/run.sh

