# Rust中的闭包

**`move`可以把所有权强制移入闭包，但是使用闭包内的字段的方式由实现的`Trait`决定**，如果是`FnOnce`可以以`self.field`所有权的方式使用捕获的字段或使该字段`return`出闭包；
而`FnMut`就算闭包拥有`field`的所有权，也只能`(&mut self).field`必须加上引用`&mut (&mut self).field`才行，不可以以所有权的方式使用该字段或把该字段`return`出闭包（因为给一个引用，不可以通过引用窃取所有权）

例子：
```rust
///编译通过
fn c_mut1() -> impl FnMut() -> [i32; 2] {
    let mut s = [1, 2];
    move || {
        s[0] = 0;
        s
    }
}

///会有如下编译错误
fn c_mut2() -> impl for<'a> FnMut(&'a str) -> String {
    let mut s = "sss".to_owned();
    move |i| {
        s += i;
        s
    }
}

```

![错误](/assets/img/rust/closure/closure1.png)

- 因为实现的是`FnMut Trait`,即使`move`使闭包获得了捕获变量的所有权，但是只能于`&mut self`的方式使用`field`，及无法从`self.field`获取所有权，只能`&mut self.field`来使用。

- 所有会有如上错误，`let mut s = "sss".to_owned();`中的`self.s`没有实现`Copy Trait`，不可以复制出去。

- 而`c_mut1`方法中的`let mut s = [1, 2];`实现了`Copy Trait`，可以`self.s`复制出去。


## 总结

- Rust 闭包中的捕获外部变量的方式取决于实现了哪一个`Trait`，而实现哪一个`Trait`取决于怎么使用外部变量。这些都是默认前提下的反应。

- 而`move`可以强制以拥有所有权的方式捕获外部变量。我们还可以指定实现哪一个`Trait`，不取决于如何使用外部变量。
比如`c_mut2()`方法中通过返回类型指定实现`FnMut Trait`，虽然`move`进去了所有权，但`FnMut Trait`使得闭包中只能以`&mut (&mut self).field`使用捕获的外部变量，无法获取所有权来使用。（因为不能通过一个引用来获取字段的所有权）