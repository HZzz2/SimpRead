> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.devgenius.io](https://blog.devgenius.io/integrating-rust-with-android-development-ef341c2f9cca)

> Rust 是一种通用的系统编程语言，已经存在了很长一段时间......

![](https://miro.medium.com/max/1400/1*asJ0X12wzo_gN_zxscdV5Q.jpeg)

[**Rust**](https://www.rust-lang.org/)是一种通用的系统编程语言，已经存在了很长一段时间。作为一种系统编程语言，它可用于执行类似于**C**和**C++**等语言的任务，但具有更高的内存安全性。这使得 Rust 可用于在包括 Android 在内的众多操作系统上编写程序或脚本。您可能想知道这怎么可能，以及是否有一种简单的方法可以做到这一点。嗯，这就是这篇文章的内容！

目前，关于如何为 Android 应用程序编写 Rust 代码的信息并不多。[谷歌在这里](https://source.android.com/setup/build/rust/building-rust-modules/overview)提供了一些信息，但是对于初学者来说理解起来很复杂。本演练的目标是提供一个简单但有效的指南，将 Rust 代码与 Android 开发集成，并最终成为首选**指南。不需要 C 或 C++ 或 JNI 的先验知识！**

我们将使用[Android Studio](https://developer.android.com/studio)。让我们从设置它开始。

Rust 插件⚙️
---------

首先，我们需要 Rust 插件。打开安卓工作室。将打开一个类似于下面显示的对话框。选择 Plugins 选项卡，搜索“Rust”，然后安装 JetBrains 的官方 Rust 插件。或者，如果您已经打开了一个项目，请单击 Android Studio 左上角的文件，然后选择设置。单击插件。

> Rust 插件需要在您的系统上安装 Rust。您可以按照此[**链接**](https://www.rust-lang.org/tools/install)在您的系统上安装 Rust。

![](https://miro.medium.com/max/1400/1*vIp429xNBAmk56_FdhOZmw.png)

创建一个空的应用程序📱
------------

现在让我们创建一个新的空应用程序。首先完成标准的新项目设置。

![](https://miro.medium.com/max/1400/1*eMtX0LGhY1ZVlBt7p12CGg.png)

让我们将我们的项目命名为 Rust Application，然后单击完成。

将出现一个类似于下图的窗口。

![](https://miro.medium.com/max/1400/1*75wCKhDm7MfQ0UPu3m8ZSQ.png)

加入货运项目💼
--------

[**现在让我们按照此处**](https://doc.rust-lang.org/cargo/guide/creating-a-new-project.html)提供的教程添加一个[Cargo 项目](https://doc.rust-lang.org/cargo/)。要嵌入 Rust 项目，只需单击**Android Studio中的****终端**并键入，然后按 Enter。[](https://doc.rust-lang.org/cargo/guide/creating-a-new-project.html) `cargo new rust_lib --lib`

```
C:\Users\USER1\AndroidStudioProjects\RustApplication>cargo new rust_lib --lib

```

这将创建一个新库，以便可以在我们的应用程序中使用它。我们将库命名为`rust_lib`。记下创建文件夹的位置。

要在 Android Studio 中查看该文件夹，请导航到 Project 窗格并从下拉菜单中选择 Project 而不是 Android。

![](https://miro.medium.com/max/1140/1*T-2-mcR6WYG_qKYLfgj0ZQ.png)

然后，您将看到一个名为`rust_lib`. `git`默认情况下，此文件夹包含一个新存储库、一个`Cargo.toml`文件和一个`src`包含`lib.rs`. 现在让我们修改我们刚刚使用该文件创建[的库的类型。](https://doc.rust-lang.org/cargo/reference/cargo-targets.html#configuring-a-target)`Cargo.toml`对于移动应用程序开发，库应该是动态的。

将以下内容添加到`Cargo.toml`文件的内容中。

```
[库]
名称 = “rust_lib”
板条箱类型 = [“cdylib”]

```

[在这里](https://gist.github.com/Kofituo/e02bd971b75866103438961e98e3711f)查看要点。

添加依赖🧶
------

让我们添加将创建必要文件的依赖项，以使将我们的 rust 代码链接到 Android 结束一个顺利的过程。

使用“ ***** ”拉取最新版本的依赖。

`[**flapigen**](https://crates.io/crates/flapigen)` 是从我们的 rust 代码生成适当代码以在我们的 android 应用程序中使用它的主要构建依赖项。`flapigen`使用接口文件，但每次更改代码时都必须修改接口文件可能会变得乏味。这就是`[**rifgen**](https://crates.io/crates/rifgen)` 它的来源。它使接口文件的创建变得轻而易举。

额外的依赖项用于日志记录。

创建构建文件📄
--------

如前所述，flapigen 和 rifgen 是构建依赖项并与`[build.rs](https://doc.rust-lang.org/cargo/reference/build-scripts.html#build-scripts)`文件交互。右键单击`rust_lib`文件夹并选择新建 > Rust 文件。

![](https://miro.medium.com/max/1400/1*N7IPHePJFtDux9YTVt0JwA.png)

命名文件`build.rs`。

`[flapigen](https://dushistov.github.io/flapigen-rs/java-android-example.html)`按照和给出的教程`[rifgen](https://docs.rs/rifgen/latest/rifgen/)`，我们`build.rs`应该类似于以下内容：

构建.rs

您可能想知道`build.rs`. `flapigen`将接口文件中的内容转换为`java_glue`rust 文件。所以我们先指定接口文件（`in_src`）的源文件，然后指定输出文件（`java_glue.rs`）。的目录`java_glue.rs`必须与`OUT_DIR` 环境变量相同。之后，使用`rifgen`生成接口文件，根据我们添加 rust 项目的语言指定我们的偏好。的最后一个参数`Generator::new`指定包含我们的 rust 代码的文件夹的开始，该`generate_interface`函数采用接口文件的路径。即`in_scr`。指定创建的`java_folder`java 文件应该去哪里。打电话`swig_gen.expand`告诉`flapigen`生成相应的文件。当我们在 Gradle 构建中使用这些参数时，请注意这些参数。

最后，在文件夹中创建一个名为`java_glue.rs`的`rust_lib/src`文件，并放入以下内容：

并添加：

```
mod java_glue;
pub 使用 crate::java_glue::*;

```

到你的`lib.rs`文件。这样做会将您的 rust 代码连接到生成的代码。

添加 Android 工具链和链接器 ⛓
--------------------

要为 android 编译 Rust，我们需要将 android 工具链添加到`rustup`. 为此，只需在终端中运行以下命令：

```
>rustup 默认每晚
>rustup 目标添加 aarch64-linux-android armv7-linux-androideabi

```

由于`rifgen`crate 可以在 nightly 中使用，因此您必须先在 nightly 中安装 Rust。

然后分别为 64 位和 32 位 android 版本添加 rust 工具链和标准库。现在，让我们添加[编译器链接器](https://en.wikipedia.org/wiki/Linker_(computing))。它应该被添加到`rust_lib/.cargo/config.toml`文件中。

右键单击该`rust_lib`文件夹，创建一个名为的新目录，`.cargo`然后在`.cargo`名为`config.toml`.

相应的链接器随**Android NDK**一起提供，因此应在继续之前[下载。](https://developer.android.com/studio/projects/install-ndk#default-version)

将以下内容添加到您创建的配置文件中。

将**ANDROID SDK**替换为 android SDK 路径（或包含 NDK 捆绑包的文件夹）。此外，将**OS VERSION**替换为您的 OS 版本。例如，在我的 Windows 电脑上，完整路径是：

```
[target.aarch64-linux-android]
链接器 = "C:\\Users\\taimoor\\AppData\\Local\\Android\\Sdk\\ndk-bundle\\toolchains\\llvm\\prebuilt\\windows-x86_64\\bin\\aarch64-linux -android21-clang++.cmd"

```

请注意`.cmd`Windows 版本的末尾。

> 如果您使用带有 NDK v25 的 Mac OS，您可能会遇到问题。如果您确实遇到问题，可以尝试以前版本的 NDK。或者，您可以[联系更多以获得更多帮助](https://www.upwork.com/freelancers/~0196d30a485de56f48)。

使用 Gradle 自动化构建🐘
-----------------

Gradle 可用于`cargo build`在我们想要测试应用程序时自动运行。这将导致我们在 Rust 方面所做的更改自动更新到我们的应用程序中。

构建.gradle

请注意，这是**应用程序或模块级别**的 Gradle 文件，而不是项目级别的 Gradle 文件。添加第**20 到 27**行和第**65**行。第 65 行以后只是指示 Gradle 在我们运行应用程序时运行 cargo build。运行 cargo build 命令后，我们将创建的 java 文件和动态库文件 (.so ) 复制**并**粘贴到 Android 可以读取并用于我们的 Kotlin 代码的目录中。

应用程序现在应该运行 😄。如果您在运行它时遇到困难，请在下面发表评论或随时通过_otukof@gmail.com_给我发邮件！

**奖励部分：日志记录**📃
---------------

让我们快速实现日志记录以帮助调试我们的 Rust 代码。在`lib.rs`文件中，添加以下内容：

我们使用`[android_logger](https://docs.rs/android_logger/0.10.1/android_logger/index.html)`crate 生成日志，然后调用`[log_panics::init()](https://docs.rs/log-panics/2.0.0/log_panics/fn.init.html)`以重定向所有恐慌以记录而不是打印到标准错误。注意属性`#[generate_interface]`。这告诉`rifgen`我们要从 Kotlin 调用这个函数，所以它应该将该方法调用添加到接口文件中。

从 Kotlin 加载库📲
--------------

现在我们已经设置了 Rust 方面，让我们继续讨论 Kotlin 方面。我们现在将从单例加载库：

MainActivity.kt

为我们创建的 Logs 类添加各种导入。请注意，即使我们将函数命名为`initialise_logging`，我们也可以将其用作`initialiseLogging`. 这是因为我们在设置时指定了 CamelCase `rifgen`。

运行程序以查看来自被记录的 Rust 代码的信息。