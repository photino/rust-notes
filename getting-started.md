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

加上`--bin`选项是因为我们需要生成一个可执行程序，即`src/`目录下有一个`main.rs`文件。
如果不加则默认生成Rust库，`src/`下有一个`lib.rs`文件。

在项目目录下，Cargo还生成了一个[TOML](https://github.com/toml-lang/toml)格式的配置文件`Cargo.toml`，
其功能类似于Node.js中的`package.json`。

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
### 代码规范

可以参考[Rust风格指南](https://github.com/rust-lang/rust-guidelines)，
这里简单强调几点：
* 使用4个空格进行缩进。
* 在单行的花括号内侧各使用一个空格。
* 不要特意在行间使用多余的空格来实现对齐。
* 避免使用块注释。
* 文档注释的第一行应该是关于该部分代码的一行简短总结。
* 当结束分隔符出现在一个单独的行尾时，应该在其末尾加上逗号。
