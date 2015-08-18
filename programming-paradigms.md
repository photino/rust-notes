## 编程范式

Rust是一个多范式 (multi-paradigm) 的编译型语言。除了通常的结构化、命令式编程外，
还支持以下范式。

### 函数式编程

Rust使用闭包 (closure) 来创建匿名函数：

```rust
let num = 5;
let plus_num = |x: i32| x + num;
```
其中闭包`plus_num`借用了它作用域中的`let`绑定`num`。如果要让闭包获得所有权，
可以使用`move`关键字：

```rust
let mut num = 5;

{ 
    let mut add_num = move |x: i32| num += x;

    add_num(5);
}

assert_eq!(5, num);
```

Rust 还支持高阶函数 (high order function)，允许把闭包作为参数来生成新的函数：

```rust
fn apply(f: &Fn(f64) -> f64, x: f64) -> f64 {
    f(1.0) + x
}

fn factory(x: f64) -> Box<Fn(f64) -> f64> {
    Box::new(move |y| x + y)
}

let f = factory(2.0);
let g = &(*f);

assert_eq!(5.0, f(3.0));
assert_eq!(5.0, g(3.0));
assert_eq!(6.0, apply(g, 3.0));
```

### 面向对象编程

Rust通过`impl`关键字在`struct`或者`trait`上实现方法调用语法 (method call syntax)。
关联函数 (associated function) 的第一个参数通常为`self`参数，有3种变体：
`self`，`&self`和`&mut self`。不含`self`参数的关联函数称为静态方法 (static method)。

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }

    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());

    // use associated function and method chaining
    println!("{}", Circle::new(0.0, 0.0, 2.0).area());
}
```

为了描述类型可以实现的抽象接口 (abstract interface)，
Rust引入了特性 (trait) 来定义函数类型签名 (function type signature)：

```rust
trait HasArea {
    fn area(&self) -> f64;
}

struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

struct Square {
    x: f64,
    y: f64,
    side: f64,
}

impl HasArea for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}

fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```
其中函数`print_area()`中的泛型参数`T`被添加了一个名为`HasArea`的特性约束 (trait constraint)，
用以确保任何实现了`HasArea`的类型将拥有一个`.area()`方法。
如果需要多个特性限定 (multiple trait bounds)，可以使用`+`：

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn bar<T, K>(x: T, y: K)
    where T: Clone,
          K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
}
```
其中第二个例子使用了更灵活的`where`从句，它还允许限定的左侧可以是任意类型，
而不仅仅是类型参数。

定义在特性中的方法称为默认方法 (default method)，可以被该特性的实现覆盖。
此外，特性之间也可以存在继承 (inheritance)：

```rust
trait Foo {
    fn foo(&self);

    // default method
    fn bar(&self) { println!("We called bar."); }
}

// inheritance
trait FooBar : Foo {
    fn foobar(&self);
}

struct Baz;

impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
}

impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
}
```

如果两个不同特性的方法具有相同的名称，可以使用通用函数调用语法 (universal function call syntax)：

```rust
// short-hand form
Trait::method(args);

// expanded form
<Type as Trait>::method(args);
```

关于实现特性的几条限制：

* 如果特性并不定义在当前作用域内，它就不能被实现。
* 不管是特性还是`impl`，都只能在当前的包装箱内起作用。
* 带有特性约束的泛型函数使用单态 (monomorphization)，
所以它是静态派分的 (statically dispatched)。

下面列举几个非常有用的标准库特性：

* `Drop`提供了当一个值退出作用域后执行代码的功能，它只有一个`drop(&mut self)`方法。
* `Borrow`用于创建一个数据结构时把拥有和借用的值看作等同。
* `AsRef`用于在泛型中把一个值转换为引用。
* `Deref<Target=T>`用于把`&U`类型的值自动转换为`&T`类型。
* `Iterator`用于在集合 (collection) 和惰性值生成器 (lazy value generator) 上实现迭代器。

推荐阅读：
* [Visualizing Rust's type-system](http://jadpole.github.io/rust/type-system/)
* [Rust's Built-in Traits, the When, How & Why](https://llogiq.github.io/2015/07/30/traits.html)
* [Effectively Using Iterators In Rust](http://hermanradtke.com/2015/06/22/effectively-using-iterators-in-rust.html)

### 元编程

泛型 (generics) 在类型理论中称作参数多态 (parametric polymorphism)，
意为对于给定参数可以有多种形式的函数或类型。先看Rust中的一个泛型例子：

```rust
enum Option<T> {
    Some(T),
    None,
}

let x: Option<i32> = Some(5);
let y: Option<f64> = Some(5.0f64);
```
其中`<T>`部分表明它是一个泛型数据类型。当然，泛型参数也可以用于函数参数和结构体域：

```rust
// generic functions
fn make_pair<T, U>(a: T, b: U) -> (T, U) {
    (a, b)
}
let couple = make_pair("man", "female");

// generic structs
struct Point<T> {
    x: T,
    y: T,
}
let int_origin = Point { x: 0, y: 0 };
let float_origin = Point { x: 0.0, y: 0.0 };
```

对于多态函数，存在两种派分 (dispatch) 机制：静态派分和动态派分。
前者类似于C++的模板，Rust会生成适用于指定类型的特殊函数，然后在被调用的位置进行替换，
好处是允许函数被内联调用，运行比较快，但是会导致代码膨胀 (code bloat)；
后者类似于Java或Go的`interface`，Rust通过引入特性对象 (trait object) 来实现，
在运行期查找虚表 (vtable) 来选择执行的方法。特性对象`&Foo`具有和特性`Foo`相同的名称，
通过转换 (casting) 或者强制多态化 (coercing) 一个指向具体类型的指针来创建。

当然，特性也可以接受泛型参数。但是，往往更好的处理方式是使用关联类型 (associated type)：

```rust
// use generic parameters
trait Graph<N, E> {
    fn has_edge(&self, &N, &N) -> bool;
    fn edges(&self, &N) -> Vec<E>;
}

fn distance<N, E, G: Graph<N, E>>(graph: &G, start: &N, end: &N) -> u32 {

}

// use associated types
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}

fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> uint {

}

struct Node;

struct Edge;

struct SimpleGraph;

impl Graph for SimpleGraph {
    type N = Node;
    type E = Edge;

    fn has_edge(&self, n1: &Node, n2: &Node) -> bool {

    }

    fn edges(&self, n: &Node) -> Vec<Edge> {

    }
}

let graph = SimpleGraph;
let object = Box::new(graph) as Box<Graph<N=Node, E=Edge>>;
```
Rust中的宏 (macro) 允许我们在语法级别上进行抽象。先来看`vec!`宏的实现：

```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```
其中`=>`左边的`$x:expr`模式是一个匹配器 (matcher)，`$x`是元变量 (metavariable)，
`expr`是片段指定符 (fragment specifier)。匹配器写在`$(...)`中，
`*`会匹配0个或多个表达式，表达式之间的分隔符为逗号。
`=>`右边的外层大括号只是用来界定整个右侧结构的，也可以使用`()`或者`[]`，
左边的外层小括号也类似。扩展中的重复与匹配器中的重复会同步进行：
每个匹配的`$x`都会在宏扩展中产生一个单独的`push`语句。

### 并发计算

Rust提供了两个特性来处理并发 (concurrency)：`Send`和`Sync`。
当一个`T`类型实现了`Send`，就表明该类型的所有权可以在进程间安全地转移；
而实现了`Sync`就表明该类型在多线程并发时能够确保内存安全。

Rust的标准库`std::thread`提供了并行执行代码的功能：

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hello from a thread!"
    });

    println!("{}", handle.join().unwrap());
}
```
其中`thread::scoped()`方法接受一个闭包，它将在一个新线程中执行。

Rust尝试解决可变状态的共享问题，通过所有权系统来帮助排除数据竞争 (data race)：

```rust
use std::sync::{Arc, Mutex};
use std::sync::mpsc;
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(0u32));

    // Creates a shared channel that can be sent along from many threads
    // where tx is the sending half (tx for transmission),
    // and rx is the receiving half (rx for receiving).
    let (tx, rx) = mpsc::channel();

    for i in 0..10 {
        let (data, tx) = (data.clone(), tx.clone());

        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            *data += i;

            tx.send(*data).unwrap();
        });
    }

    for _ in 0..10 {
        println!("{}", rx.recv().unwrap());
    }
}
```
其中`Arc<T>`类型是一个原子引用计数指针 (atomic reference counted pointer)，
实现了`Sync`，可以安全地跨线程共享。`Mutex<T>`类型提供了互斥锁 (mutex's lock)，
同一时间只允许一个线程能修改它的值。`mpsc::channel()`方法创建了一个通道 (channel)，
来发送任何实现了`Send`的数据。`Arc<T>`的`clone()`方法用来增加引用计数，
而当离开作用域时计数减少。

