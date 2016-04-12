## 入门指南

我们首先介绍如何配置开发环境，编译流程，以及代码规范。

### 开发环境

在Ubuntu 14.04下安装最新版的Rust：

```shell
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh -s -- --channel=nightly
```

为[Atom](https://atom.io/)编辑器安装Rust语法高亮支持：

```shell
$ apm install language-rust
```

如果需要一个IDE，推荐使用[Tokamak](https://github.com/vertexclique/tokamak)。

### 编译流程

使用[Cargo](https://crates.io/)创建`hello-world`项目：

```shell
$ cargo new hello-world --bin 
```

加上`--bin`选项是因为我们需要生成一个可执行程序，即`src/`目录下有一个`main.rs`文件。
如果不加则默认生成Rust库，`src/`下有一个`lib.rs`文件。当一个外部包使用`extern crate`导入时，
Cargo会自动将连字符`-`转化为下划线`_`。

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
如果需要编译并执行，可以使用`cargo run`命令。如果需要发布项目，
应该使用`--release`选项开启优化：

```shell
$ cargo build --release
```
在Rust中解决依赖性相当容易，只需要在`Cargo.toml`中添加`[dependencies]`字典：

```toml
[dependencies]
semver = "0.1.19"
```
这里的`semver`库主要负责按语义化版本规范来匹配版本号：

```rust
// in src/main.rs

extern crate semver;

use semver::Version;
use semver::Identifier::{AlphaNumeric, Numeric};

fn main() {
    assert!(Version::parse("1.2.3-alpha.2") == Ok(Version {
        major: 1u64,
        minor: 2u64,
        patch: 3u64,
        pre: vec!(AlphaNumeric("alpha".to_string()), Numeric(2)),
        build: vec!(),
    }));

    println!("Versions compared successfully!");
}
```
其中函数`Ok()`来自于`std::result::Result`，通过`std::prelude`模块被预先导入。

### 代码规范

可以参考[Rust风格指南](http://doc.rust-lang.org/nightly/style/)，
这里简单强调几点：
* 使用4个空格进行缩进。
* 在单行的花括号内侧各使用一个空格。
* 不要特意在行间使用多余的空格来实现对齐。
* 避免使用块注释`/* ... */`。
* 文档注释的第一行应该是关于该部分代码的一行简短总结。
* 当结束分隔符出现在一个单独的行尾时，应该在其末尾加上逗号。
