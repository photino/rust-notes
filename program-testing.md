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

