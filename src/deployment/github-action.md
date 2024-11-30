# Github Action

地址：[os-checker/os-checker-action](https://github.com/os-checker/os-checker-action)

示例：

```yaml
- name: Prepare config JSONs
  run: echo '{"os-checker/os-checker-test-suite":{}}' > repo.json

- uses: os-checker/os-checker-action@HEAD
  with:
    configs: repo.json
    database_repo: os-checker/test-database
    base_url: /os-checker-action/
    docs_url: https://os-checker.github.io/os-checker-action/docs
    gh_token: ${{ secrets.GH_TOKEN }}

- name: Upload pages artifacts
  uses: actions/upload-pages-artifact@v3
  with:
    path: /tmp/check/dist
```

完整文件：[test.yml](https://github.com/os-checker/os-checker-action/blob/main/.github/workflows/test.yml)。

字段解释见 [action.yml](https://github.com/os-checker/os-checker-action/blob/main/action.yml)，填写格式见
[此处](./docker.html#--env-file)。它们之间几乎没有区别，只是大小写转换了，以及个别字段有略微调整，比如

* Github Action 尚不支持定义 `CHECK_DIR`，因为它是固定的 `/tmp/check`（将来可能调整）
* `OS_CHECKER_CONFIGS` 在 Github Action 中为 `configs`（将来可能调整）

此外，Docker 和 Github Action 在安装软件上不一样，原则是
* Docker 尽量通过源码安装或者 `cargo binstall` 安装检查工具和环境，镜像直接提供基础环境，但 os-checker
  工具集是运行时源码安装的。
* Action 尽量通过预编译压缩包快速安装所有软件，尽可能减少源码安装，这是为了搭建环境的时间和快速应用。
