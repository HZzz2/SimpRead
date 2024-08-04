> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [elastio.github.io](https://elastio.github.io/bon/blog/how-to-do-named-function-arguments-in-rust)

> Generate builders for everything!为所有内容生成构建器！

Many modern languages have a built-in feature called "named function arguments". This is how it looks in Python, for example:  
许多现代语言都具有一种名为“命名函数参数”的内置特性。 例如，在Python中，它是这样的：

py

```
def greet(name: str, age: int) -> str:
    return f"Hello {name} with age {age}!"

greeting = greet(
    # Notice the `key = value` syntax at the call site
    ,                                        
    age=24,                                            
)

assert greeting == "Hello Bon with age 24!"
```

This feature allows you to associate the arguments with their names when you call a function. This improves readability, API stability, and maintainability of your code.  
这个功能允许你在调用函数时将参数与它们的名称关联起来。这可以提高代码的可读性、API 的稳定性和可维护性。

It's just such a nice language feature, which is not available in Rust, unfortunately. However, there is a way to work around it, that I'd like to share with you.  
这是一个非常好的语言特性，在Rust中不幸地无法使用。然而，有一种解决方法可以绕过这个问题，我想和你一起分享。

Naive Rust solution [​](#naive-rust-solution) 幼稚的 Rust 解决方案
-----------------------------------------------------------

Usually, Rust developers deal with this problem by moving their function parameters into a struct. However, this is inconvenient, because it's a lot of boilerplate, especially if your function arguments contain references:  
通常，Rust 开发人员通过将其函数参数移入结构体来处理这个问题。然而，这种方法不太方便，因为它涉及大量样板代码，尤其是当函数参数包含引用时。

rust

```
// Meh, we need to annotate all lifetime parameters here
struct GreetParams<'a> {
    name: &'a str,
    age: u32
}

fn greet(params: GreetParams<'_>) -> String {
    // We also need to prefix each parameter with `params`
    format!("Hello {} with age {}!", params.name, params.age)
}

// Ugh... consumers need to reference the params struct name here,
// which usually requires an additional `use your_crate::GreetParams`
greet(GreetParams {
    name: "Bon",
    age: 24
});
```

This situation becomes worse if you have more references in your parameters or if your parameters need to be generic over some type. In this case, you need to duplicate the generic parameters both in your function and in the parameters struct declaration. Also, the convenient function's `impl Trait` syntax is unavailable with this approach.  
如果您的参数中有更多的引用，或者您的参数需要对某种类型进行泛化，那么情况会变得更糟。在这种情况下，您需要在函数和参数结构声明中都复制泛型参数。此外，这种方法无法使用便捷函数的 impl Trait 语法。

The other caveat is that the struct literal syntax doesn't entirely solve the problem of omitting optional parameters. There is a clunky way to have optional parameters by requiring the caller to spread the `Default` parameters struct instance in the struct literal.  
另一个警告是，结构体字面量语法并不能完全解决省略可选参数的问题。通过要求调用者在结构体字面量中展开默认参数结构实例的方式，可以使用笨拙的方式来实现可选参数。

rust

```
struct GreetParams<'a> {
    name: &'a str,
    age: u32,

    // Optional
    job: Option<String> 
}

GreetParams {
    name: "Bon",
    age: 24,

    // But.. this only works if your whole `GreetParams` struct
    // can implement the `Default` trait, which it can't in this
    // case because `name` and `age` are required
    ..GreetParams::default() 
}
```

This situation can be slightly improved if you derive a builder for this struct with [`typed-builder`](https://docs.rs/typed-builder/latest/typed_builder/), for example. However, `typed-builder` doesn't remove all the boilerplate. I just couldn't stand, but solve this problem the other way.  
这种情况可以稍微改善，如果您使用 typed-builder 为该结构派生一个构建器，例如。然而，typed-builder 并不能消除所有样板代码。我就是忍受不了，所以用另一种方式解决了这个问题。

Better Rust solution [​](#better-rust-solution) 更好的Rust解决方案
-----------------------------------------------------------

I've been working on a crate called [`bon`](https://elastio.github.io/bon/docs/guide/overview) that solves this problem. It allows you to have almost the same syntax for named function arguments in Rust as you'd have in Python. [`bon`](https://elastio.github.io/bon/docs/guide/overview) generates a builder directly from your function:  
我一直在开发一个叫做 bon 的 crate 来解决这个问题。它允许你在 Rust 中拥有几乎与 Python 中相同的命名函数参数的语法。bon 直接从你的函数中生成一个构建器：

RustPython

rust

```
#[bon::builder] 
fn greet(name: &str, age: u32) -> String {
    format!("Hello {name} with age {age}!")
}

let greeting = greet()
    .name("Bon") 
    .age(24)     
    .call();

assert_eq!(greeting, "Hello Bon with age 24!");
```

py

```
def greet(name: str, age: int) -> str:
    return f"Hello {name} with age {age}!"


greeting = greet(
    , 
    age=24,     
)

assert greeting == "Hello Bon with age 24!"
```

We only placed `#[bon::builder]` on top of a regular Rust function. We didn't need to define its parameters in a `struct` or write any other boilerplate. In fact, this Rust code is almost as succinct as the Python version. Click on `Python` at the top of the code snippet above to compare both versions.  
我们只是在常规Rust函数的顶部放置了#[bon::builder]。我们不需要在结构体中定义其参数，也不需要编写任何其他样板文件。实际上，这个Rust代码几乎和Python版本一样简洁。单击上面代码片段顶部的Python以比较两个版本。

I hope you didn't break that button 🐱  
希望你没有按坏那个按钮 🐱

The builder generated by [`bon`](https://elastio.github.io/bon/docs/guide/overview) allows the caller to omit any parameters that have `Option` type. That part is covered.  
bon 生成的构建器允许调用者省略任何具有 Option 类型的参数。这部分已经覆盖。

It supports almost any function syntax. It works fine with functions inside of `impl` blocks, `async` functions, generic parameters, lifetimes (including implicit and anonymous lifetimes), `impl Trait` syntax, etc. The full list of supported syntax is [here](https://elastio.github.io/bon/docs/guide/overview#supported-syntax-for-functions).  
它支持几乎所有的函数语法。它可以很好地处理 impl 块内部的函数、异步函数、泛型参数、生命周期（包括隐式和匿名生命周期）、impl Trait 语法等。支持的语法完整列表请参见这里。

It also never panics. All errors such as missing required arguments are compile-time errors ✔️.  
它也从不恐慌。所有的错误，比如缺少必需的参数，都是编译时错误✔️。

[`bon`](https://elastio.github.io/bon/docs/guide/overview) allows you to _easily_ switch your regular function into a function "with named parameters" that returns a builder. Just adding `#[builder]` to your function is enough even in advanced use cases.  
bon 允许您轻松地将常规功能切换为返回构建器的带命名参数的功能。即使在高级用例中，只需将#[builder]添加到您的功能中即可。

One `#[builder]` to rule them all [​](#one-builder-to-rule-them-all)  
一个#[builder]统治所有人
----------------------------------------------------------------------------------------

`bon`'s `#[builder]` was designed to cover all your needs for a builder. It works not only with functions and methods, but it also works with `struct`s as well:  
bon 的#[builder]被设计用于满足您对构建器的所有需求。它不仅可以与函数和方法一起使用，还可以与结构体一起使用：

rust

```
use bon::builder;

#[builder]
struct User {
    id: u32,
    name: String,
}

User::builder()
    .id(1)
    .name("Bon")
    .build();
```

So.. you can use just one `#[builder]` solution consistently for everything. Builders for functions and structs both share the same API design, which allows you, for example, to switch between a `#[builder]` on a struct and a `#[builder]` on a method that creates a struct. This won't be an API-breaking change for you consumers ([details](https://elastio.github.io/bon/docs/guide/compatibility#moving-builder-from-the-struct-to-the-new-method)).  
这样，您可以始终只使用一个#[builder]解决方案来处理所有事情。函数和结构体的构建器共享相同的 API 设计，这使您可以在结构体上切换到#[builder]，也可以在创建结构体的方法上切换到#[builder]。这对您的用户不会产生 API 破坏性变化（详细信息）。

Summary [​](#summary) 摘要
------------------------

If you'd like to know more about `bon` and use it in your code, then visit the [guide page with the full overview](https://elastio.github.io/bon/docs/guide/overview).  
如果您想了解更多关于 bon 的信息并在代码中使用它，请访问包含完整概述的指南页面。

[`bon`](https://elastio.github.io/bon/docs/guide/overview) is fully open-source, and [available on crates.io](https://crates.io/crates/bon). Its code is public on [Github](https://github.com/elastio/bon). Consider giving it a star ⭐ if you liked the idea and the implementation.  
bon 是完全开源的，并且可以在 crates.io 上获得。 它的代码是公开的在 Github 上。 如果您喜欢这个想法和实现，考虑给它点赞⭐。

Acknowledgements [​](#acknowledgements) 致谢
------------------------------------------

This project was heavily inspired by such awesome crates as [`buildstructor`](https://docs.rs/buildstructor), [`typed-builder`](https://docs.rs/typed-builder) and [`derive_builder`](https://docs.rs/derive_builder). [`bon`](https://elastio.github.io/bon/docs/guide/overview) crate was designed with many lessons learned from them.  
这个项目深受 buildstructor、typed-builder 和 derive_builder 等优秀库的启发。bon 库的设计汲取了从这些库中学到的许多经验教训。

TIP 提示

You can leave comments for this post [on Reddit](https://www.reddit.com/r/rust/comments/1eeem92/how_to_do_named_function_arguments_in_rust/).  
你可以在 Reddit 上为这篇文章留言。