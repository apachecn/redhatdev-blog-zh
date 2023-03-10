# MIR C 解释器和实时(JIT)编译器

> 原文：<https://developers.redhat.com/blog/2021/04/27/the-mir-c-interpreter-and-just-in-time-jit-compiler>

在过去的两年里，我一直在做一个项目，实现一个名为 MIR T1 的通用轻量级实时(JIT)编译器。该项目的基石是一个独立于机器的[中级中级代表](https://en.wikipedia.org/wiki/Intermediate_representation) (MIR)。

该项目的很大一部分由将 C 源代码编译成 MIR 的代码组成。因为 MIR 可以被解释并且是实时的，所以我很容易地将这个 C-to-MIR 编译器扩展为 C 解释器和 JIT 编译器。

我以前写过 MIR 项目的其他部分(参见 [*MIR:一个轻量级 JIT 编译器项目*](/blog/2020/01/20/mir-a-lightweight-jit-compiler-project/) )，但是我从未详细写过 C-to-MIR 编译器、C 解释器或 JIT。在这篇文章中，我想纠正这些疏漏。

## C-to-MIR 编译器的动机

最初，我启动了 MIR 项目，通过添加分层 JIT 编译来解决当前的 CRuby JIT 的缺点。

为了开始使用 MIR 项目实现一个 [Ruby](/blog/category/ruby/) JIT 编译器，我需要一个 C-to-MIR 编译器。用 C 实现的标准 Ruby 方法可以被翻译成 MIR，并且可以与从 Ruby [字节码](https://en.wikipedia.org/wiki/Bytecode)生成的 MIR 代码内联。组合后的代码可以被优化并及时转化为高性能的机器代码。

有几种方法可以实现 C-to-MIR 编译器。我可以实现一个 [LLVM](/blog/category/clang-llvm/) IR-to-MIR 编译器，或者编写一个 [GCC](https://gcc.gnu.org/) 端口来瞄准 MIR。但是这会产生对外部项目的依赖。这也不是一项简单的任务，而且可能会在将来造成维护负担。

另一方面，有些人很快就写出了小型 C 编译器。这里我可以提一下 [lacc](https://github.com/larmel/lacc) 、 [8cc](https://github.com/rui314/8cc) 和 [9cc](https://github.com/rui314/9cc) 编译器项目。

此外，我希望在未来扩展 C 语言，以标记 C-to-MIR 编译器应该分析代码并根据执行配置文件生成[推测和去优化代码](https://chrisseaton.com/truffleruby/deoptimizing)的程序点。例如，实现 CRuby 虚拟机的整数字节码的 C 代码检查操作数类型。它还检查整数的`plus`方法没有被重定义，没有溢出，不需要使用多精度数字，等等。所有这些检查将是我所说的标记程序点。

很难为 GCC 或 [Clang](/blog/category/clang-llvm/) 实现这样的扩展，很难获准将它们包含到 GCC 或 Clang 库中，也很难支持它们。

所以我决定先写自己的 C-to-MIR 编译器。它应该实现标准的 [C11](https://en.wikipedia.org/wiki/C11_(C_standard_revision)) ，没有很少使用的可选标准特性，比如变量数组、复数和原子数据。

主要的实现目标是简单性，而不是编译速度。这使得其他人更容易学习代码，并减少了维护代码所需的工作量。

## C 解释器和 JIT

C-to-MIR 编译器有一个 API 接口，我可以把编译器当作一个库来使用。另一方面，MIR 包括一个解释器和一个 JIT。通过将所有这些代码粘在一起，我发现创建一个 C 解释器和 JIT 很容易。我只需要写一个小的驱动程序。

将 C-to-MIR 编译器、MIR 解释器和 JIT 与驱动程序结合起来，产生了一个可执行文件`c2m`,它可以作为 C 解释器和 JIT 使用。

作为 MIR，JIT 为 x86-64、aarch64、ppc64(大小端)和 s390x 体系结构生成代码。也在这些架构上工作。

`c2m`选项类似于广泛使用的`cc`选项:`-E`、`-c`、`-S`、`-o <file>`、`-I*include_dir*`、`-D*macro*[=*value*]`、`-U*macro*`、`-L*library_dir*`、`-l*dynamic_library*`。

代替汇编代码，`c2m`生成文本的镜像表示。例如，该命令

```
c2m -S file1.c file2.c

```

创建文本镜像文件`file1.mir`和`file2.mir`。

代替目标文件，`c2m`生成二进制镜像表示。例如，该命令

```
c2m -c file1.c file2.c

```

创建二进制镜像文件`file1.bmir`和`file2.bmir`。

`c2m`生成一个链接的二进制镜像文件，而不是可执行文件。例如，该命令

```
c2m file1.c file2.c

```

创建一个链接的二进制镜像文件`a.bmir`(名称可通过`-o`选项更改)。

类似于在汇编程序和目标文件上使用`cc`，您可以在`c2m`命令行上使用文本和二进制镜像文件，例如:

```
c2m file1.mir file2.bmir

```

该命令创建一个链接的二进制镜像文件`a.bmir`。

有几个选项是针对`c2m`的。要解释 C 代码，使用`-ei`选项。例如，该命令

```
c2m echo.c -ei 1 2 Hello

```

将程序`echo.c`(应该包含一个`main`函数)编译成 MIR，MIR 将在解释器中执行。命令行上`-ei`之后的选项将作为参数传递给主函数(`argc`和`argv`)。

要在 MIR JIT 中执行相同的程序，使用`-eg`选项:

```
c2m echo.c -eg 1 2 Hello

```

`-el`选项请求延迟 JIT 生成。`c2m`在第一次调用函数之前，不会为函数生成机器码:

```
c2m echo.c -el 1 2 Hello

```

要查看 MIR JIT 编译器如何优化 MIR 代码并生成机器码，请使用`-dg`选项。但是要小心，因为它会输出很多信息。

## C-to-MIR 编译器内部详细信息

正如我之前所写的，C-to-MIR 编译器的主要实现目标是简单性。通常，我们可以通过将任务分成小的、可管理的子任务来实现简单性。甚至有一种极端的 [nano-pass 编译器设计](https://www.cs.indiana.edu/~dyb/pubs/nano-jfp.pdf)用于教育中研究编译器主题。

我对 C 编译器实现的方法是一个经典的划分，分为大小大致相同的四个阶段:预处理器、解析器、上下文检查器和 MIR 生成器，如图 1 所示。

[![The MIR compiler passes through four stages in order: preprocessor, parser, context checker, and MIR generator.](img/9f2f4575680ed8fb32ff0486591f7cad.png "mir2c")](/sites/default/files/blog/2020/11/mir2c.png)

Figure 1: The the MIR compiler passes through four stages.

我不使用任何工具，如 YACC 的编译器。虽然 ANSI C 标准语法[歧义](https://en.wikipedia.org/wiki/Ambiguous_grammar)，但是我不修改。我使用一个[解析表达式语法(PEG)](https://en.wikipedia.org/wiki/Parsing_expression_grammar) ，一个带有罕见回溯的手动解析器。它简单小巧，但比[确定性解析器](https://en.wikipedia.org/wiki/Deterministic_parsing)稍慢。

典型的 JIT，比如 Java 虚拟机，在线程中与代码执行并行运行。我还设计了在多线程环境中使用的 MIR JIT 和 C-to-MIR 编译器。因此，很容易让`c2m`并行编译不同的源文件，并为不同的 C 函数并行生成机器码。

您可以通过使用`-p`选项来定义`c2m`同时运行的并行任务的数量。例如，在这个调用中

```
c2m -p4 file1.c file2.c file3.c file4.c -eg *program arguments* 
```

`c2m`将首先创建四个线程。这些线程可以并行编译四个源文件。线程将从源文件队列中获取源文件，以生成镜像代码。

编译并链接生成的 MIR 代码后，四个新线程将为每个 MIR 函数进行优化并生成机器代码。类似地，这些新线程从镜像函数队列中获取镜像函数。所以，你可以同时为四个函数生成机器码。图 2 说明了`c2m`的并行操作。

[![](img/c69ee11e3bc2e35b7a1dc36bd4ad8436.png "c2m-parallel")](/sites/default/files/blog/2020/11/c2m-parallel.png)

Figure 2: Running compilation and machine-code generation in parallel.

## C 解释器和 JIT 的当前状态

C-to-MIR 编译器大部分已经实现。它通过了来自不同 C 测试套件的大约 1000 次测试。

大约一年前，我实现了一个重要的里程碑:成功的[引导](https://en.wikipedia.org/wiki/Bootstrapping_(compilers))。编译自己的源代码并生成一个镜像二进制文件。然后，这个镜像二进制文件的执行再次处理`c2m`源，并生成另一个镜像二进制文件。这两个 MIR 二进制文件是相同的:

```
cc -O3 -fno-tree-sra -std=gnu11 -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -ldl -o c2m
  ./c2m -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -o 1.bmir
  ./c2m 1.bmir -el -Dx86_64 -I. mir-gen.c c2mir.c c2mir-driver.c mir.c -o 2.bmir

```

为了更好地理解 MIR 项目的规模，图 3 中的饼状图显示了所有`c2m`源代码中的代码行，由[slocount](https://dwheeler.com/sloccount/)报告。

[![](img/56d42b3be8d0b7c7d850691419326d1c.png "sloc")](/sites/default/files/blog/2020/11/sloc.png)

Figure 3: Sizes of source code in the major c2m components.

## ABI 兼容性

ABI(应用程序二进制接口)可能非常复杂。例如，在某些架构上，C ABI 要求代码完全在寄存器中(例如参见 [aarch64 ABI](https://developer.arm.com/documentation/ihi0055/latest/) 或 [s390x ABI](http://legacy.redhat.com/pub/redhat/linux/7.1/es/os/s390x/doc/lzsabi0.pdf) )或部分在寄存器中(例如参见 [ppc64 BE ABI](https://refspecs.linuxfoundation.org/ELF/ppc64/PPC-elf64abi.html) 和 [ppc64 LE ABI](https://openpowerfoundation.org/?resource_lib=64-bit-elf-v2-abi-specification-power-architecture) )传递小结构，或者在不同类的寄存器中传递结构的不同部分，无论是整数还是浮点(例如在 [x86-64 ABI](https://gitlab.com/x86-psABIs/x86-64-ABI) )。

全面实施 ABI 不是一项简单的任务。一些 C 实现忽略了全调用 ABI 兼容性，或者省略了目标 ABI 的长双精度实现。因为 C-to-MIR 编译器将与 GCC/Clang 生成的代码通信，以在 Ruby 中实现 JIT，所以完全调用 ABI 兼容性是必须的。例如， [MRuby](https://github.com/mruby/mruby) 中的解释器值由一个小的 C 结构表示。

我花了相当大的努力来实现 C ABI 的四个目标。除了更改 C-to-MIR 代码之外，实施还需要更改 MIR 设计。我相信当前的设计将使将来添加一个具有不同 ABI 的新目标变得容易。

## MIR C 编译器与其他 C 编译器相比如何

C 有很多实现。为了与我的 C 实现进行比较，我选择了以下 C 编译器:

*   [GCC](https://gcc.gnu.org/) 版本 10.2.1。这个编译器应用广泛，是最具移植性的工业 C 编译器。
*   [Clang](http://llvm.org/)10 . 0 . 1 版本。这是一个流行的工业 C 编译器，具有支持许多目标的现代设计。
*   这是一个现代版本的可移植 C 编译器，最初发布于很久以前(1979 年)。它在很多目标上支持 C11，包括 x86_64，但大多数支持的目标都过时了。
*   [TCC](https://bellard.org/tcc/) 版本 0.9.27。tiny C11 编译器是一个两遍编译器，有自己的汇编器和链接器。它支持 i386/x86-64、arm、arm64 和 riscv64。
*   [Cproc](https://github.com/michaelforney/cproc) 。迈克尔·福尼的 C11 实现基于 [QBE](https://c9x.me/compile/) 编译器后端。QBE 可以被认为是一个具有类似 IR 和小型 SSA 优化管道的迷你 LLVM。QBE 支持 x86-64，并正在向 arm64 发展。比较 Cproc 有助于比较 MIR-generator 和 QBE 的性能，因为两者都可以用于轻量级 JIT 编译器。
*   [cparser](https://github.com/libfirm/cparser) 。这是一个基于非常复杂的后端的 C99 实现， [libFirm](https://pp.ipd.kit.edu/firm/) 版本 1.22。该编译器面向 i386/x86-64 和 32 位 arm、mips、riscv 和 sparc。
*   [lacc](https://github.com/larmel/lacc) 。这是一个仅支持 x86_64 的 C89 实现。
*   [Chibicc](https://github.com/rui314/chibicc) 。Rui Ueyama 最新的 C11 实现，针对教育目标；仅针对 x86_64。

我还尝试对以下 C 编译器进行基准测试:

*   [SCC](https://www.simple-cc.org/) 。不幸的是，在我的基准测试中，它因内部编译器错误而崩溃。
*   [LCC](https://github.com/drh/lcc) 。这个旧的 C 编译器在《T2:一个可重定目标的 C 编译器:设计和实现》一书中有所描述。我拒绝了它，因为它不支持 x86-64。
*   8cc 和 [9cc](https://github.com/rui314/9cc) 。很难使用这些编译器，并且检查它们可能没有什么意义，因为 Chibicc 是它们开发的延续。

### 生成的代码性能

为了比较编译器生成的代码的性能，我使用了一组 14 个[基准](https://github.com/vnmakarov/mir/tree/master/c-benchmarks)，其中大部分来自[旧计算机语言枪战](https://dada.perl.it/shootout/)。我选择这些微基准是因为由所有编译器来编译更严肃的基准是有问题的。我在标杆管理方面的长期经验告诉我，无论你使用什么样的标杆，人们总是会批评你的选择。我只有一个借口:有信息总比没有好。

图 4 显示了 Fedora Core 32 下 i9-10900 上每个编译器生成的代码的平均和几何相对速度(通过测量 CPU 时间)。基线是 GCC 用`-O2`生成的代码。对于`c2m`来说，执行时间包括了 C-to-MIR 编译器和 JIT 的工作，但是这个工作只是整个执行的一小部分。我将每个基准测试运行了三次，并选择了最佳时间。

[![The MIR C interpreter and Just-in-Time (JIT) compiler](img/1557f343c07f56497a17eb52bd3e5ffd.png "commet-lake-speed")](/sites/default/files/blog/2020/11/commet-lake-speed.png)

Figure 4: Relative speed of code generated by the compilers tested on an Intel i9 processor.

人们有时批评我很少对 AMD CPUs 进行基准测试。图 5 显示了 AMD 锐龙 7 3800x 上的基准测试结果，基本相同。

[![](img/3d5ca7234fa58dce4c3e63a760ac10ff.png "ryzen-speed")](/sites/default/files/blog/2020/11/ryzen-speed.png)

Figure 5: Relative speed of code generated by the compilers tested on an AMD processor.

### 批量编译速度

为了获得批量编译的速度，我将 [bzip2](https://people.csail.mit.edu/smcc/projects/single-file-programs/) 源文件编译为一个文件。该文件大约有 6500 行 C 代码。我在非并行模式下使用了`c2m`来生成一个 MIR 二进制文件。还是那句话，我把源文件编译了三遍，选择了最好的时间。图 6 显示了编译器相对于`gcc -O2`速度的编译速度。

[![](img/255659d70237ac111283cff2427008b5.png "commet-lake-compilation-speed")](/sites/default/files/blog/2020/11/commet-lake-compilation-speed.png)

Figure 6: Relative speed of bulk compilation.

速度的差异非常显著，所以我必须在图表中使用对数刻度。微型 C 编译器有着惊人的速度。如果它生成了更好的代码，它就可以作为 JIT 编译器，以 C 语言作为它的接口。

除了批量编译速度之外，编译器启动时间对于动态编程语言的 JIT 编译器也非常重要，因为即时方法非常小。GCC 和 Clang 是这里表现最差的。如果你对这个话题感兴趣，可以看看我的[上一篇关于轻量级 JIT 编译器项目的文章](https://developers.redhat.com/blog/2020/01/20/mir-a-lightweight-jit-compiler-project/)。

### 编译器代码大小

编译器代码大小的差异甚至更大。最小的编译器(Chibicc)几乎比最大的编译器(Clang)小 1000 倍，如图 7 所示。

[![](img/ac940dac46cc857c212dfe599a4f1692.png "commet-lake-size")](/sites/default/files/blog/2020/11/commet-lake-size.png)

Figure 7: Relative code sizes of the compilers themselves.

一些编译器有一个很大的 bss 部分(例如，在 Cproc 上是 17MB)。因此，对于大小计算，我只使用了文本和数据部分的大小。

我使用了来自 Fedora Core 32 发行版的 GCC 和 Clang。对于所有其他编译器，我都是通过优化来构建的(也就是说，在发布模式下)。

对于 GCC，我采用了`gcc`和`cc1`可执行文件。对于 Clang，我使用了`clang`及其库`libclang-cpp`和`libLLVM`。对于 PCC，我使用了它的可执行文件`cc`、`ccom`和`cpp`。对于 Cproc，我使用了`cproc`、`cproc-qbe`和`qbe`。所有其他编译器都由一个可执行文件组成。

## C-to-MIR 编译器的未来计划

我的短期计划是在 2021 年底之前提供 MIR 项目的第一个版本，包括 C-to-MIR 编译器。

在以下版本中，我希望:

*   提高 C 到 MIR 的编译速度
*   改进 C-to-MIR 生成的代码
*   将 MIR 移植到更多目标(64 位 riscv 和 mips64 Linux 和苹果 M1 macOS)
*   在 C 和 MIR 级别实现推测/去优化扩展

尽管 C-to-MIR 的编译速度从来不是主要目标，但它的速度与其他 C 编译器相比是相当有竞争力的。我发现有些人更喜欢使用 C 语言而不是 MIR 来实现他们的 JIT。这使得提高 C-to-MIR 编译速度成为一项重要任务。因此，我把它作为第一要务。

有人问我有没有可能做一个正规的 C 编译器，而不是 JIT。如果使用现有的汇编程序，编写编译器并不是一件困难或大的任务。要制作一个编译器，我们只需要发出汇编代码，而不是当前的机器码。然而，要做到这一点，需要发出调试信息(例如，以 [dwarf](https://en.wikipedia.org/wiki/DWARF) 格式)并生成[位置无关代码(PIC)](https://en.wikipedia.org/wiki/Position-independent_code) 来为共享库生成代码。

## 结论

MIR 项目中实现的 C JIT 编译器和解释器显示了具有竞争力的生成代码和编译速度。它有潜力用于 C 语言的脚本编写，并作为 JIT 编译器来实现不同的编程语言。希望也能用于教育目的。

*Last updated: October 14, 2022*