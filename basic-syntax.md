## 基本语法

### 变量绑定

在Rust中，变量绑定 (variable bindings) 是通过`let`关键字声明的：

```rust
let x = 5;
let mut x = 5;
let x: i32 = 5;
let (x, y) = (3, 4);
```
其中变量类型如`i32`一般都是可以省略的，因为Rust使用了类型推断 (type inference)。Rust还通过模式匹配 (pattern matching) 对变量进行解构，这允许我们同时对多个变量进行赋值。

有几点是需要特别注意的：
* 变量默认是不可改变的 (immutable)，如果需要改变一个变量的值需要显式加上`mut`关键字。
* Rust编译器默认开启属性`#[warn(unused_variable)]`，会对没有使用的变量发出警告。
* 使用未被初始化的变量是绝对禁止的，会产生一个编译时错误。

### 数值类型
