# os-checker JSON 配置格式

## 快速指定一组仓库

并采用默认的检查方式

```json
{
  "user1/repo": {},
  "user2/repo": {},
}
```

## 指定仓库的配置选项

在 `{}` 中指定仓库的配置选项

```json
{
  "user1/repo": {
    ...,
  }
}
```


### `cmds`

使用 `cmds`，自定义某个检查命令。未指定的检查命令以默认方式进行检查。

```json
{
  "user/repo": {
    // 一组自定义检查命令：键为检查工具名称，值为 bool、字符串或者字符串数组
    "cmds": {
      "fmt": "cargo fmt ...",
      "clippy": [
        "cargo clippy --target x86_64-unknown-linux-musl",
        "cargo clippy --target x86_64-unknown-linux-gnu"
      ],
      "lockbud": false
    }
  }
}
```

注意：
* 检查命令字符串暂时只支持单命令，而不支持多命令，也就是不支持 `cmd1; cmd2`。
* 对于字符串数组，每个数组元素表示一次检查。对于上面的 clippy 检查命令数组，它表示进行
  2 次检查，分别在两个目标编译架构上编译和执行检查。
* 对于 bool 值，true 表示按默认方式检查（可以无需设置为 true），而 false 表示不要这项检查。
* cmds 里定义的每项检查是覆盖性质的。
* cmds 里定义的每项检查最终都在 package 的 Cargo.toml 所在的目录中执行，因此无需 cd。

### `packages`

使用 `packages` 指定某个 package 的检查方式（不限于检查命令和其他配置）。

```json
{
  "user/repo": {
    "packages": { // 键为 package name，值为检查配置
      "package1": { // 指定 repo 中名称为 package1 的包的检查方式
        "cmds": {
          "lockbud": false
        }
      }
    }
  }
}
```


通常我们有上面几种配置就足够使用了。但为了简化编写检查命令，有一些额外的配置参数：

### `targets`

`targets` 接收一个字符串或者一个字符串数组。

```json
{
  "user/repo": {
    "targets": [
      "x86_64-unknown-linux-musl",
      "x86_64-unknown-linux-gnu"
    ]
  }
}
```

这表示 target 数组里每个元素都添加到检查命令参数上，它等价于

```json
{
  "user/repo": {
    "cmds": {
      "fmt": [
        "cargo fmt --target x86_64-unknown-linux-musl",
        "cargo fmt --target x86_64-unknown-linux-gnu"
      ],
      "clippy": [
        "cargo clippy --target x86_64-unknown-linux-musl",
        "cargo clippy --target x86_64-unknown-linux-gnu"
      ],
      ...
    }
  }
}
```

当 cmds 和 targets 同时指定时，cmds 具有更高的优先级：

```json
// 只在 x86_64-unknown-linux-gnu 上执行 clippy，
// 但对其他检查工具，依然应用指定的两个 targets。
{
  "user/repo": {
    "targets": [
      "x86_64-unknown-linux-musl",
      "x86_64-unknown-linux-gnu"
    ],
    "cmds": {
      "clippy": "cargo clippy --target x86_64-unknown-linux-gnu"
    }
  }
}
```

如前所述，`packages` 可以使用这些检查选项：

```json
{
  "user/repo": {
    "packages": {
      "pkg1": {
        "targets": "riscv64gc-unknown-none-elf"
      },
      "pkg2": {
        "targets": [
          "riscv64gc-unknown-none-elf",
          "x86_64-unknown-none"
        ]
      }
    }
  }
}
```

### `no_install_targets`

```json
{
  "user/repo": {
    "no_install_targets": "x86_64-unknown-uefi"
  }
}
```

从 targets 安装列表中移除某些 targets。值为字符串或者字符串数组。

当 `rustup` 不支持安装该 targets 时，那么应该使用此字段明确排除不要安装它们，因为
os-checker 会对所有检测到的 targets 执行 `rustup target install`。

实际案例见 [#96](https://github.com/os-checker/os-checker/issues/96#issuecomment-2373513203)。


### `setup`

使用 `setup` 选项来设置编译环境，它在下载仓库之前运行，并且只会执行一次。

它接收字符串或者字符串数组。

目前只适用于 repo，并且在仓库的根目录下执行，不作用于 `packages`。

```json
{
  "user/repo": {
    "setup": "make setup"
  }
}

{
  "user/repo": {
    "setup": [
      "apt install ...",
      "curl ..."
    ]
  }
}
```

Tracking: [#81](https://github.com/os-checker/os-checker/issues/81)

提示：在 docker 容器中，不需要使用 sudo，否则出现报错。

### `features`

这个暂时还在考虑是否要支持。我初步的想法是，只适用于 `packages`：

```json
{
  "user/repo": {
    "packages": {
      "pkg1": {
        "features": "feat1,feat2"
      }
    }
  }
}
```

它和 `targets` 类似，表示附加到每个检查命令上，所以等价于

```json
{
  "user/repo": {
    "packages": {
      "pkg1": {
        "cmds": {
          "fmt": "cargo fmt --features=feat1,feat2",
          "clippy": "cargo clippy --features=feat1,feat2"
        }
      }
    }
  }
}
```

它和 `targets` 选项一起设置的效果：

```json
{
  "user/repo": {
    "targets": ["t1", "t2"],
    "packages": {
      "p1": { "features": "xxx" },
      "p2": { "features": "yyy" }
    }
  }
}

// 等价于写
{
  "user/repo": {
    "targets": ["t1", "t2"],
    "packages": {
      "p1": {
        "cmds": {
          "fmt": [
            "cargo fmt --target t1 -features xxx",
            "cargo fmt --target t2 -features xxx"
          ],
          "clippy": [...] // 类似 fmt 的参数
        }
      },
      "p2": {
        "cmds": {
          "fmt": [
            "cargo fmt --target t1 -features yyy",
            "cargo fmt --target t2 -features yyy"
          ],
          ...
        }
      }
    }
  }
}
```

当 `features` 接收数组，与 `targets` 类似，表示附加每个元素到每个检查命令：

```json
{
  "user/repo": {
    "packages": {
      "pkg1": {
        "features": ["feat1", "feat2"]
      }
    }
  }
}

// 等价于
{
  "user/repo": {
    "packages": {
      "pkg1": {
        "cmds": {
          "fmt": [ // 注意，不是 --features feat1,feat2
            "cargo fmt --features feat1",
            "cargo fmt --features feat2"
          ],
          ...
        }
      }
    }
  }
}
```

Tracking: [#241](https://github.com/os-checker/os-checker/issues/241)

### `meta.skip_pkg_dir_globs`

如果 package 所处的目录符合 glob 条件，那么不检查这个 package。示例：

```json
{
  "AsyncModules/embassy-priority": {
    "meta": {
      "skip_pkg_dir_globs": [
        "*test*", "*bench*", "*example*", "*template*", "*doc*"
      ]
    }
  }
}
```

注意：该字段作用于每级目录，控制是否进入该目录，os-checker 通过递归查找来匹配这些 globs。

也就是说，`*test*` 适用于
* `test/` 目录
* `tests/` 目录
* `prefix-test/` 目录
* `prefix-tests/` 目录
* `test-posfix/` 目录
* `tests-posfix/` 目录
* `path/to/test/` 目录
* `path/to/tests/` 目录
* 等等

### `meta.only_pkg_dir_globs`

只检查符合条件的包，可与 `meta.skip_pkg_dir_globs` 同时使用。比如

```json
{
  "paritytech/polkadot-sdk": {
    "meta": {
      "only_pkg_dir_globs": [ "polkadot/*" ],
      "skip_pkg_dir_globs": [ "*test*", "*bench*", "*example*", "*template*", "*doc*" ]
    }
  }
}
```

表示只检查 polkadot 目录中所有包，但排除（不进入）测试、示例之类的目录。

对于大型项目（一个仓库至少几十个包的情况），最好指定 only_pkg_dir_globs，并且鼓励同时指定
skip_pkg_dir_globs，以减少检查时间。

注意：skip_pkg_dir_globs 具有更高的优先级，意味着

```json
{
  "paritytech/polkadot-sdk": {
    "meta": {
      "only_pkg_dir_globs": [ "*sub*" ],
      "skip_pkg_dir_globs": [ "*test*", "*bench*", "*example*", "*template*", "*doc*" ]
    }
  }
}
```

如果 test 目录存在符合 `*sub*` 的路径，该路径不会出现，因为首先排除了 test 目录。

### `meta.target_env` 和 `env`

`1`: 仓库级别的环境变量作用于所有成员库的所有检查命令

```json
{
  "user/repo": { "env": {"ENV1": "val1", "ENV2": "val2"} }
}
```


`3`: 与 target 交互，则放到 `meta.target_env`


```json
"meta": {
  "target_env": {
    "target1": { "ENV1": "val" },
    "target2": { "ENV2": "val" }
  }
}
```

当环境变量重复时，3 覆盖 1。

环境变量被放入命令的缓存键中。

见 [PR#244](https://github.com/os-checker/os-checker/pull/244)。

### `meta.rerun`

重新运行某个仓库的检查。对某个仓库中进行重新下载，但使用缓存来检查。

```json
{
  "user/repo": {
    "meta": { "rerun": true }
  }
}
```

这主要用于临时使用，保持配置文件，但局部重新检查某个仓库。

见 [PR#330](https://github.com/os-checker/os-checker/pull/330)。

如果强制重新检查，需要使用如下两个环境变量：

```bash
# force downloading repos to run check
export FORCE_REPO_CHECK=true
# force running checks after downloading repos
export FORCE_RUN_CHECK=true
```

注意：该 `meta.rerun` 的行为是仅仅是某个仓库上设置 `FORCE_REPO_CHECK=true`，与 `FORCE_RUN_CHECK` 
无关 —— 应该可以配合 `FORCE_RUN_CHECK=true` 让这个仓库重新检查，并保持其他仓库使用缓存。

### `meta.use_last_cache`

如果一个仓库已经存在数据库，无论它最后一次是否完成检查，那么直接使用那个检查结果，并跳过 github 访问。

这通常用于快速生成已有的检查结果，因此提供了命令行参数 `os-checker run --use-last-cache` 控制对所有配置仓库启用。

在配置文件中，对单独一个启用它，使用

```json
{
  "user/repo": {
    "meta": { "use_last_cache": true }
  }
}
```

见 [PR#351](https://github.com/os-checker/os-checker/pull/351)。

### `meta.run_all_checkers`

主要用于禁用所有检查：

```json
{
  "user/repo": {
    "meta": { "run_all_checkers": false }
}
```

此外，它可与 `cmds.checker` 配合使用，表示只进行某中检查：

```json
{
  "user/repo": {
    "meta": { "run_all_checkers": false },
    "cmds": { "lockbud": true }
  }
}
```

注意：os-checker 默认开启所有检查，因此该字段的值默认为 true。

见 [PR#356](https://github.com/os-checker/os-checker/pull/356)。
