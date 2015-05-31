## 高级主题

### 外部函数接口

在Rust中，通过外部函数接口 (foreign function interface) 可以直接调用C语言库：

```rust
extern crate libc;
use libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```
其中`#[link]`属性用来指示链接器链接`snappy`库，`extern`块是外部库函数标记的列表。
外部函数被假定为不安全的，所以调用它们需要包装在`unsafe`块中。

当然，我们也可以把Rust代码编译为动态库来供其它语言调用：
```rust
// in embed.rs
use std::thread;

#[no_mangle]
pub extern "C" fn process() {
    let handles: Vec<_> = (0..10).map(|_| {
        thread::spawn(|| {
            let mut _x = 0;
            for _ in (0..5_000_001) {
                _x += 1
            }
        })
    }).collect();

    for h in handles {
        h.join().ok().expect("Could not join a thread!");
    }
}
```
其中`#[no_mangle]`属性用来关闭Rust的命名改编，
默认的应用二进制接口 (application binary interface) `"C"`可以省略。
然后使用`rustc`进行编译：
```shell
$ rustc embed.rs -O --crate-type=dylib
```
这将生成一个动态库`libembed.so`。在Node.js中调用该函数，需要安装`ffi`库：

```shell`
$ npm install ffi
```

```js
// in test.js
var ffi = require('ffi');

var lib = ffi.Library('libembed', {
  'process': [ 'void', [] ]
});

lib.process();

console.log("done!");
```
其中`ffi.Library()`用来加载动态库`libembed.so`，
它需要标明外部函数的返回值类型和参数类型 (空数组表明没有参数)。
