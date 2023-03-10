# 使用 Red Hat 开发工具进行跨语言链接时间优化

> 原文：<https://developers.redhat.com/blog/2020/03/18/cross-language-link-time-optimization-using-red-hat-developer-tools>

几个月前，LLVM 项目博客发表了一篇文章， *[“缩小差距:Rust 和 C/c++](http://blog.llvm.org/2019/09/closing-gap-cross-language-lto-between.html)T3 之间的跨语言 LTO”。在这篇文章中，他们解释了链接时优化可以通过优化整个程序来提高性能，比如在不同对象之间内联函数调用。由于 Rust 和 Clang 都使用 LLVM 进行代码生成，我们甚至可以在不同的编程语言之间实现这一优势。*

在[红帽开发者工具](https://developers.redhat.com/topics/developer-tools/)中，我们有 [Rust 和 LLVM 工具集](https://access.redhat.com/documentation/en-us/red_hat_developer_tools/1/)，它们可以很容易地一起用于跨语言链接时间优化(LTO)，所以让我们来试试吧。

## 设置

LLVM 博客中提到的一个警告是[工具链兼容性](https://doc.rust-lang.org/rustc/linker-plugin-lto.html#toolchain-compatibility)，因为跨语言 LTO 需要`rustc`和`clang`使用的 LLVM 库之间的紧密匹配。上游的 Rust 构建使用 LLVM 的私有副本，有时甚至是快照而不是最终版本，因此获得`clang`的相应构建可能具有挑战性。

Red Hat 的 Rust 工具集使用的共享 LLVM 库来自我们得到 Clang 的同一个 LLVM 工具集，所以我们总是知道这些编译器正在使用相同的代码生成后端，为 LTO 产生兼容的字节码。

在 Red Hat Enterprise Linux 8 (RHEL 8)上，这两个工具集在默认的 AppStream 存储库中都可用。我们可以安装他们的默认模块配置文件，以获得完整的工具链，防止生锈和磨损:

```
# yum module install rust-toolset llvm-toolset
```

在 Red Hat Enterprise Linux 7 上，工具集作为软件集合提供。在[获得](https://access.redhat.com/documentation/en-us/red_hat_developer_tools/1/html/using_rust_1.39_toolset/chap-rust-intro#sect-Rust-access)对`devtools`库的访问权后，我们可以安装每个工具集的当前版本:

```
# yum install rust-toolset-1.39 llvm-toolset-8.0
```

然后，我们可以通过启动一个启用 Rust 的新 shell 将它们添加到我们的工作中`PATH`,这隐式地将 LLVM 作为一个依赖项启用:

```
$ scl enable rust-toolset-1.39 bash
```

## 示例:内联求和

Rust 为 LTO 、`-C linker-plugin-lto`编写了一个[的特殊选项，C 调用 Rust 的示例命令，反之亦然。让我们将这个选项应用到一个简单的从 1 到 n 求和的例子中。这个例子有一个大多数优化器都知道的封闭形式`n(n+1)/2`,对于常量输入，它也可以编译成一个常量。](https://doc.rust-lang.org/rustc/linker-plugin-lto.html)

首先，在`main.c`中有我们的 C 代码:

```
#include <stdio.h>

int sum(int, int);

int main() {
    printf("sum(%d, %d) = %d\n", 1, 100, sum(1, 100));
    return 0;
}
```

然后，我们在`sum.rs`中有了我们的 Rust 代码:

```
#[no_mangle]
pub extern "C" fn sum(start: i32, end: i32) -> i32 {
    (start..=end).sum()
}
```

如果我们在没有 LTO 的情况下编译这个组合:

```
$ rustc --crate-type=staticlib -Copt-level=2 sum.rs
$ clang -c -O2 main.c
$ clang ./main.o ./libsum.a -o main
$ ./main
sum(1, 100) = 5050

```

在反汇编(`objdump -d main`)中我们可以看到`main`对`sum`进行了一次正常的函数调用:

```
00000000004005d0 <main>:
  4005d0:       50                      push   %rax
  4005d1:       bf 01 00 00 00          mov    $0x1,%edi
  4005d6:       be 64 00 00 00          mov    $0x64,%esi
  4005db:       e8 20 00 00 00          callq  400600 <sum>
  4005e0:       bf e8 06 40 00          mov    $0x4006e8,%edi
  4005e5:       be 01 00 00 00          mov    $0x1,%esi
  4005ea:       ba 64 00 00 00          mov    $0x64,%edx
  4005ef:       89 c1                   mov    %eax,%ecx
  4005f1:       31 c0                   xor    %eax,%eax
  4005f3:       e8 d8 fe ff ff          callq  4004d0 <printf@plt>
  4005f8:       31 c0                   xor    %eax,%eax
  4005fa:       59                      pop    %rcx
  4005fb:       c3                      retq
  4005fc:       0f 1f 40 00             nopl   0x0(%rax)

0000000000400600 <sum>:
...
```

现在，让我们用跨语言 LTO 来试试:

```
$ rustc --crate-type=staticlib -Clinker-plugin-lto -Copt-level=2 sum.rs
$ clang -c -flto=thin -O2 main.c
$ clang -flto=thin -fuse-ld=lld -O2 ./main.o ./libsum.a -pthread -ldl -o main
$ ./main
sum(1, 100) = 5050
```

这次在反汇编中我们可以看到调用没有了，取而代之的是一个简单的常数`0x13ba`，就是 5050:

```
00000000002a6250 <main>:
  2a6250:       50                      push   %rax
  2a6251:       bf b4 66 20 00          mov    $0x2066b4,%edi
  2a6256:       be 01 00 00 00          mov    $0x1,%esi
  2a625b:       ba 64 00 00 00          mov    $0x64,%edx
  2a6260:       b9 ba 13 00 00          mov    $0x13ba,%ecx
  2a6265:       31 c0                   xor    %eax,%eax
  2a6267:       e8 04 03 00 00          callq  2a6570 <printf@plt>
  2a626c:       31 c0                   xor    %eax,%eax
  2a626e:       59                      pop    %rcx
  2a626f:       c3                      retq
```

这个例子不足以让我们展示任何可测量的性能差异，但我希望它清楚地表明，在更大的程序中存在更大的优化机会——即使是在如此不同的语言之间。

## 例如:Firefox

正如 LLVM 博客中提到的，Firefox 是跨语言 LTO 的一个主要推动因素，这个项目大部分是用 C++编写的，但也越来越多地使用 Rust。两种语言都可以调用对方的 FFI，但是如果它们永远不能被内联，那么这些调用是有开销的，即使是在微不足道的情况下。

我不是测试浏览器性能的专家，但我能够在 Red Hat Enterprise Linux 8 上使用 Rust 和 LLVM 工具集尝试几次构建 Firefox 73.0。这是我的`mozconfig`文件:

```
export CC=clang
export CXX=clang++
export AR=llvm-ar
export NM=llvm-nm
export RANLIB=llvm-ranlib

ac_add_options --disable-elf-hack
ac_add_options --enable-release
ac_add_options --enable-linker=lld
```

然后，我通过添加`--enable-lto=thin`和`--enable-lto=cross`(也意味着“瘦”)来比较构建。大部分代码都内置在`libxul.so`中，下面是对生成的二进制文件的粗略比较:

| 低温氧化物 | `.text`尺寸 | `.data`尺寸 | `nm`符号 |
| - | One hundred and seventeen million seven hundred and seventy thousand one hundred and seventy-four | Four million two hundred and fifty-nine thousand six hundred and nine | Three hundred and three thousand six hundred and seventy-seven |
| 薄的 | One hundred and fifteen million six hundred and two thousand three hundred and fifty-one | Four million two hundred and forty-nine thousand nine hundred and sixty-eight | Two hundred and sixty-eight thousand three hundred and seventy-seven |
| 跨过 | One hundred and fifteen million six hundred and twenty-seven thousand four hundred and eighty-three | Four million two hundred and thirty-two thousand nine hundred and twenty-nine | Two hundred and sixty-seven thousand nine hundred and twenty-five |

根据这些标准，大部分的好处来自于启用瘦 LTO，在交叉构建中，代码大小变得稍微大了一些。尽管如此，在交叉构建中，符号数确实下降了一点，这粗略地表明，至少可能有更多的内联。

这个结果实际上对性能有什么影响？我将把这留给 Firefox 开发人员自己衡量。

## 结论

链接时优化是在大型构建中获得更多性能的好方法，但是如果您使用多种编程语言，它们之间的调用可能是一个障碍。现在，Clang、Rust 和它们共同的 LLVM 后端的结合使得跨语言 LTO 成为可能，我们可以看到一些真正的好处！

*Last updated: June 29, 2020*