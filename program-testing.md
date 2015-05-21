## 程序测试

### 测试属性

在测试函数前加上`#[test]`属性：

```rust
#[test]
fn it_works() {
    assert!(false);
}
```
其中`assert!`宏接受一个参数，如果参数为`false`，它会导致`panic!`。
运行`cargo test`命令，可见该测试失败。如果要反转测试失败，
可以加上`#[should_panic]`属性：

```rust
#[test]
#[should_panic(expected = "assertion failed")]
fn it_works() {
    assert_eq!("Hello", "world");
}
```

### 测试模块

在测试模块前加上`#[cfg(test)]`属性：

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod test {
    use super::add_two;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

### 测试目录

对于集成测试，可以新建一个`tests`目录，这样其中的代码就不需要再引入单元风格的测试模块了。

### 文档测试

对于包含有测试例子的注释文档中，运行`cargo test`时也会运行其中包含的测试。

````rust
//! The `adder` crate provides functions that add numbers to other numbers.
//!
//! # Examples
//!
//! ```
//! assert_eq!(4, adder::add_two(2));
//! ```

/// This function adds two to its argument.
///
/// # Examples
///
/// ```
/// use adder::add_two;
///
/// assert_eq!(4, add_two(2));
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}
````

### 错误处理

Rust明确区分两种形式的错误：失败 (failture) 和恐慌 (panic)。
失败是可以通过某种方式恢复的错误，而恐慌（panic）则不能够恢复。

最简单的表明函数会失败的方法是使用`Option<T>`类型：

```rust
fn from_str<A: FromStr>(s: &str) -> Option<A> {

}
```
其中`from_str()`返回一个`Option<A>`。如果转换成功，它会返回`Some(value)`；
如果失败，直接返回`None`。对于需要提供出错信息的情形，可以使用`Result<T, E>`类型：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

如果不想处理错误，可以使用`unwrap()`方法来产生恐慌：

```rust
let mut buffer = String::new();
let input = io::stdin().read_line(&mut buffer).unwrap();
```
当`Result`是`Err`时，`unwrap()`会`panic!`，直接退出程序。另一个更好的做法是：

```rust
let input = io::stdin().read_line(&mut buffer)
                       .ok()
                       .expect("Failed to read line");
```
其中`ok()`将`Result`转换为`Option`，`expect()`和`unwrap()`功能类似，
可以用来提供更多的错误信息。

此外，还可以使用宏`try!`来封装表达式，当`Result`是`Err`时会从当前函数提早返回`Err`。

