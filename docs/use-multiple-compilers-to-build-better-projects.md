# 使用多个编译器来构建更好的项目

> 原文：<https://developers.redhat.com/blog/2021/05/07/use-multiple-compilers-to-build-better-projects>

出于多种原因，开发人员通常只用一个编译器来编译他们正在进行的项目。在[红帽企业 Linux 8](/products/rhel/overview) 上， [C 和 C++](/topics/c) 的系统编译器是 GNU 编译器集合(GCC) 8，更新的版本可以通过 GCC 工具集获得。

然而，有几个原因可以解释为什么你也可以用 [Clang](https://clang.llvm.org/) 来构建你的项目。红帽企业 Linux 8 提供了 [LLVM](https://llvm.org/) 工具集，其中包含了 [Clang](/blog/category/clang-llvm/) 。

在本文中，我们将看看为什么一个人可能会使用多个编译器。我们将关注一个系统，在这个系统中，GCC 是当前的默认编译器，而 Clang 是主要的替代编译器。

## 检测编译器特定的行为

当您尝试使用项目通常不使用的编译器来编译项目时，最常见的问题之一是项目对正在使用的编译器的假设。这些假设可能出现在项目中的许多地方，从命令行选项到支持的功能，再到编译器扩展。

一般来说，所有这些编译器特有的特性都应该避免，除非项目明确地只支持提供非标准特性的编译器。如果您依赖于特定的编译器或非标准特性，那么这种依赖性应该记录在项目的构建文档中，并且最好由构建系统强制执行。

为了确保一个项目可以用其他编译器构建，定期用新的编译器构建它是很有用的。这种构建还可以检测实现定义的行为、未定义的行为以及(在极少数情况下)编译器错误。

其他潜在的问题包括支持的`__attribute__((foo))`指令(或 C++的`[[attribute-name]]`)的差异，支持的语言标准的差异，以及未实现的编译器内置的使用。

## 从多个编译器获得不同的错误信息

使用现代编译器的最大好处之一是它会根据开发人员传递的命令行选项生成警告和错误消息。修复这些警告可以提高代码质量，增加可移植性，减少错误。

一个常见的问题是，并非所有编译器都接受相同的命令行参数，因此项目必须在配置时检查它们。根据项目的需要，大多数人最终会为他们构建的每个编译器维护一个编译器选项列表。

因为判断编译器是否支持命令行选项是如此普遍，所以通常的构建系统都有执行这些检查的内置方式。

CMake 使用`[check_c_compiler_flag()](https://cmake.org/cmake/help/v3.14/module/CheckCCompilerFlag.html)`或`[check_cxx_compiler_flag()](https://cmake.org/cmake/help/latest/module/CheckCXXCompilerFlag.html)`功能:

```
include(CheckCCompilerFlag)
check_c_compiler_flag("-Werror=header-guard", CC_SUPPORTS_HEADER_GUARD)
check_c_compiler_flag("-Werror=logical-op", CC_SUPPORTS_LOGICAL_OP)

```

Meson 使用编译器对象的`get_supported_arguments()`函数:

```
test_cflags = [
    '-Werror=header-guard',
    '-Werror=logical-op'
]

cc = meson.get_compiler('c')
supported_flags = cc.get_supported_arguments(test_cflags)

```

当然，还有更多的构建系统，但是它们都或多或少地提供了处理这个问题的优雅方式。

因此，在为我们的构建配置了检查编译器选项之后，我们就可以用 Clang 和 GCC 或者其他编译器成功地构建了。

下面的代码显示了用 GCC 构建时的一个错误，使用了`-Wlogical-op`命令行选项。Clang 不支持这个选项，并且对逻辑运算符的可疑使用保持沉默:

```
int main(int argc, char **argv) {
    if (argc > 0 && argc > 0) {
        return 1;
    }
    return 0;
}

```

GCC 10 使用`-Wlogical-op`的输出如下所示:

```
test.c: In function ‘main’:
test.c:2:16: warning: logical ‘and’ of equal expressions [-Wlogical-op]
    2 |     if (argc > 0 && argc > 0) {
      |         ~~~~~~~~~^~~~~~~~~~~

```

**注**:参见 godbolt.org 上的[代码。](https://godbolt.org/z/Mz191P4oc)

正如已经提到的，Clang 忽略了这个(潜在的)问题。但是如果我们在之前的测试中包含了下面的头文件，GCC 将会对头文件保护中的错别字保持沉默，而 Clang 将会指出这个错误。像这样的头文件保护在 C 和 C++代码中仍然比较常见，而且问题通常很难发现:

```
#ifndef __TEST_HEADER_H__
#define __TEST_HEADRE_H__

/* ... Code ...*/

#endif

```

带`-Werror=header-guard`选项的铿锵声告诉我们割台防护装置损坏的情况:

```
In file included from test.c:3:
./test.h:1:9: error: '__TEST_HEADER_H__' is used as a header guard here, followed by #define of a different macro [-Werror,-Wheader-guard]
#ifndef __TEST_HEADER_H__
        ^~~~~~~~~~~~~~~~~

./test.h:2:9: note: '__TEST_HEADRE_H__' is defined here; did you mean '__TEST_HEADER_H__'?
#define __TEST_HEADRE_H__
        ^~~~~~~~~~~~~~~~~
        __TEST_HEADER_H__

```

**注**:参见 godbolt.org 上的[代码。](https://godbolt.org/z/87668f1sE)

GCC 无法检测到这个问题。我在本节中展示的不同行为的例子只是人们可以找到的许多例子中的两个。

实际上，如果编译器只发出警告而不发出错误，那么它们的有用性取决于开发人员实际查看编译器的输出。在开发期间将`-Werror`传递给编译器也很有用，因为它使编译器将所有警告视为错误。

## 静态分析

Clang 的一个众所周知的部分是静态分析器。静态分析可以用来“静态地”分析程序的某些方面，也就是说，在运行之前。这允许更彻底的检查，因为分析器花费的时间没有编译器花费的时间重要。

因为它不需要人工干预，静态分析在[持续集成(CI)](/topics/ci-cd) 中特别有用，我们可以在每次推送到存储库时使用它。然而，通常很难保持代码 100%没有来自静态分析器的报告。这部分归因于静态分析器的彻底性:它们发现了许多编译器没有发现的问题，但其中一些是假阳性，由分析器不知道的隐式不变量引起。以断言的形式对这些假设进行编码通常会提高代码的清晰度，也会让分析者感到高兴。

## 消毒剂

Clang 附带了几个*杀毒器*，它们在运行时检测编译后的程序。杀毒程序通常用于需要开发人员重新运行程序并获取有关问题行为的信息的问题，这需要额外的时间。它们也用于那些不会中止程序但会导致后续问题的问题，比如整数溢出或访问未初始化的数据。

Clang 中支持的最有用和最常见的杀毒程序如下:

*   [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) 可用于检测各种内存问题，如空指针取消引用、释放后使用和双/无效释放。可以通过`-fsanitize=address`启用。
*   [内存初始化器](https://clang.llvm.org/docs/MemorySanitizer.html)可以用来检测对未初始化内存的访问。可以通过`-fsanitize=memory`启用。
*   [ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) 检测多线程程序中的数据竞争。可以通过`-fsanitize=thread`启用。
*   [未定义行为初始化器](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)检测各种未定义的行为。可以通过`-fsanitize=undefined`启用。

GCC 支持所有这些杀毒软件，除了内存杀毒软件。这些都是重要的杀毒程序，但是 GCC 和 Clang 都支持许多更细粒度的检查，所以检查它们的文档以获得有用的杀毒程序和它们的选项可能是值得的。

如果软件有一个测试套件(它应该有)，并且该测试套件是以持续集成的方式运行的(它应该有)，那么用本节提到的一些杀毒程序来编译测试套件是非常有意义的，并且不会像在 Valgrind 中运行程序那样增加运行时间。

然而，如果开发人员在日常工作中启用一个或另一个杀毒软件，以尽快捕捉真实世界数据中的错误，肯定也是有帮助的。

要更深入地了解杀毒软件并与超级有用的 Valgrind 工具进行比较，请查看 Jan Kratochvil 最近关于这个主题的文章。

## 叮当声模糊器

一个 *fuzzer* 是一个为测试中的库生成随机输入的工具。模糊测试有助于在任何类型的文件解析器中发现错误和崩溃。Clang 包含 [libFuzzer](https://llvm.org/docs/LibFuzzer.html) ，可以用于这种测试。

fuzzer 将不断向测试中的库提供新的输入，直到发现一个 bug，所以在最好的情况下，fuzzer 程序的运行会无限期地持续下去。这种模糊化的情况不能以自动化的方式使用，但是对于开发人员手动使用仍然是非常有价值的。

关于在 RHEL 上用 llvm 工具集使用 Clang 的 fuzzer 的介绍，请阅读 Tom Stellard 的这篇文章。

## 链接时间优化(LTO)版本

越来越多的发行版正在转向使用链接时间优化(LTO)构建。在这些版本中，编译器不发出本机目标代码，而是发出它的中间表示形式(IR)。然后将 IR 交给链接器，链接器可以应用模块间优化。

LTO 还通过移除未使用且未明确标记为外部可见的符号来帮助识别符号可见性问题。这可以帮助您避免意外导出非公共使用的符号。

GCC 和 LLVM 都支持 LTO 构建，以及一些配置选项来满足不同用例的需求。有关这些选项以及 LTO 编译期间编译器内部工作的更多详细信息，请参考与此主题相关的 [GCC](https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html) 和 [LLVM](https://llvm.org/docs/LinkTimeOptimization.html) 文档。

## Clang 中的控制流完整性

[Clang 的控制流完整性(CFI)](https://clang.llvm.org/docs/ControlFlowIntegrity.html) 是一种特殊类型的杀毒软件，需要使用链接时间优化(LTO)。您可以通过`-fsanitize=cfi`启用它。

CFI 允许编译程序的检测来检测某些形式的未定义行为，并在这些情况下中止程序。CFI 杀毒软件支持不同的方案，而且它们通常都经过了充分的优化，因此甚至可以在发布版本中启用。例如，众所周知谷歌在 Android 上做了这个[。](https://source.android.com/devices/tech/debug/cfi)

这是目前仅在 Clang 上可用的安全加固的一个例子。

## 结论

不同的编译器各有优缺点。用不同的编译器测试你的项目将确保你不依赖于特定的编译器行为——甚至是错误。许多开源项目已经有了一个 CI 管道，它利用了不止一个编译器、配置或平台。这是理想的，如果对他们有意义的话，你应该试着对你所有的项目都这样做。如果不能，至少偶尔使用另一个编译器是有意义的，或者尝试将这种实践集成到您的本地开发工作流中。

在您的测试套件和静态分析器中定期使用杀毒程序(或者甚至通过 CI 中的特殊构建)是提前发现 bug 的一个很好的方法。同样，不同的工具会显示代码中不同的缺陷。从长远来看，仔细评估它们是值得的。尝试其他工具，每天使用你觉得最舒服的工具。但是要记住其他的选择，尽你所能实现自动化。

*Last updated: October 14, 2022*