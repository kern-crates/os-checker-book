# KernMiri 与 ArceOS

## 运行 ArceOS

```bash
# 下载 ArceOS 仓库
git clone https://github.com/arceos-org/arceos
cd arceos

# 通过 Docker 运行（podman 是用户级别的 docker）
podman build -t arceos .
# 让容器在后台运行，并挂载和持久化 /arceos 目录
podman run -d -v .:/arceos --name arceos-test --replace --rm arceos sleep infinity
# 连接容器（所有状态在容器结束前保留，编译产物也在 arcos 目录中保留）
podman exec -it arceos-test make run
```

这里我运行 ArceOS 的 Dockerfile 文件，方便搭建环境。如果你的机器有 sudo 权限，并且不在乎全局安装所有依赖和工具，可以从
Dockerfile 文件中复制相应的命令来安装。

你也可以交互式地在容器内运行代码，见示例脚本 [run.sh]。

[run.sh]: https://github.com/os-checker/arceos/blob/make_run/run.sh

运行 ArceOS 的最简单的方式是 `make run`，它会编译 `examples/helloworld`，并通过 QEMU 运行内核：

```text
# 打印命令
axconfig-gen configs/defconfig.toml /usr/local/cargo/registry/src/index.crates.io-1949cf8c6b5b557f/axplat-x86-pc-0.2.0/axconfig.toml  -w arch=x86_64 -w platform=x86-pc -o "/arceos/.axconfig.toml" -c "/arceos/.axconfig.toml"
    Building App: helloworld, Arch: x86_64, Platform: x86-pc, App type: rust
cargo -C examples/helloworld build -Z unstable-options --target x86_64-unknown-none --target-dir /arceos/target --release  --features "axstd/defplat axstd/log-level-warn"
rust-objcopy --binary-architecture=x86_64 examples/helloworld/helloworld_x86-pc.elf --strip-all -O binary examples/helloworld/helloworld_x86-pc.bin
    Running on qemu...
qemu-system-x86_64 -m 128M -smp 1 -machine q35 -kernel examples/helloworld/helloworld_x86-pc.elf -nographic

# 运行 helloworld & QEMU 输出
SeaBIOS (version rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org)
iPXE (http://ipxe.org) 00:02.0 CA00 PCI2.10 PnP PMM+06FD0BF0+06F30BF0 CA00
Press Ctrl-B to configure iPXE (PCI 00:02.0)...                                                                               
Booting from ROM..TSC frequency: 4000 MHz

       d8888                            .d88888b.   .d8888b.
      d88888                           d88P" "Y88b d88P  Y88b
     d88P888                           888     888 Y88b.
    d88P 888 888d888  .d8888b  .d88b.  888     888  "Y888b.
   d88P  888 888P"   d88P"    d8P  Y8b 888     888     "Y88b.
  d88P   888 888     888      88888888 888     888       "888
 d8888888888 888     Y88b.    Y8b.     Y88b. .d88P Y88b  d88P
d88P     888 888      "Y8888P  "Y8888   "Y88888P"   "Y8888P"

arch = x86_64
platform = x86-pc
target = x86_64-unknown-none
build_mode = release
log_level = warn
smp = 1

Hello, world!
```

可以看到主要运行了 4 条命令：`axconfig-gen`、`cargo build`、`rust-objcopy` 和 `qemu-system-x86_64`。

如果你单独运行前两条，在 `cargo build` 上会遇到链接错误，这是因为需要一些 axconfig 的环境变量。

因此，对于测试 ArceOS，通常更改参数来编译和执行代码，比如 `make run A=path/to/project` 编译为
x86_64-unknown-none，并在 QEMU 上运行起来。具体参数需阅读 Makefile 文件。
