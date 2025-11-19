# Priroda - MIR Debugger

## 运行示例

[Priroda] 的 Rust 工具链是四年前的 nightly-2021-03-13，虽然可以正常使用，但无法编译新的项目。

[Priroda]: https://github.com/oli-obk/priroda

一个基本的 docker 运行 priroda 流程可以描述为：

```dockerfile
FROM ubuntu:latest

RUN apt update \
  && apt install -y git curl build-essential pkg-config libgraphviz-dev

RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain none -y && \
  echo "export CARGO_TERM_COLOR=always" >> ~/.bashrc

# COPY . /priroda
RUN git clone https://github.com/oli-obk/priroda /priroda

WORKDIR /priroda

RUN bash -lc "\
  rustup show \
  && rustup component add miri \
  && cargo install xargo --locked \
  && cargo miri setup \
"

ENV MIRI_SYSROOT=/root/.cache/miri/HOST
```

```bash
podman build -t priroda .
podman run -d --name priroda --replace --rm priroda sleep infinity
podman exec -it priroda bash
```

在 x86-64 上运行如下命令可得到

```rust
$ cargo run example.rs
     Running `target/debug/priroda example.rs`
🔧 Configured for development.
    => address: localhost
    => port: 54321
    => log: normal
    => workers: 64
    => secret key: generated
    => limits: forms = 32KiB
    => keep-alive: 5s
    => read timeout: 5s
    => write timeout: 5s
    => tls: disabled
🛰  Mounting /:
    => GET /please_panic (please_panic)
    => GET /resources/<path..> (resources)
    => GET /step_count (step_count)
🛰  Mounting /:
    => GET / (index)
    => GET /frame/<frame> (frame)
    => GET /frame/<frame> [42] (frame_invalid)
    => GET /ptr/<alloc_id>/<offset> (ptr)
    => GET /reverse_ptr/<ptr> (reverse_ptr)
🛰  Mounting /breakpoints:
    => GET /breakpoints/add_here (add_here)
    => GET /breakpoints/add/<path..> (add)
    => GET /breakpoints/remove/<path..> (remove)
    => GET /breakpoints/remove_all (remove_all)
🛰  Mounting /step:
    => GET /step/restart (restart)
    => GET /step/single (single)
    => GET /step/single_back (single_back)
    => GET /step/next (next)
    => GET /step/return (return_)
    => GET /step/continue (continue_)
🛰  Mounting /watch:
    => GET /watch/show (show)
    => GET /watch/continue_and_show (continue_and_show)
    => GET /watch/add/<id> (add)
📡 Fairings:
    => 1 launch: Priroda, because code has no privacy rights
🚀 Rocket has launched from http://localhost:54321
warning: unused variable: `u`
  --> example.rs:48:9
   |
48 |     let u = SomeUnion { a: true };
   |         ^ help: if this is intentional, prefix it with an underscore: `_u`
   |
   = note: `#[warn(unused_variables)]` on by default

error: internal compiler error: compiler/rustc_metadata/src/rmeta/decoder.rs:1157:17: get_optimized_mir: missing MIR for `DefId(1:6671 ~ std[e
5ee]::rt::lang_start_internal)`

thread 'rustc' panicked at 'Box<Any>', /rustc/b3e19a221e63dcffdef87e12eadf1f36a8b90295/library/std/src/panic.rs:59:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: aborting due to previous error; 1 warning emitted


============== Miri crashed - restart try 1 ==============

warning: unused variable: `u`
  --> example.rs:48:9
   |
48 |     let u = SomeUnion { a: true };
   |         ^ help: if this is intentional, prefix it with an underscore: `_u`
   |
   = note: `#[warn(unused_variables)]` on by default

GET /step_count:
    => Matched: GET /step_count (step_count)
    => Outcome: Success
    => Response succeeded.
GET /step_count:
    => Matched: GET /step_count (step_count)
    => Outcome: Success
    => Response succeeded.
```

在 aarch64 机器上运行示例将得到如下错误：

```rust
$ cargo run example.rs
error[E0308]: mismatched types
  --> /root/.cargo/git/checkouts/cgraph-7a547510dd5167be/5e6a6d4/src/lib.rs:90:67
   |
90 |             let res = gvRenderFilename(gvc, self.0, svg.as_ptr(), file_path.as_ptr() as *const i8);
   |                                                                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected `u8`, found `i8`
   |
   = note: expected raw pointer `*const u8`
              found raw pointer `*const i8`
```

在指出的源码中修改指针类型，继续编译，就可以得到类似的终端输出结果。

然后打开 [localhost:54321](http://localhost:54321/)，浏览器中将看到 MIR 调试界面：

![](https://github.com/user-attachments/assets/d32ee940-89fd-4ff2-b2e1-d0412092180d)

但点击 Step、Next 之类的按钮，并不能工作，可能因为后端 Miri 找不到 DefId 而 panic （见上面的输出）。

![](https://github.com/user-attachments/assets/b7706ed4-4220-4880-9ea2-fd20e912e5b8)

