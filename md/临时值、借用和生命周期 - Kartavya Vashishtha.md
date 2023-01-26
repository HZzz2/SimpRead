> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.vashishtha.in](https://blog.vashishtha.in/temporary-lifetimes/)

> `temporary value dropped while borrowed`, anyone?

Prior to Rust [v1.61.0](https://releases.rs/docs/1.61.0/ "发行说明"), the [`std::io::stdio::Stdin::lock`](https://doc.rust-lang.org/stable/std/io/struct.Stdin.html#method.lock) function [had a signature](https://github.com/rust-lang/rust/pull/93965/ "相关的拉取请求") that looked like this:

```
pub fn lock(&self) -> StdinLock<'_>
```

Lifetimes [un-elided](https://doc.rust-lang.org/reference/lifetime-elision.html#lifetime-elision-in-functions "Rust 参考 - 函数中的生命周期省略"), it looks like:

```
pub fn lock<'a>(&'a self) -> StdinLock<'a>
```

which translates to the following:

*   The returned `StdinLock` is valid for the lifetime `'a`
*   The lifetime `'a` originates from a borrow of a `Stdin` struct.

> It also implies that the `StdinLock` must be dropped before mutably borrowing the `Stdin` struct, and that we can't drop the `Stdin` struct before dropping the `StdinLock` struct.

So what happens when we use `lock`?

```
use std::io;
fn main() {
  let locked = io::stdin().lock();
}
```

> `pub fn stdin() -> Stdin`, btw.

**A wild compile error appears!**

```
error[E0716]: temporary value dropped while borrowed
 --> src/main.rs:3:14
  |
3 |     let locked = io::stdin().lock();
  |                  ^^^^^^^^^^^       - temporary value is freed at the end of this statement
  |                  |
  |                  creates a temporary which is freed while still in use
4 | }
  | - borrow might be used here, when `locked` is dropped and runs the destructor for type `StdinLock<'_>`
  |
  = note: consider using a `let` binding to create a longer lived value
```

> But _where_ did the borrow come from? Well, when calling a method with an `&self` parameter with an owned value, the Rust compiler can [implicitly borrow](https://doc.rust-lang.org/reference/expressions.html#implicit-borrows "Rust 参考 - 隐式借用") the owned value to call the `&self` method.

*   A borrow is _against_ an owned value. That is, borrows can't be conjured from thin air. Borrows _aren't_ pointers. You can't _hold_ a value with a borrow, you need an owner backing it.
*   A borrow can only last as long as the owned value. It can't live after the owned value is dropped.
*   [Temporaries](https://doc.rust-lang.org/reference/expressions.html#temporaries "Rust 参考 - 临时文件") are dropped at the end of a statement ([except when they're not](https://doc.rust-lang.org/reference/destructors.html#temporary-lifetime-extension "临时延长寿命")), while values stored in `let`s drop at the end of a block.

For more information on dropping behaviour, see [Drop Order](https://doc.rust-lang.org/reference/destructors.html#drop-scopes).

For a much more concise explanation of the problem, I defer to the original issue for changing `std::io::Stdin::lock`:

> The explanation is that the lock behaves as if it borrows the original handle from stdin(), and the temporary value created for the call to the lock() method is dropped at the end of the statement, invalidating the borrow.
> 
> -- [tlyu](https://github.com/rust-lang/rust/issues/86845#issue-936286530%3E "tlyu 的 Issue 评论")

Why don't you encounter the same problem when borrowing in other places? Most reference parameters to functions don't have their lifetimes tied to the output in the same way. This compiles just fine:

```
fn print_str (s: &str) {
	println!("{}", s);
}

fn main () {
	print_str(String::from("abc").as_str());
}
```

while the de-elided (delided?) signature of `as_str()` is `pub fn as_str(&self) -> &str`, which also returns a reference with the [same](https://doc.rust-lang.org/reference/lifetime-elision.html#lifetime-elision-in-functions "Rust 参考 - 函数中的生命周期省略") lifetime as the input.. However! If we add but a return argument to `print_str`...

```
fn print_str (s: &str) -> &str {
	println!("{}", s);
	s
}

fn main () {
	print_str(String::from("abc").as_str());
}
```

The output:

```
abc
```

What. I totally thought I was going to prove my point there. What if I...

```
fn print_str (s: &str) -> &str {
	println!("{}", s);
	s
}

fn main () {
	let s = print_str(String::from("abc").as_str());
	dbg!(s);
}
```

Aha!

```
error[E0716]: temporary value dropped while borrowed
 --> src/main.rs:7:20
  |
7 |     let s = print_str(String::from("abc").as_str());
  |                       ^^^^^^^^^^^^^^^^^^^          - temporary value is freed at the end of this statement
  |                       |
  | 创建一个临时文件，它在仍在使用时被释放8 | 数据库！（小号）； 

  | - 借来以后用在这里
  |
help :考虑 使用`let`绑定来 创建一个寿命更长的值
  |
7 ~ let binding = String :: from ( "abc" );
8 ~ 让 s = print_str(binding.as_str());
  |
```

我们借来的价值（参考）只能与我们借来的价值保持一致，在这种情况下，临时值在语句末尾被删除。以前，Rust 编译器不会给我们一个错误，因为我们后面没有使用借用，而借用的生命周期可能与临时的一样。

但在这种情况下，返回的引用与`print_str`返回的引用具有相同的[ish](https://doc.rust-lang.org/nomicon/subtyping.html "Rustonomicon - 子类型和方差")生命周期，`as_str`受临时`String`.

> 一旦你开始了解 Rust 编译器为了让[生命周期更方便](https://doc.rust-lang.org/nomicon/subtyping.html "Rustonomicon - 子类型和方差")而使用的所有怪异技巧，你就会对称两个生命周期“相等”或“相同”感到不舒服。

解决方案正是编译器所建议的：一旦我们将拥有的值放入`let`语句中，它就会在作用域的末尾被丢弃，我们可以自由地借用它直到那时。

那么 Rust 社区是如何改变的`Stdin::lock`，让用户不再面临这个错误呢？他们将签名更改为`pub fn lock(&self) -> StdinLock<'static>`，这解耦了生命周期关系，允许返回`StdinLock`值比`&self`引用更有效。[他们如何做到这](https://github.com/rust-lang/rust/pull/93965/ "相关的拉取请求")一点留给读者。

*   [方差和协方差](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science))，以及它与[Rust 生命周期](https://doc.rust-lang.org/nomicon/subtyping.html "Rustonomicon - 子类型和方差")的关系——它们可以缩小。
*   [](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)如果您到目前为止只读过这[本书，](https://doc.rust-lang.org/book/) [Pretzelhammer 关于常见 Rust 生命周期误解的文章](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)会让_您大吃一惊。_[](https://doc.rust-lang.org/book/)
*   [fasterthanlime 的“A Rust made in hell”](https://fasterthanli.me/articles/a-rust-match-made-in-hell)探讨了一个类似的问题，该问题与匹配表达式检查器如何延长临时生命周期以及它如何与锁冲突有关。

  

* * *

一如既往，请随时[给我写信](mailto:sendtokartavya@gmail.com)指出错误、建议主题或打个招呼！