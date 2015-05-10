## 内存安全

Rust推崇安全与速度至上，它没有垃圾回收机制，却成功实现了内存安全 (memory safety)。

### 所有权

在Rust中，所有权 (ownership) 系统是零成本抽象 (zero-cost abstraction) 的一个主要例子。
对所有权的分析是在编译阶段就完成的，并不带来任何运行时成本 (run-time cost)。
默认情况下，Rust是在栈 (stack) 上分配内存，对栈上空间变量的再赋值都是复制的。
如果要在堆 (heap) 中分配，必须使用`box`来构造：

```rust
let x = Box::new(5);
```
其中`Box::new`创建了一个`Box<i32>`来存储整数`5`，此时变量`x`具有该盒子的所有权。
当`x`退出代码块的作用域时，它所分配的内存资源将随之释放，这是编译器自动完成的。

考虑下面这段代码：

```rust
fn main() {
    let x = Box::new(5);

    add_one(x);

    println!("{}", x);
}

fn add_one(mut num: Box<i32>) {
    *num += 1;
}
```
调用`add_one`时，变量`x`的所有权也转移 (move) 给了变量`num`。函数完成后，
`num`占有的内存将自动释放。当`println!`再次使用已经没有所有权的变量`x`时，
编译器就会报错。一种可行的解决办法是修改`add_one`函数使其返回`box`，
把所有权再转移回来。更好的做法是引入所有权租借 (borrowing)。

当所有权转移时，可变性可以从不可变变成可变的：

```rust
let x = Box::new(5);
let mut y = x;
*y += 1;
```

### 租借

在Rust中，所有权的租借是通过引用`&`来实现的：

```rust
fn main() {
    let mut x = 5;

    add_one(&mut x);

    println!("{}", x);
}

fn add_one(num: &mut i32) {
    *num += 1;
}
```
调用`add_one()`时，变量`x`把它的所有权以可变引用借给了变量`num`。函数完成后，
`num`又把所有权还给了`x`。如果是以不可变引用借出，则租借者只能读而不能改。

有几点是需要特别注意的：

* 变量、函数、闭包以及结构体都可以成为租借者。
* 一个资源只能有一个拥有者，但是可以有多个租借者。
* 资源一旦以可变出租，拥有者就不能再访问资源内容，也不能再出租给其它绑定。
* 资源一旦以不可变出租，拥有者就不能再改变资源内容，也不能再以可变的形式出租，
但可以以不可变的形式继续出租。

### 生存期

Rust 通过引入生存期 (lifetime) 的概念来确定一个引用的作用域：

```rust
struct Foo<'a, 'b> {
    x: &'a i32,
    y: &'b i32,
}

fn main() {
    let a = &5;
    let b = &8;
    let f = Foo { x: a, y: b };

    println!("{}", f.x + f.y);
}
```
因为结构体`Foo`有自己的生存期，所以我们需要给它所包含的域指定新的生存期`'a`和`'b`，
从而确保对`i32`的引用比对`Foo`的引用具有更长的生存期，避免悬空指针 
(dangling pointer) 的问题。

Rust预定义的`'static`具有和整个程序相同的生存期，主要用于声明全局常量以及字符串：

```rust
static NUM: i32 = 5;
let x: &'static i32 = &NUM;
let s: &'static str = "Hello world!";
```

对于共享所有权，需要使用标准库中的`Rc<T>`：

```rust
use std::rc::Rc;

struct Car {
    name: String,
}

struct Wheel {
    size: i32,
    owner: Rc<Car>,
}

fn main() {
    let car = Car { name: "DeLorean".to_string() };

    let car_owner = Rc::new(car);

    for _ in 0..4 {
        Wheel { size: 360, owner: car_owner.clone() };
    }
}
```
如果是在并发中共享所有权，则需要使用线程安全的`Arc<T>`。
