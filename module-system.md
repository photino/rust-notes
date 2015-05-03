## 模块系统

Rust有两个与模块系统相关的独特术语：`crate`和`module`，
其中`crate`与其它语言中的 libary 或者 package 作用一样。
每个`crate`都有一个隐藏的根模块，在根模块下可以定义一个子模块树，
其路径采用`::`作为分隔符。

在Rust中，`crate`是由条目 (item) 构成的，每个条目都可以有附加的属性 (attribute)。
多个条目就是通过模块 (module) 组织在一起的。


### 定义模块

使用`mod`关键字定义我们的模块：

```rust
// in src/lib.rs

mod chinese {
    mod greetings {
    
    }

    mod farewells {
    
    }
}

mod english {
    mod greetings {
    
    }

    mod farewells {
    
    }
}
```
定义了四个子模块`chinese::{greetings, farewells}`和`english::{greetings, farewells}`。
模块默认是私有的，为了使其公有需要`pub`关键字。

实践中更好的组织方式是将一个`crate`分拆到多个文件：

```rust
// in src/lib.rs

pub mod chinese;

pub mod english;
```
这两句声明告诉Rust查看`src/chinese.rs`和`src/english.rs`，
或者`src/chinese/mod.rs`和`src/english/mod.rs`。
先添加一些函数：

```rust
// in src/chinese/greetings.rs

pub fn hello() -> String {
    "你好！".to_string()
}
```

```rust
// in src/chinese/farewells.rs

pub fn goodbye() -> String {
    "再见！".to_string()
}
```

```rust
// in src/english/greetings.rs

pub fn hello() -> String {
    "Hello!".to_string()
}
```

```rust
// in src/english/farewells.rs

pub fn goodbye() -> String {
    "Goodbye!".to_string()
}
```
函数默认也是私有的，为了后面的使用我们需要`pub`关键字使其成为共有。

### 导入 crate

为了使用我们前面创建的名为`phrases`的`crate`，需要先声明导入

```rust
// in src/main.rs

extern crate phrases;

fn main() {
    println!("Hello in Chinese: {}", phrases::chinese::greetings::hello());
}
```

Rust还有一个`use`关键字，允许我们导入`crate`中的条目到当前的作用域内：

```rust
// in src/main.rs

extern crate phrases;

use phrases::chinese::greetings;
use phrases::chinese::farewells::goodbye;

fn main() {
    println!("Hello in Chinese: {}", greetings::hello());
    println!("Goodbye in Chinese: {}", goodbye());
}
```
但是，我们不推荐直接导入函数，这样更容易导致命名空间冲突，只导入模块是更好的做法。
如果要导入来自同一模块的多个条目，可以使用大括号简写：

```rust
use phrases::chinese::{greetings, farewells};
```
如果是导入全部，可以使用通配符`*`，一般不推荐这么做。

有时我们需要将外部`crate`里面的函数导入到另一个模块内，
这时可以使用`pub use`来提供扩展接口而不映射代码层级结构。
比如

```rust
// in src/english/mod.rs

pub use self::greetings::hello;
pub use self::farewells::goodbye;

mod greetings;

mod farewells;
```
其中`pub use`声明将函数带入了当前模块中，
使得我们现在有了`phrases::english::hello()`函数和`phrases::english::goodbye()`函数，
即使它们的定义位于`phrases::english::greetings::hello()`
和`phrases::english::farewells::goodbye()`中，
内部代码的组织结构不能反映我们的扩展接口。

默认情况下，`use`声明表示从根`crate`开始的绝对路径。
此外，我们可以使用`use self::`表示相对于当前模块的位置，
`use super::`表示当前位置的上一级，以`::`为前缀的路径表示根`crate`路径。

```rust
use foo::baz::foobaz; // foo is at the root of the crate

mod foo {
    use foo::bar::foobar; // foo is at crate root
    use self::baz::foobaz; // self refers to module 'foo'

    pub mod bar {
        pub fn foobar() { }
    }

    pub mod baz {
        use super::bar::foobar; // super refers to module 'foo'
        pub fn foobaz() { }
    }
}
```

