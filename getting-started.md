## 入门指南

我们首先介绍如何配置开发环境，编译流程，以及代码规范。

### 开发环境

在Ubuntu 14.04下安装最新版的Rust：

```shell
$ curl -s https://static.rust-lang.org/rustup.sh | sudo sh
```

为[Atom](https://atom.io/)编辑器安装Rust语法高亮支持：

```shell
$ apm install language-rust
```

### 编译流程

使用[Cargo](https://crates.io/)创建`hello-world`项目：

```shell
$ cargo new hello-world --bin 
```

加上`--bin`选项是因为我们需要生成一个可执行程序，即`src/`目录下有一个`main.rs`文件。如果不加则默认生成Rust库，`src/`下有一个`lib.rs`文件。

在项目目录下，Cargo还生成了一个[TOML](https://github.com/toml-lang/toml)格式的配置文件`Cargo.toml`，其功能类似于Node.js中的`package.json`。

Cargo自动生成的`src/main.rs`代码：

```rust
fn main() {
    println!("Hello, world!");
}
```

其中`println!`是一个宏 (macro)，在Rust中宏通常是以`!`结尾。

使用Cargo编译项目非常简单：

```shell
$ cargo build
```
如果需要编译并执行，使用`cargo run`命令就可以了。如果需要发布项目，应该使用`--release`选项：

```shell
$ cargo build --release
```

为了实现打印当前时间的功能，我们需要在`Cargo.toml`中添加`time`依赖：

```toml
[dependencies]
time = "0.1.24"
```
再运行`cargo build`，Cargo会自动处理库的依赖关系。

### 代码规范
