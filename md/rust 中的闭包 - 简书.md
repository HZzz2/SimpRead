> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/275cc5f0e0a4)

Rust 的 **闭包**（closures）是可以保存进变量或作为参数传递给其他函数的匿名函数。可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。不同于函数，**闭包允许捕获调用者作用域中的值**。我们将展示闭包的这些功能如何复用代码和自定义行为。

一、闭包的语法
=======

![](http://upload-images.jianshu.io/upload_images/6201627-70e2c09edb74c088.png) 闭包基本语法

可以从示例上看出，闭包的语法跟函数很像。三个部分如下：

> | 参数 1， 参数 2, ...| 
> 
> -> 返回值类型
> 
> {执行体}

执行结果如下图：

![](http://upload-images.jianshu.io/upload_images/6201627-7673182f71dce14a.png) 执行结果

可以发现没有报错正常通过了，即完成了我们预期的功能。

#### 闭包的常见省略形式有如下三种：

![](http://upload-images.jianshu.io/upload_images/6201627-42e4de51a4a87c05.png) 三种常见省略

而当闭包在**捕获上文的自由变量**时，管道符里的参数也可以省略：

![](http://upload-images.jianshu.io/upload_images/6201627-68bddcf679f94059.png) 捕获外部环境变量时的省略

如上图展示的代码，也是能正常通过编译的。

二、闭包的二次调用错误
===========

示例代码如下：

![](http://upload-images.jianshu.io/upload_images/6201627-c2cb03498e56b58e.png) 示例

可以看到首先我们有一个闭包的定义，当我们在调用闭包的时候，先传进去了 string 类型的参数，之后又调用闭包传进了 int32 类型的参数，此时编译就会出错了。

![](http://upload-images.jianshu.io/upload_images/6201627-b2e45a9433bf0c9d.png) 编译出错

编译会出错的原因如下。**第一次使用 String 值调用 example_closure 时，编译器推断 x 和此闭包返回值的类型为 String 。接着这些类型被锁定进闭包 example_closure 中，如果尝试对同一闭包使用不同类型则会得到类型错误。**

**三、闭包捕获环境中的值**
===============

闭包可以作为内联匿名函数来使用，不过闭包还有另一个函数所没有的功能：他们可以捕获其环境并访问其被定义的作用域的变量。

有示例代码如下所示：

![](http://upload-images.jianshu.io/upload_images/6201627-6600fa7b338583c3.png) 闭包捕获环境

这里，即便 x 并不是 equal_to_x 的一个参数， equal_to_x 闭包也被允许使用变量 x ，因为它与 equal_to_x 定义于相同的作用域。

这就是闭包的神奇之处，函数是不能做到这一点的。

当闭包从环境中捕获一个值，闭包会在闭包体中储存这个值以供使用。这**会使用内存并产生额外的开销**，在更一般的场景中，当我们不需要闭包来捕获环境时，我们不希望产生这些开销。因为函数从未允许捕获环境，定义和使用函数也就从不会有这些额外开销。**闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，可变借用和不可变借用。**这三种捕获值的方式被编码为如下三个 Fn trait：

![](http://upload-images.jianshu.io/upload_images/6201627-b85208400ea8ca7c.png) 文档中描述的三种 trait

看完了 rust 文档中的描述，来做个简单总结：

> **Fn** trait，表示闭包以**不可变引用**的方式来捕获环境中的自由变量，同时也表示该闭包没有改变环境的能力，并且可以多次调用。对应 **&self**。
> 
> **FnMut** trait，表示闭包以**可变引用**的方式来捕获环境中的自由变量，同时意味着该闭包有改变环境的能力，也可以多次调用。对应 **&mut self**。
> 
> **FnOnce** trait，表示闭包通过**转移所有权**来捕获环境中的自由变量，同时意味着该闭包没有改变环境的能力，只能调用一次，因为该闭包会消耗自身。对应 **self**。

1、Fn 闭包
-------

有如下示例代码：

![](http://upload-images.jianshu.io/upload_images/6201627-f236411d96f60c94.png) 示例

编译运行后，结果如下：

![](http://upload-images.jianshu.io/upload_images/6201627-7f7105148216acff.png) 示例输出

可以看到，闭包对环境中 a 的捕获，是以一种类似函数中**不可变引用**传参的方式进行的，闭包没有取得 a 的所有权，所以在闭包调用完之后，在 println! 中还可以正常使用 a。

2、FnMut 闭包
----------

有如下示例代码：

![](http://upload-images.jianshu.io/upload_images/6201627-6b0b220ed11a8c92.png) 示例

当闭包中会修改外部环境的自由变量值时，函数闭包 fn_mut_closure 也得加上 mut 关键词。当闭包执行时，它默认以**可变引用**的形式传入闭包，闭包没有获取其所有权，所以闭包执行结束后，println! 可以正常输出 a 的值。输出如下：

![](http://upload-images.jianshu.io/upload_images/6201627-c12ab3fc69909aed.png) 输出

输出中可以看出，环境中的自由变量 a 的值确实被改变了。

3、FnOnce 闭包
-----------

有示例如下：

![](http://upload-images.jianshu.io/upload_images/6201627-f6ece1996ab2e44f.png) 示例

这个示例中，闭包 call_me 中发生了所有权的移交 (move 语义)，Box 指针所指向的堆内存的所有权从 a 移交给了 c，而随着 c 离开包后生命周期的结束，该堆内存也被释放掉。

环境中的自由变量 a 在被闭包执行一次之后，就失效了。

![](http://upload-images.jianshu.io/upload_images/6201627-b08cfe5aa91ec572.png) 报错

这种情况下二次调用肯定就发生报错了。

4、闭包强行获得环境中自由变量的所有权
-------------------

如果你希望强制闭包获取其使用的环境值的所有权，可以**在参数列表前使用 move 关键字**。这个技巧在将闭包传递给新线程以便将数据移动到新线程中时最为实用。

如我们在 Fn 闭包中展示的代码，如果我们在参数列表前加上了 move 关键字，如下图：

![](http://upload-images.jianshu.io/upload_images/6201627-8ec11a2a63c2bb2e.png) move 关键字会获取所有权

闭包获得了 a 的所有权，因此编译会出错：

![](http://upload-images.jianshu.io/upload_images/6201627-b8b6363f07e4a635.png) 编译出错

编译出错的信息一目了然，就是因为 move 强行让所有权从 a 移到了闭包上，原来的 a 也就失效了，没有办法被 println! 打印出来。