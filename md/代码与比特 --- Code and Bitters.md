> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [codeandbitters.com](https://codeandbitters.com/once-upon-a-lazy-init/)

> I've often wondered about the differences between lazy_static and once_cell, and in Rust 1.70 and 1.8......

I've often wondered about the differences between `lazy_static` and `once_cell`, and in Rust 1.70 and 1.80 the standard library is gaining the ability to create one-time-initialized values.

Most big projects end up using one of these crates, because lazy initialization is a very convenient way to implement almost-`const` global values that can't actually be `const`-initialized, because they need heap memory (e.g. `String` or `Vec`), read files or environment variables, or use types that don't have `const` initializers yet.

But how do they work? Let's spend some time looking at the differences between these three implementations.

![](https://codeandbitters.com/images/rust_logo_hand_drawn_1b.png)

#### Part 1: Lazy-initialize all the things! [§](#part-1-lazy-initialize-all-the-things)

The [`lazy_static`](https://crates.io/crates/lazy_static) crate was first published in November 2014, over 5 months before Rust 1.0. It basically does one thing: provide a macro that allows you to declare a static variable at module scope. For example, this allows us to declare a global hashmap:

```
use lazy_static::lazy_static;

lazy_static! {
    static ref ENTITIES: Mutex<HashMap<String, u32>> =
        Mutex::default();
}
```

The [`once_cell`](https://crates.io/crates/once_cell) crate came along much later, in Augest 2018. It allows you to do very similar things, but without any macros. The previous example might look like this:

```
use once_cell::sync::Lazy;

static ENTITIES: Lazy<Mutex<HashMap<String, u32>>> =
    Lazy::new(Mutex::default);
```

`once_cell` contains a few other tools as well. For example if you want to place your initialization code where the value is retrieved (rather than where it's defined) you can use `OnceCell`:

```
use once_cell::sync::OnceCell;

static ENTITIES: OnceCell<Mutex<HashMap<String, u32>>> =
    OnceCell::new();

fn do_initialization() {
    let entities = ENTITIES.get_or_init(Mutex::default);
    // do stuff with entities...
}
```

The `OnceCell` type can also be embedded in your own data structure, and has thread-safe and non-thread-safe variants.

The `lazy_static` and `once_cell` crates share a few common characteristics:

*   If concurrent initialization is attempted, exactly one thread will run the initialization function and place the result in the variable. Any other threads will block until it completes.
*   Initialization will happen at most once, and only shared references are available afterwards (internal mutability can be used if necessary).

Starting with Rust 1.70, the standard library stabilized its own lazy-initialized containers. They are similar to the types found in the `once_cell` crate, but have slightly different names:

<table><thead><tr><th></th><th>once_cell</th><th>std (1.70+)</th></tr></thead><tbody><tr><td>thread-safe</td><td><code>once_cell::sync::OnceCell</code></td><td><code>std::sync::OnceLock</code></td></tr><tr><td>non-thread-safe</td><td><code>once_cell::unsync::OnceCell</code></td><td><code>std::cell::OnceCell</code></td></tr></tbody></table>

As of Rust 1.80, the standard library [`LazyLock`](https://doc.rust-lang.org/std/sync/struct.LazyLock.html) type has also been stabilized. So now there are std equivalents for the `once_cell` `Lazy` types:

<table><thead><tr><th></th><th>once_cell</th><th>std (1.80+)</th></tr></thead><tbody><tr><td>thread-safe</td><td><code>once_cell::sync::Lazy</code></td><td><code>std::sync::LazyLock</code></td></tr><tr><td>non-thread-safe</td><td><code>once_cell::unsync::Lazy</code></td><td><code>std::cell::LazyCell</code></td></tr></tbody></table>

Not all methods from `once_cell` are present in the standard library, but std handles most simple use cases.

#### Part 2: I have preferences. [§](#part-2-i-have-preferences)

I think that in most cases, `once_cell` should be preferred over `lazy_static`. The reason is mostly that `lazy_static!` is a macro, and `OnceCell` isn't. This affects the programmer experience in a bunch of ways:

*   `static ref FOO` isn't valid Rust syntax. This makes it difficult for people new to the language— they're trying to learn proper Rust syntax, and when they see code examples that use "incorrect" syntax it's easy to get confused.
*   rustfmt won't format code inside a macro, so you wind up with quirky code styles inside `lazy_static!` blocks.
*   `lazy_static` disables the `dead_code` lint, so you won't notice if the value becomes unused in the future.
*   variables defined inside a `lazy_static!` block _appear_ to have a straightforward type, but you can't use them as though they are that type— often one needs to add a gratuitous deref `*`, and it's not obvious why.

For example, this code doesn't compile:

```
lazy_static! {
    static ref LAZY_STRING: String = format!("hello!");
}

fn foo() {
    println!("{}", LAZY_STRING);
}
```

The error is unhelpful:

```
error[E0277]: `LAZY_STRING` doesn't implement `std::fmt::Display`
 --> src/lib.rs:8:20
  |
8 |     println!("{}", LAZY_STRING);
  |                    ^^^^^^^^^^^ `LAZY_STRING` cannot be formatted with the default formatter
  |
  = help: the trait `std::fmt::Display` is not implemented for `LAZY_STRING`
  = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```

The reason this error is confusing is that `LAZY_STRING` in the error message is not a _static variable_ called `LAZY_STRING`, it's a type called `LAZY_STRING`.

If we expand the output of the `lazy_static!` macro we see this:

```
#[allow(missing_copy_implementations)]
#[allow(non_camel_case_types)]
#[allow(dead_code)]
struct LAZY_STRING {
    __private_field: (),
}
#[doc(hidden)]
static LAZY_STRING: LAZY_STRING = LAZY_STRING {
    __private_field: (),
};
impl $crate::__Deref for LAZY_STRING {
    type Target = String;
    fn deref(&self) -> &String {
        #[inline(always)]
        fn __static_ref_initialize() -> String {
            ({
                let res = $crate::fmt::format(::core::fmt::Arguments::new_v1(&["hello!"], &[]));
                res
            })
        }
        #[inline(always)]
        fn __stability() -> &'static String {
            static LAZY: $crate::lazy::Lazy<String> = $crate::lazy::Lazy::INIT;;
            LAZY.get(__static_ref_initialize)
        }
        __stability()
    }
}
impl $crate::LazyStatic for LAZY_STRING {
    fn initialize(lazy: &Self) {
        let _ = &**lazy;
    }
}
```

That's a lot to digest. But here are the things that jump out to me:

1.  There is a `struct LAZY_STRING` and a `static` variable also called `LAZY_STRING`. This leads to confusing compiler warnings or errors, and also confusing behavior within rust-analyzer.
2.  The inner type is not a member of the `LAZY_STRING` struct. It's somewhat hidden away as an associated type, and the output of an associated function called `__stability`.
3.  The inner value is not contained within the static variable! It's stored in another static variable inside the associated function `__stability`.

I like code that is understandable and debuggable by a casual reader of the source code, and contains minimal surprises. For those reasons I find `lazy_static` to be a bit obfuscated. For the most part we can get away with using it and not understanding the inner magic, but if we can get the same result with `once_cell` (and none of the lurking surprises), I think that's a better choice.

The code that's generated by the `lazy_static!` macro is also not handled well by rust-analyzer. rust-analyzer can't tell what type of data is stored inside the static variable, so you can't easily click through to the type definition. Even basic things like "Find all References" don't work (though that might also be due to the macro rather than the code emitted.)

I also think that `std` should be preferred over third-party crates when there's no functional difference. It makes builds faster and simpler, and fewer dependencies is always an improvement. The only exceptions would be if you need methods in `once_cell` that aren't yet available in `std`, or are held back by an msrv lower than 1.80.

#### Part 3: Discussion of `once_cell` [§](#part-3-discussion-of-once-cell)

`once_cell`'s `Lazy` type is the most similar to `lazy_static` in that you declare the initialization function at the same time as the variable.

`Lazy` is a bit more powerful than `lazy_static`, in that it can also be used in structs and enums that aren't global. Any member variable or enum field can be lazy-initialized.

`once_cell` also has the `OnceCell` type, which is quite similar to `Lazy`, except that it allows you to specify the initialization code separately, e.g. in a call to `get_or_init()`. This allows use of a closure that captures local variables, for example.

`once_cell` has two versions of `Lazy` and `OnceCell`, that share the same names but live in different modules: `unsync` contains non-thread-safe versions that don't implement Sync, while the `sync` module contains thread-safe versions that do implement `Sync` (assuming the inner `T` is also `Sync`.)

The `sync` types rely on `std::sync::Once` to provide blocking of threads that race to initialize the contents. This is normally the strategy you want if your target is a regular operating system and not, say, a bare-metal microcontroller.

The non-sync version does not need to do thread blocking because that situation should never arise— `once_cell::unsync::OnceCell` doesn't implement `Sync` so it should not be possible for two threads to access it concurrently (at initialization time or otherwise).

#### Part 4: Surprises lurk in `lazy_static` [§](#part-4-surprises-lurk-in-lazy-static)

`lazy_static` has support for spinlock based blocking. This isn't something you should want in threads running on a real OS, so hopefully you'll never have this happen, but because the `spin` feature of the lazy_static crate isn't strictly additive, it's technically possible that some other crate could accidentally turn on the spinlock behavior without your knowledge.

I almost deleted this paragraph from an earlier draft, and in the weeks between when I started this post and when I finished it, I fell victim to this problem myself! There are public crates with millions of downloads that enable the `spin_no_std` feature in `lazy_static`, which will result in the rest of the workspace unintentionally using spinlocks instead of OS-based blocking.

`lazy_static` is a `macro_rules!` macro, not a procedural macro, so it shouldn't add much to your build time.

`lazy_static` can easily be used with values that are not Send. This is only possible because the macro generates the value at module scope, where it can never be dropped. `once_cell::sync` types (and std) require `Send + Sync`, otherwise the resulting container will not be `Sync`. `static` variables require `Sync`, so this means that in most cases you'll need your inner `T` to impl `Send + Sync`.

Another unusual property of `lazy_static` is that it creates a type and a static variable, and it's possible to actually impl traits against the type. This can be really confusing, because the code inside the macro invocation doesn't give any hint that this is possible. The result is so obfuscated that I don't think it's a good idea to use this feature.

#### Part 5: Async support [§](#part-5-async-support)

All of the three implementations discussed here will invoke OS-level blocking if multiple threads race to initialize. This approach may be problematic in an async environment, where blocking the executor is considered a bit rude.

If lazy initialization is only used for a small number of globals, it's possible no one will notice the difference. If an async code base contains data structures with lazy-initialized members that are created often, then it may be a good idea to consider other approaches instead.

Tokio offers an async version of [`OnceCell`](https://docs.rs/tokio/latest/tokio/sync/struct.OnceCell.html#) with limited async initialization support.

#### Closing thoughts [§](#closing-thoughts)

Lazy initialization can often solve issues that would have been tricky to code up by hand. For example: if you want to spawn a bunch of concurrent threads or tasks that might need access to a config file later, but only want to read the config file when it's first needed (but never read it more than once.)

It's not always the first tool one should reach for, though. Sometimes things can be plain `static`, instead of using lazy initialization. For example, `Mutex<Option<T>>` can be const-initialized to `None`, without any extra machinery. It's worth trying to do this before resorting to any lazy-initialized type; your code will be simpler and easier to debug.

Also, maybe it should be argued that globals are harmful. They can be a source of spooky action at a distance, and can make the code harder to understand and debug. It might be better to pass things as parameters or in member variables instead.

Cheers! Good luck with your Rust projects.