# rust 工具集入门

> 原文：<https://developers.redhat.com/blog/2017/11/01/getting-started-rust-toolset-rhel>

我们今年秋天推出的新软件系列之一是针对 [Rust](https://www.rust-lang.org/) 的，这是一种编程语言，旨在实现内存和线程安全，而不牺牲性能。悬空指针和数据竞争在编译时被捕获，同时仍然优化到没有语言运行时的快速本机代码！

在 rust-toolset-7 中，我们包含了在 Red Hat Enterprise Linux 7 上开始 rust 编程所需的一切，采用了熟悉的软件集合格式。在这个版本中，我们将推出 Rust 1.20 及其配套的 Cargo 0.21——两者都是技术预览版。(注意:我们的工具集名称中的“-7”是为了与现在发布的其他集合同步，即 devtoolset-7、go-toolset-7 和 llvm-toolset-7。)

## 安装防锈工具套件-7

该工具集可在 RHEL 7 服务器的`rhel-7-server-devtools-rpms` repo 中获得，或在 RHEL 7 工作站的`rhel-7-workstation-devtools-rpms`中获得。(如果你还没有 RHEL 7，红帽提供免费的 RHEL 订阅供开发使用[这里](https://developers.redhat.com/products/rhel/download/)。)可以使用 subscription manager 来启用 repo，然后像往常一样使用 yum 来安装它。

```
# subscription-manager repos --enable rhel-7-server-devtools-rpms
# yum install rust-toolset-7
```

这将安装`rustc`编译器、标准库和`cargo`构建工具。安装后，您可以验证版本和安装路径:

```
$ scl enable rust-toolset-7 'rustc -V'
rustc 1.20.0
$ scl enable rust-toolset-7 'cargo -V'
cargo 0.21.1
$ scl enable rust-toolset-7 'rustc --print sysroot'
/opt/rh/rust-toolset-7/root/usr
```

运行每个命令的`scl`可能会很麻烦，但是启用适当的路径来启动 shell 是很容易的:

```
$ scl enable rust-toolset-7 bash
```

本文的其余部分假设了这样一个已启用的 shell。

## 从 Hello World 开始

如果没有这种通用的问候，编程语言会是什么样子呢？这是如此根深蒂固，以至于我们在这里就可以直接看到这个例子。Rust 项目是用 cargo 工具管理的，所以让我们创建一个新程序:

```
$ cargo new --bin hello      
     Created binary (application) `hello` project
```

这将自动创建一个新的项目清单，`hello/Cargo.toml`:

```
[package]
name = "hello"
version = "0.1.0"
authors = ["Josh Stone <jistone@redhat.com>"]

[dependencies]
```

和一个新的源文件，`hello/src/main.rs`:

```
fn main() {
    println!("Hello, world!");
}
```

请随意编辑它，然后用 cargo 构建并运行它:

```
$ cd hello/
$ cargo run
   Compiling hello v0.1.0 (file:///home/jistone/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.18 secs
     Running `target/debug/hello`
Hello, world!
```

## 尝试一些外部依赖

Rust 库被称为 crates，在线托管在 [crates.io](https://crates.io/) ，cargo 会自动下载并构建你所有的依赖项。让我们创建一个快速项目来生成一个随机数向量，然后对它们进行排序。首先运行`cargo new --bin sorter`，然后在`Cargo.toml`中添加对[和](https://crates.io/crates/rand)机箱的依赖:

```
[dependencies]
rand = "0.3"
```

然后编辑`src/main.rs`如下所示:

```
extern crate rand;

use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let mut numbers: Vec<i8> = rng.gen_iter().take(10).collect();
    println!("random numbers: {:?}", numbers);

    numbers.sort();
    println!("sorted numbers: {:?}", numbers);
}
```

然后构建并运行它:

```
$ cargo run
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.17
 Downloading libc v0.2.32
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/sorter`
random numbers: [-83, -69, -75, -72, -48, 12, 82, -30, -101, 106]
sorted numbers: [-101, -83, -75, -72, -69, -48, -30, 12, 82, 106]
```

这很简单——增加一些并行性怎么样？Rust 具有强大的编译时保护，可以防止数据竞争，并且有许多 crates，可以由此构建并行抽象，所以让我们尝试一下 [rayon](https://crates.io/crates/rayon) 中的排序功能。首先，向`Cargo.toml`添加另一个依赖项:

```
[dependencies]
rand = "0.3"
rayon = "0.8"
```

然后在`src/main.rs`中使用:

```
extern crate rand;
extern crate rayon;

use rand::Rng;
use rayon::prelude::*;

fn main() {
    let mut rng = rand::thread_rng();
    let mut numbers: Vec<i8> = rng.gen_iter().take(10).collect();
    println!("random numbers: {:?}", numbers);

    numbers.par_sort(); // now sorting in parallel threads!
    println!("sorted numbers: {:?}", numbers);
}
```

然后构建并再次运行它:

```
$ cargo run
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rayon v0.8.2
 Downloading rayon-core v1.2.1
 Downloading num_cpus v1.7.0
 Downloading lazy_static v0.2.9
 Downloading coco v0.1.1
 Downloading futures v0.1.16
 Downloading either v1.3.0
 Downloading scopeguard v0.3.3
   Compiling either v1.3.0
   Compiling scopeguard v0.3.3
   Compiling rayon-core v1.2.1
   Compiling lazy_static v0.2.9
   Compiling futures v0.1.16
   Compiling num_cpus v1.7.0
   Compiling coco v0.1.1
   Compiling rayon v0.8.2
   Compiling sorter v0.1.0 (file:///home/jistone/sorter)
    Finished dev [unoptimized + debuginfo] target(s) in 6.90 secs
     Running `target/debug/sorter`
random numbers: [120, -103, 88, -1, 102, -122, -125, -20, -70, -114]
sorted numbers: [-125, -122, -114, -103, -70, -20, -1, 88, 102, 120]
```

现在有了更多的可传递依赖，但是我们不需要重新构建我们已经拥有的两个。结果应该不会令人惊讶——当然是不同的随机数，但仍然是有序的。

我们刚刚运行了默认的`[unoptimized + debuginfo]`构建。如果您想开始测量性能，您应该使用“`cargo run --release`”，启用优化。当然，一个微小的 10 项向量在这里不是很有趣。如果我增加这个程序来生成 1 亿个项目——`gen_iter().take(100_000_000)`——并删除打印，那么我的双核/四线程笔记本电脑在 8.2 秒内运行串行排序，在 3.8 秒内运行并行排序。虽然这是一个非常随意的基准测试，但对于几乎没有努力的人来说，这仍然是一个不错的加速！

## 安装一个有用的 Rust 程序

现在你有了一个可以工作的 Rust 工具链，也许你会喜欢用 Rust 编写一个完整的程序。你可以依靠`rustc`和`cargo`本身，但是我也推荐安装 [ripgrep](https://github.com/BurntSushi/ripgrep) 。这是一个递归 grep 工具，类似于 Ack 或 Silver Searcher，但是[甚至更快](http://blog.burntsushi.net/ripgrep/)！

```
$ cargo install ripgrep
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading ripgrep v0.7.1
  Installing ripgrep v0.7.1
 Downloading grep v0.1.7
[... and all other dependencies]
   Compiling termcolor v0.3.3
[... and all other dependencies]
   Compiling ripgrep v0.7.1
    Finished release [optimized + debuginfo] target(s) in 114.67 secs
  Installing /home/jistone/.cargo/bin/rg
```

只需将`~/.cargo/bin`添加到您的`PATH`中，开始搜索！

```
$ PATH="~/.cargo/bin:$PATH"
$ rg 'extern crate'
sorter/src/main.rs
1:extern crate rand;
2:extern crate rayon;
```

注意:用 rust-toolset-7 构建的程序没有任何运行时依赖性，所以您可以自由地运行它们，而不用担心任何`scl enable`。

## 进一步阅读

要了解更多信息，尤其是如果您想首先了解 Rust 编程的更多信息，请访问以下资源:

*   [rust-toolset-7 在线文档](https://access.redhat.com/documentation/en-us/red_hat_developer_toolset/7/html-single/user_guide/#part-Rust_Toolset)
*   [Rust 项目的文件](https://doc.rust-lang.org/)
    *   也可以在 rust-toolset-7-rust-doc 包中找到，该包安装到`/opt/rh/rust-toolset-7/root/usr/share/doc/rust/html/index.html`
*   [来自板条箱的货物文件](http://doc.crates.io/)
    *   也可以在 rust-toolset-7-cargo-doc 包中获得，该包安装到`/opt/rh/rust-toolset-7/root/usr/share/doc/cargo/html/index.html`

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: October 26, 2017*