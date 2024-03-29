> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ntietz.com](https://ntietz.com/blog/rust-references-vs-pointers/)

> 2023 年 2 月 6 日，星期一

**2023 年 2 月 6 日，星期一**

我一直致力于编写[Rust 培训课程](https://yet-another-rust-resource.pages.dev/)，其中我很难解释的一件事是引用和指针之间的区别。

最终，底层表示是**相同的**：两者都持有一些内存的地址。它们之间的区别最终在于语义。

引用有一些由编译器强制执行的[规则。](https://doc.rust-lang.org/nomicon/references.html)具体来说，它们不能比它们所引用的内容（“引用”）更长寿，并且可变引用不能被别名。除此之外，引用的行为很像它们指向的变量。它们有一个类型，您可以与该类型交互以读取它或（使用可变引用）修改它。

另一方面，指针在语义上更多地是关于地址的。这意味着当我们与它们交互时，我们将修改地址（诸如`add`指针偏移而不是添加到基础值之类的事情）。当我们打印它们时，我们不会打印底层值——事实上，如果没有关键字，我们根本无法获得底层值`unsafe`。相反，我们打印出地址。

我们可以通过一个简单的程序看到这一点。

```
fn主要() {
    让x: u32 = 10 ;
    让ref_x: &u32 = &x;
    让pointer_x: * const u32 = &x;

    println!( "x: {x}" );
    println!( "ref_x: {}" , ref_x);
    println!( "pointer_x: {:?}" , pointer_x);
}
```

首先，我们创建一个无符号的 32 位整数并为其赋值。然后我们创建一个对相同值的引用，我们还将创建一个指向它的指针。然后我们尝试打印出来。

当我们执行它时，我们得到这个输出：

```
x：10
 ref_x：10
 pointer_x：0x7ffd046a6444
```

当我们直接与变量或引用交互时，我们得到了底层值。但是有了指针，我们就得到了地址！

您仍然可以使用指针访问基础值，但您必须使用`unsafe`才能这样做。要了解原因，我们可以尝试在没有的情况下取消引用原始指针`unsafe`并得到一条错误消息：

```
错误[E0133]：取消引用原始指针是不安全的，需要不安全的函数或块
  --> 源代码/main.rs: 10 : 32
   |
10 | println!( "*pointer_x: {}" , *pointer_x);
   | ^^^^^^^^^^ 取消引用原始指针
   |
   = 注意：原始指针可能为null、悬挂或未对齐；它们可能违反别名规则并导致数据竞争：所有这些都是未定义的行为
   = 注意：此错误源于宏` $crate::format_args_nl `，它来自宏` println `的扩展（在夜间构建中，使用 -Z macro-backtrace 运行以获取更多信息）
```

重要的一点在注释中：

> 注意：原始指针可能为空、悬空或未对齐；它们可能违反别名规则并导致数据竞争：所有这些都是未定义的行为

事实上，如果我们将它包装在 中`unsafe`，它将起作用：

```
println!( "*pointer_x: {}" , unsafe { *pointer_x } );
```

使用引用是安全的。编译器将检查您是否多次为同一个可变变量设置别名，以确保您没有数据竞争。它将确保任何引用都不会超过它们所引用的内存。你必须自己用原始指针验证所有这些东西，所以这是不安全的。

这就是 Rust 中引用和指针的区别：**它们具有相同的底层数据，但与编译器的约束和语义不同**。

如果这篇文章对您有好处或有用，**请分享！** 如果您有意见、问题或反馈，请发送电子邮件至[我的公共收件箱](mailto:~ntietz/public-inbox@lists.sr.ht)或[我的个人邮箱](mailto:me@ntietz.com)。要获取新帖子，请使用我的[RSS 提要](https://ntietz.com/atom.xml)。