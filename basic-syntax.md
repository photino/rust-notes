## 基本语法

### 变量绑定

在Rust中，变量绑定 (variable bindings) 是通过`let`关键字声明的：

```rust
let x = 5;
let mut x = 5;
let x: i32 = 5;
let (x, y) = (3, 4);
```
其中变量类型如`i32`一般都是可以省略的，因为Rust使用了类型推断 (type inference)。
Rust还通过模式匹配 (pattern matching) 对变量进行解构，这允许我们同时对多个变量进行赋值。

有几点是需要特别注意的：
* 变量默认是不可改变的 (immutable)，如果需要改变一个变量的值需要显式加上`mut`关键字。
* Rust编译器默认开启属性`#[warn(unused_variable)]`，会对没有使用的变量发出警告。
* 使用未被初始化的变量是绝对禁止的，会产生一个编译时错误。

### 基本类型

Rust内置的基本类型 (primitive types) 有以下几类：

* 布尔类型：有两个值`true`和`false`。
* 字符类型：表示单个Unicode字符，存储为4个字节。
* 数值类型：分为有符号整数 (`i8`, `i16`, `i32`, `i64`, `isize`)、
无符号整数 (`u8`, `u16`, `u32`, `u64`, `usize`) 以及浮点数 (`f32`, `f64`)。
* 字符串类型：分为字符串切片`&str`和堆分配字符串`String`，其中字符串切片是静态分配的，
有固定的大小，并且不可变，而堆分配字符串是可变的。
* 数组：具有固定大小，并且元素都是同种类型，可表示为`[T; N]`。
* 切片：引用一个数组的部分数据并且不需要拷贝，可表示为`&[T]`。
* 元组：具有固定大小的有序列表，每个元素都有自己的类型，通过解构来获得每个元素的值。
* 结构体：所包含的每个元素都有一个名称称为域，并且每个结构体也都有一个名称。
* 元组结构体：由元组和结构体混合构成，元组结构体有名称，但是它的域没有。
* 枚举：对于一个指定的名称有一组可替代的值，其中子数据结构可以存储也可以不存储数据。
* 函数：具有函数类型的变量实质上是一个函数指针。
* 元类型：即`()`，只有一个值`()`。

```rust
// boolean type
let t = true;
let f: bool = false;

// char type
let c = 'c';

// numeric types
let x = 42;
let y: u32 = 123_456;
let z: f64 = 1.234_56E+5;
let bin = 0b1111_0000;
let oct = 0o7320_1546;
let hex = 0xf23a_b049;

// string types
let str = "Hello, world!";
let mut string = str.to_string();

// arrays and slices
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4];
let mut ten_zeros = [0; 10];

// tuples
let tuple: (&i32, &str) = (50, "hello");
let (fifty, hello) = tuple;

// structs
struct Point {
  x: i32,
  y: i32,
}
let origin = Point { x: 0, y: 0 };

// tuple structs
struct Color(u8, u8, u8);
let android_green = Color(0xa4, 0xc6, 0x39);
let (red, green, blue) = android_green;

// newtype: a tuple struct with only one element
struct Inches(i32);
let length = Inches(10);
let Inches(integer_length) = length;

// enums
enum Character {
    Digit(i32),
    Other,
}
let ten = Character::Digit(10);

// functions
fn foo(x: i32) -> i32 { x }
let bar: fn(i32) -> i32 = foo;
```

有几点是需要特别注意的：
* 数值类型可以使用`_`分隔符来增加可读性。
* Rust还支持单字节字符`b'H'`以及单字节字符串`b"Hello"`，仅限制于ASCII字符。 
此外，还可以使用`r#"..."#`标记来表示原始字符串，不需要对特殊字符进行转义。
* 使用`&`符号将`String`类型转换成`&str`类型很廉价， 
但是使用`to_string()`方法将`&str`转换到`String`类型涉及到分配内存，
除非很有必要否则不要这么做。
* 元组可以使用`==`运算符来判断是否相同，但是结构体和枚举不可以。

### 函数

要声明一个函数，需要使用关键字`fn`，后面跟上函数名，比如
```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```
其中函数参数的类型不能省略，可以有多个参数，但是最多只能返回一个值，
提前返回使用`return`关键字。

Rust有一个特殊语法适用于分叉函数 (diverging function)，它不返回值：

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```
其中`panic!`是一个宏，使当前执行线程崩溃并打印给定信息。返回类型`!`可用作任何类型：

```rust
let x: i32 = diverges();
let y: String = diverges();
```

### 注释

Rust有三种注释：
* 行注释 (line comments)：以`//`开头，仅能注释一行。
* 块注释 (block comments)：以`/*`开头，以`*/`结尾，能注释多行，但是不建议使用。
* 文档注释 (doc comments)：以`///`或者`//!`开头，支持Markdown标记语言，
其中`///`等价于写属性`#[doc="..."]`，`//!`等价于`#![doc="/// ..."]`，
配合`rustdoc`工具用于自动生成说明文档。

