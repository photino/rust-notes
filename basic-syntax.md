## 基本语法

### 变量绑定

在Rust中，变量绑定 (variable bindings) 是通过`let`关键字声明的：

```rust
let x = 5;
let mut x = 5;
let x: i32 = 5;
let (a, b) = (3, 4);
```
其中变量类型如`i32`一般都是可以省略的，因为Rust使用了类型推断 (type inference)。
Rust还通过模式匹配 (pattern matching) 对变量进行解构，这允许我们同时对多个变量进行赋值。

有几点是需要特别注意的：
* 变量默认是不可改变的 (immutable)，如果需要改变一个变量的值需要显式加上`mut`关键字。
* 变量具有局部作用域，被限制在所属的代码块内，并且允许变量覆盖 (variable shadowing)。
* Rust默认开启属性`#[warn(unused_variable)]`，会对未使用的变量 (以`_`开头的除外) 发出警告。
* Rust允许先声明变量然后再初始化，但是使用未被初始化的变量会产生一个编译时错误。

### 原生类型

Rust内置的原生类型 (primitive types) 有以下几类：

* 布尔类型：有两个值`true`和`false`。
* 字符类型：表示单个Unicode字符，存储为4个字节。
* 数值类型：分为有符号整数 (`i8`, `i16`, `i32`, `i64`, `isize`)、
无符号整数 (`u8`, `u16`, `u32`, `u64`, `usize`) 以及浮点数 (`f32`, `f64`)。
* 字符串类型：最底层的是不定长类型`str`，更常用的是字符串切片`&str`和堆分配字符串`String`，
其中字符串切片是静态分配的，有固定的大小，并且不可变，而堆分配字符串是可变的。
* 数组：具有固定大小，并且元素都是同种类型，可表示为`[T; N]`。
* 切片：引用一个数组的部分数据并且不需要拷贝，可表示为`&[T]`。
* 元组：具有固定大小的有序列表，每个元素都有自己的类型，通过解构或者索引来获得每个元素的值。
* 指针：最底层的是裸指针`*const T`和`*mut T`，但解引用它们是不安全的，必须放到`unsafe`块里。
* 函数：具有函数类型的变量实质上是一个函数指针。
* 元类型：即`()`，其唯一的值也是`()`。

```rust
// boolean type
let t = true;
let f: bool = false;

// char type
let c = 'c';

// numeric types
let x = 42;
let y: u32 = 123_456;
let z: f64 = 1.23e+2;
let zero = z.min(123.4);
let bin = 0b1111_0000;
let oct = 0o7320_1546;
let hex = 0xf23a_b049;

// string types
let str = "Hello, world!";
let mut string = str.to_string();

// arrays and slices
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4];
let mut ten_zeros: [i64; 10] = [0; 10];

// tuples
let tuple: (i32, &str) = (50, "hello");
let (fifty, _) = tuple;
let hello = tuple.1;

// raw pointers
let x = 5;
let raw = &x as *const i32;
let points_at = unsafe { *raw };

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
* 数组的长度是不可变的，动态的数组称为向量 (vector)，可以使用宏`vec!`创建。
* 元组可以使用`==`和`!=`运算符来判断是否相同。
* 不多于32个元素的数组和不多于12个元素的元组在值传递时是自动复制的。
* Rust不提供原生类型之间的隐式转换，只能使用`as`关键字显式转换。
* 可以使用`type`关键字定义某个类型的别名，并且应该采用驼峰命名法。

```rust
// explicit conversion
let decimal = 65.4321_f32;
let integer = decimal as u8;
let character = integer as char;

// type aliases
type NanoSecond = u64;
type Point = (u8, u8);
```

### 结构体

结构体 (struct) 是一种记录类型，所包含的每个域 (field) 都有一个名称。
每个结构体也都有一个名称，通常以大写字母开头，使用驼峰命名法。
元组结构体 (tuple struct) 是由元组和结构体混合构成，元组结构体有名称，
但是它的域没有。当元组结构体只有一个域时，称为新类型 (newtype)。
没有任何域的结构体，称为类单元结构体 (unit-like struct)。
结构体中的值默认是不可变的，需要使用`mut`使其可变。

```rust
// structs
struct Point {
  x: i32,
  y: i32,
}
let mut point = Point { x: 0, y: 0 };

// tuple structs
struct Color(u8, u8, u8);
let android_green = Color(0xa4, 0xc6, 0x39);
let (red, green, blue) = android_green;

// A tuple struct’s constructors can be used as functions.
struct Digit(i32);
let v = vec![0, 1, 2];
let d: Vec<Digit> = v.into_iter().map(Digit).collect();

// newtype: a tuple struct with only one element
struct Inches(i32);
let length = Inches(10);
let Inches(integer_length) = length;

// unit-like structs
struct Null;
let empty = Null;
```

一个包含`..`的`struct`可以用来从其它结构体拷贝一些值或者在解构时忽略一些域：

```rust
#[derive(Default)]
struct Point3d {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point3d::default();
let point = Point3d { y: 1, ..origin };
let Point3d { x: x0, y: y0, .. } = point;
```

需要注意，Rust在语言级别不支持域可变性 (field mutability)，所以不能这么写：

```rust
struct Point {
    mut x: i32,
    y: i32,
}
```
这是因为可变性是绑定的一个属性，而不是结构体自身的。可以使用`Cell<T>`来模拟：

```rust
use std::cell::Cell;

struct Point {
    x: i32,
    y: Cell<i32>,
}

let mut point = Point { x: 5, y: Cell::new(6) };

point.y.set(7);
```

此外，结构体的域默认是私有的，可以使用`pub`关键字将其设置成公开。

### 枚举

Rust有一个集合类型，称为枚举 (enum)，对于一个指定的名称有一组可替代的值，
其中子数据结构可以存储也可以不存储数据，需要使用`::`语法来获得每个元素的名称。

```rust
// enums
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}

let x: Message = Message::Move { x: 3, y: 4 };
```

与结构体一样，枚举中的元素默认不能使用关系运算符进行比较 (如`==`, `!=`, `>=`)，
也不支持像`+`和`*`这样的双目运算符，需要自己实现，或者使用`match`进行匹配。

枚举默认也是私有的，如果使用`pub`使其变为公有，则它的元素也都是默认公有的。
这一点是与结构体不同的：即使结构体是公有的，它的域仍然是默认私有的。
此外，枚举和结构体也可以是递归的 (recursive)。

### 函数

要声明一个函数，需要使用关键字`fn`，后面跟上函数名，比如
```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```
其中函数参数的类型不能省略，可以有多个参数，但是最多只能返回一个值，
提前返回使用`return`关键字。Rust编译器会对未使用的函数提出警告，
可以使用属性`#[allow(dead_code)]`禁用无效代码检查。

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
其中`///`等价于写属性`#[doc = "..."]`，`//!`等价于`#![doc = "/// ..."]`，
配合`rustdoc`可自动生成说明文档。

