> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [litchipi.github.io](https://litchipi.github.io/infosec/2023/01/24/git-code-audit-viewed-as-rust-programmer.html)

> On January 17th 2023, X41 and Gitlab published a report of the source code audit they performed on Gi......

On January 17th 2023, **X41** and **Gitlab** published a report of the source code audit they performed on Git (funded by the **OSTIF** foundation).

This post is based on the (_great_) report available [here](https://www.x41-dsec.de/static/reports/X41-OSTIF-Gitlab-Git-Security-Audit-20230117-public.pdf) and aims to investigate how Rust mitigates some of the vulnerabilities shown in this report, but also to put some light on **what it doesn’t** mitigate by itself, and how a programmer can address these issues using good practices.

> The role of these kinds of studies are primordial, and the **OSTIF** allows to fund such initiatives.  
> Please for the sake of great opensource software and computer security **consider [making a donation](https://ostif.org/donate-to-ostif/)**.

If I say anthing that is wrong / oversimplified, _do tell me please_ and I will correct this article in consequence.

The explanations are _intentionally kept simple_, please refer to the corresponding report section for more informations.  
I’ll be focusing on the basics (that should prevent me from saying too much wrong statements), and how these issues could be seen in a program made in Rust.

GIT-CR-22-01
------------

This vulnerability (described at section _4.1.1_) is an _Uncontrolled Resource Consumption_ ([CWE 400](https://cwe.mitre.org/data/definitions/400.html)), leading to a possible Denial of Service.  
The issue is caused by this loop occuring in one of the functions:

```
while (slen > 0) {
    int len = slen;      // Here if slen is too big, it loops backs to 0
    // Allocate some memory
    slen -= len;        // len = 0, slen > 0, so the loop goes infinite
}
```

When the input text grows, the value of `slen` does as well, and researchers suceeded to allocate **2.5GB** of memory using a _30MB_ file.

Let’s try to reproduce the same kind of loop in Rust now:

```
let mut slen: u64 = (u32::MAX as u64) + 1;
while slen > 0 {
    let len: u32 = slen as u32;
    println!("Some memory allocation");
    slen -= len as u64;
}
```

This code compiles without any warning and gives you a nice infinite loop indeed.  
However note the _2 casts_ that we are required to perform in order to get to this point, it’s much _easier to notice_ that something may break here, but on a large codebase with an example much more complex, _it may still be occuring_.

Lesson number one: **Rust doesn’t protect from casting overflows** if you cast naively using `as`. However you can use the `try_into` function for casting that will return a `Result<T, T::Error>` triggered if such things happen.

GIT-CR-22-02
------------

This vulnerability (described at section _4.1.2_) is an _Out of Bound Read_ ([CWE 125](https://cwe.mitre.org/data/definitions/125.html)), allowing possible sensitive data reading, or even a buffer overflow.

When dealing with strings, the computer maps the data in memory as:

```
字母：“r”   “u”   “s”   “t”
字节：0x72 0x75 0x73 0x74 0x00
```

注意到`0x00`末尾的字节了吗？这就是计算机检测字符串结尾的方式。  
由于此字节的数值为`0`，在 C 中我们使用以下方法检测它：

```
int next_char = get_next_char(some_string);
if (!next_char) {
     // 字符是字符串的结尾
}
```

然而，想象`get_next_char`不返回值，而是_指向_该值的指针，然后`!char`不再检查该值是否是`0`，但它检查_指针是否是`NULL`指针_，即使它的值可能是，**它也不会**`0`是。

这个漏洞巧妙地利用了代码如何在执行条件时_忘记取消引用_返回的指针（指向字符串中的某处）。由于此条件_始终为 true_，因此它允许执行具有无效输入的函数，从而导致易受攻击的行为。

在 Rust 中，您必须使用显式`unsafe`块来对原始指针执行这样的操作，而这并不能为您提供“全部功能”。  
由于我对_unsafe Rust_太陌生，我不会在这里尝试重现。

只要你没有_非常具体的理由_使用它，你就不应该**使用不安全的 Rust。**

GIT-CR-22-03
------------

此漏洞（在第_4.1.3_节中描述）是一个_整数溢出到缓冲区溢出_( [CWE 680](https://cwe.mitre.org/data/definitions/680.html) )，可能导致代码执行。

这里的问题是，在 Windows 64 位中， an 的大小`unsigned long`是**4 bytes**，而在 Linux 64bit 中是**8 bytes 。**  
正如在_Linux 中_`git`创建的那样，在以下代码中，出现了问题：

```
size_t msg_A_len = get_message_length(msg_a);
size_t msg_B_len = get_message_length(msg_b);
unsigned  long new_buffer_len = msg_A_len + msg_B_len + 2 ;
```

实际上，我们可以溢出 的边界`unsigned long new_buffer_len`，循环回到`0`。现在我们可以获得使`new_buffer_len`变量变小的大输入。  
现在想象一下剩下的代码会是这样的：

```
char * new_buffer = malloc (new_buffer_len);
memcpy (new_buffer, msg_a, msg_A_len);
memcpy (new_buffer + msg_A_len, msg_b, msg_B_len);
```

在这种情况下`new_buffer_len < (msg_A_len + msg_B_len)`，这意味着我们只是向内存中写入了比分配的_内存能够包含的__字节更多的字节_，我们只是**溢出**了缓冲区。

现在让我们在 Rust 中做一些类似的事情吧？

```
println!("size of usize: {}", std::mem::size_of::<usize>());
// size of usize: 8

let msg_a_len: u64 = u64::MAX >> 1;
let msg_b_len: u64 = u64::MAX >> 1;
assert_eq!(msg_a_len + msg_b_len + 1, u64::MAX);

let new_buffer_len: usize = (msg_a_len as usize) + (msg_b_len as usize) + 3;
// debug compilation:     thread panick here because of integer overflow

let some_array: Vec<u8> = Vec::with_capacity(new_buffer_len);
println!("{}", some_array.capacity());
// release compilation:         1
```

Several things on this:

*   I am running on a `x86_64` machine, meaning `usize` is the same length as an `u64`, so this code does not reproduce what actually happens in this vulnerability.
*   Notice the casts that we are required to do in order to make this compile, _this is a hint_ that the programmer have to check his bounds.
*   The overflow protection is only on in `debug` mode (and can be set/unset in the [compilation profile](https://doc.rust-lang.org/cargo/reference/profiles.html))
*   This code **cannot be exploited** in safe code to perform a _memory overflow_, because you would have to use a `Vec` here, which has _all the safeguards_ to not overflow. Arrays are _not possible_ here as their size has to be known at compile time.
*   The size of each numeric type (except `usize`) _is obvious_ to the developper and _doesn’t change_ across platforms, this is by itself a great protection as long as we pay attention to the types we use. (`let size: i32` is not a good practice at all.)

This kinds of issues can **definitly happen** in Rust if you use `as` castings all the time, and even if the memory size of variable is much more simple in Rust than in C, `usize` size in memory is arch-dependant (as described in the [usize type documentation](https://doc.rust-lang.org/std/primitive.usize.html)).

The issue may exist in Rust, but the **memory vulnerability** doesn’t (at least in safe Rust), attempts to write something and run some code would lead to the program termination.

GIT-CR-22-04
------------

This vulnerability (described at section _4.1.4_) is a _Synchronous Access of Remote Resource without Timeout_ ([CWE 1088](https://cwe.mitre.org/data/definitions/1088.html)), leading to a possible Denial of Service.

This issue is really simple to understand. When initiating a new connection for a `git clone` operation, no timeout is set, meaning that if the remote endpoint doesn’t answer, the connection is kept open.

If an attacker open X connections to endpoints he controls (and that doesn’t send any data), then the connections are kept active in `git`, the resources aren’t freed, leading to a Denial of Service.

Rust is not really protected by that kinds of things, and it’s to the attention of the developper to pay attention to _always_ put some kind of _boundaries_ to the connections, wether it is a hard limit of simultaneously opened connections, a timeout, etc …

Timeouts can be annoying to set up, but they may really be worth it as there is nothing as unexpected as a bad Internet connection, a crash from a distant server, or any kind of I/O scenario a sane programmer wouldn’t think of.

Even a long one is useful for these cases.

GIT-CR-22-05
------------

This vulnerability (described at section _4.1.5_) is an _Inefficient Regular Expression Complexity_ ([CWE 1333](https://cwe.mitre.org/data/definitions/1333.html)), leading to a possible Denial of Service by consumming excessive CPU resources.

This vulnerability happens because somewhere in the code, the user input gets interpreted as a regular expression.  
Using this, an attacker can pass a nasty Regexp that cause a denial of Service, called _ReDos_ in that case.

See the [OWASP article](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS) about this kind of attacks.

For the [regex](https://docs.rs/regex/latest/regex/) crate, the reference when dealing with regular expressions in Rust, the security is taken seriously and some features at the root cause of this kind of problems are simply not implemented. That mean that the crate is less powerful than other systems out there, but it is a tradeoff that the developpers chose to make.

See the “[Untrusted inputs](https://docs.rs/regex/latest/regex/#untrusted-input)” section from the docs of the crate for more details

GIT-CR-22-06
------------

This vulnerability (described at section _4.1.6_) is a _Heap-based Buffer Overflow_ ([CWE 122](https://cwe.mitre.org/data/definitions/122.html)), leading in the worst case scenario to arbitrary code execution.

This is also a vulnerability caused by the overflow of the type `int` when getting numbers bigger than its bounds and loops back to the minimum bound, in this case negative:

```
size_t len = get_length(buffer);
size_t padding = get_padding(input_string);

int offset = padding - len;
memcpy(buff + offset, input_string, len);
```

However here this is much more problematic as it allows to set a negative offset, and so perform a `memcpy` operation **before** the start of the buffer, and write data controlled by the attacker.

Heap overflows may not be as bad as Stack overflows, but they do have really nasty exploit possible (see [CTF101 article](https://ctf101.org/binary-exploitation/heap-exploitation/) about it).

Now would that kind of things be possible in a world covered in (_safe_) Rust ?  
We saw that Rust doesn’t always protect against numeric types overflow.

However you would have to choose between different types to use, meaning that for this situation to happen, you would have to explicitly write `i64 offset`, as `usize` is unsigned and neither are `u8 u16 u32 u64`.

I decided to count this as a _protection_, as the amount of work required to make this behavior happen assures that you brought it on yourself.  
Using `unsafe`, it is possible to get similar problems leading to possible vulnerabilities as well, for example in this code:

```
let input_string = String::from("this is longer than the length of the buffer");
let strlen: usize = input_string.len();

let bufflen: usize = 10;
let buffer = String::with_capacity(bufflen);

let offset: i64 = (bufflen as i64) - (strlen as i64);
let ptr = buffer.as_mut_ptr();
unsafe {
    std::ptr::copy(input_string.as_ptr(), ptr.offset(offset), input_string.len());
}
```

> Thanks to `u/pluuth` for pointing out the correct unsafe code here, I previously thought it was much more complex to get an issue like this to appear in Rust. I still count it as “protected” because of the mandatory `unsafe` block and the type casts you need to use.

GIT-CR-22-07
------------

This vulnerability (described at section _4.1.7_) is another _Heap-based Buffer Overflow_ ([CWE 122](https://cwe.mitre.org/data/definitions/122.html)), similarly leading in the worst case scenario to arbitrary code execution.

This is yet another `int` type overflow when handling big inputs (displaying how important this is), however this vulnerability is really **critical** as now an attacker can commit a malicious `.gitattributes` file into a remote repository, and the vulnerability will be triggered to anybody trying to `clone` or `pull` the repository.

Here the recommendation for this issue is to use a `size_t` type in order to prevent the integer overflow, however it’s also pointed out to **limit the size** of the lines in the `.gitattributes` file.

However as this is can be only exploited for memory manipulation, this is where (safe) Rust protects us. It becomes critical because of the possible arbitrary code execution following, however this could be used in Rust to create _infinite loops_, trigger conditions, etc …

GIT-CR-22-08
------------

This vulnerability (described at section _4.1.8_) is an _Uncontrolled Resource Consumption_ ([CWE 400](https://cwe.mitre.org/data/definitions/400.html)), leading to a possible Denial of Service.

The report isn’t really clear about this, but when applying a patch, this code gets triggered:

```
// apply.c:4687
static int apply_patch(struct apply_state *state,
		       int fd,
		       const char *filename,
		       int options)
{
    // ...
    offset = 0;
    while (offset < buf.len) {
            nr = parse_chunk(state, buf.buf + offset, buf.len - offset, patch);
            if (nr < 0) {
                // Error case
            }
            // Some operations
            offset += nr;
    }
    // ...
}
```

The vulnerability here comes from an issue making the `parse_chunk` returns 0, resulting in an infinite loop.

It’s caused by yet another integer overflow, when parsing a _binary patch_ file. With a header / payload long enough, you can overflow the variables, and the return value from the function gets overflowed:

```
// apply.c:2124
static int parse_chunk(struct apply_state *state,
                       char *buffer, unsigned long size, struct patch *patch)
{
    // ...
    return offset + hdrsize + patchsize;
}
```

As an `int` can be overflowed to a negative value, a malicious patch can return `0` here, and performing a `git apply` over the patch would result in an infinite loop, and a Denial of Service.

This is a case that _could totally apply_ to a Rust code if we are not careful enough, the best protection is to **use proper numeric types** to ensure a size doesn’t get negative, but you should also _learn to notice_ the conditions that would make any loop go infinite, and check for these conditions.

Remember that the Rust “safety” is only relative and under particular conditions, it’s not the same if you are writing for embedded systems, or in the Linux kernel (as [Linus explained](https://lkml.org/lkml/2022/9/19/1105) in some of the mails), and it only works if _you set up_ everything Rust needs to achieve safety

> And the _reality_ is that there are no absolute guarantees. Ever.  
> The “Rust is safe” is not some kind of absolute guarantee of code safety. Never has been.

Rust is implemented so it eliminates undefined behaviors, and handles the “wrong answer” case by returning an error, or panicking. This is a choice that makes total sense when building a software / an application, but it’s not a universal “best way to do”, it depends on what you do.

> 根本不完成操作，_并不_比得到错误的答案好多少，只是更容易调试。  
> [...]  
> 所以这是我真正_需要_Rust 人理解的东西。“安全”的整个现实并不是绝对的东西，而且内核端_需要_与传统用户空间略有不同的规则。

让我们回顾一下 Rust 中的一些良好实践（一般来说），以确保我们不会遇到太多错误，或产生一些漏洞。

铸造溢出
----

如果向上`u32`转换为`u64`，则可以使用关键字`as`，但是向下转换`u64`为时`u32`，请改用关键字`try_into`。  
如果您_确定_您的`u64`变量是`< u32::MAX`，那么您甚至可以使用`try_into().unwrap()`as 在最坏的情况下将漏洞限制为拒绝服务，并且如果发生这种情况也很容易发现。

由于`usize`内存中的大小是架构相关的（请参阅[文档](https://doc.rust-lang.org/std/primitive.usize.html)），我建议尽可能使用具有固定内存空间的数字变量类型，例如`u64`,`i32`或`f32`，以减少整数溢出恐慌的可能性。  
如果您希望您的代码在不同的体系结构上运行，那就更是如此。

限制输入大小
------

这不是 Rust 特有的，但是当您可以简单地_限制_任何输入长度时，为什么`u64`要到处使用和检查溢出呢？（或您使用的任何类型）`< u8::MAX`

输入清理很重要，输入的大小是其中的一个方面，你永远不应该忽视它，因为它会导致不良行为。您可能认为“博客文章标题超过 65536 个字符永远不会发生”，但攻击者_会_想到这种情况，并**破坏**您的代码。

不安全的锈
-----

如果你在你的 Rust 代码中使用了一些_不安全的东西，要_**非常小心**里面写的东西，_尽可能少_地使用它，尽可能地_测试_它，并且只有在你不能使用_其他方式_使它时才使用。

在这些情况下使用不安全块可能没问题：

*   写入嵌入式系统和内核代码中的内存地址
*   从其他编程语言导入代码，例如 C
*   具有全局 mut 指针，在单线程应用程序中，并且仅在确实需要时使用
*   在实施[Quake 3 平方根反](https://en.wikipedia.org/wiki/Fast_inverse_square_root#Overview_of_the_code)函数或任何类似深奥的函数时

如果某些东西以意想不到的方式中断，代码中不安全的部分肯定会成为主要嫌疑人，因此请尽可能保持清晰、简单并记录在案。

如果使用`unsafe`确实提高了性能，请添加一些基准以证明这一点，如果有一天`safe`和`unsafe`实施之间的差距越来越近，请考虑回到全安全实施。

在任何情况下，如果您的代码中**确实**有，请_对其进行广泛测试_，插入您的 CI 以在任何合并之前执行测试，并确保所有代码都经过良好测试。`unsafe`

限制软件的规模
-------

*   如果每次有人连接到您的服务器时都向其中_添加一个数据_结构，这就是一种资源消耗。`Vec`
*   如果在有人连接到您的服务器时_启动一个线程_执行某种_计算_，这就是资源消耗。

在这两种情况下（以及许多其他类型的示例），您都需要考虑“如果整个地球都想连接到我的应用程序会发生什么？”

答案是，您的服务器会崩溃或融化。因此，对用户数量设置**一些限制**，或者设置一个（廉价的）等待队列，一旦“资源单元”可用，用户将从该队列中重定向。  
网上有很多资源可以学习如何处理这些问题，所以我不会在这里给你任何幼稚的建议，你自己去研究吧。

我个人认为，如果您巧妙地_设计资源的使用方式_，并采取适当的方法来_改进_软件**在压力下**的行为，您可以让应用程序在微型服务器上运行，为数百万用户提供服务。

Rust 的保护
--------

Rust 本身，当不使用任何_unsafe_时，可以防止一些漏洞，包括**2 个 Critical**、_1 个 high_和 1 个 medium 。

但是，它无法防范 4 个 Low 漏洞。

请注意，大部分保护不是因为漏洞没有发生，而是因为_它不可利用_，或者至少影响较小。这就是 Rust 关于内存操作的规则可以保护你免受的影响。

然而，由于导致这些漏洞的问题仍然可能发生，您仍然可能存在漏洞，如果您仅依赖“Rust 是安全的” ，则_可能存在严重的漏洞。_

_在大多数情况下_， Rust是内存安全的，但并非所有漏洞利用都是关于内存利用的，**没有什么**是_永远_安全的，总是**怀疑**软件的安全性，无论您是编写它还是购买它。  
在代码或生活中，应用[深度防御](https://en.wikipedia.org/wiki/Defense_in_depth_(computing))，了解您使用的技术的优缺点，并保持开放的心态:-)

编写安全的 Rust 代码
-------------

最重要的是，**您在编写代码时应该小心**，尤其是在处理系统输入（无论是人为还是网络/磁盘）时。  
这就是_广泛测试_有用的地方（例如参数测试），一般来说，攻击者越能控制易受攻击代码中的输入，他就越_能严重利用它_。

您可能认为它仅适用于大型开源项目并且您的代码没有所有这些也可以，但我认为**培训很重要，**因为这些“良好实践”只有_在重复_足够多的情况下才会变得自动。因此，在构建用 Rust 或其他任何东西重写的下一个（超快的）GNU 工具时，请尝试考虑一下安全性。  
_习惯于__默认_编写安全代码。

捐赠给OSTIF
--------

[您可以点击此链接](https://ostif.org/donate-to-ostif/)向 OSTIF 基金捐款。

该报告可[在此处](https://www.x41-dsec.de/static/reports/X41-OSTIF-Gitlab-Git-Security-Audit-20230117-public.pdf)访问，摘要可在[X41 的网站上查看](https://x41-dsec.de/security/research/news/2023/01/17/git-security-audit-ostif/)。

再一次，如果您资助本文中错误/过于简单的内容，**请告诉我**，以便我立即更正。

> 特别感谢  
> `u/Rodrigodd_`for pointing out some things to improve in the article  
> `u/Shnatsel`for pointing an imprecision in `GIT-CR-22-03`'s conclusion  
> `u/milliams`for correcting some typos  
> `u/ssokolow`for correcting some typos and grammar errors, and details about unsafe benchmarking 和测试