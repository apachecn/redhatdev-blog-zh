# 用 GCC 11 检测内存管理缺陷，第 1 部分:理解动态分配

> 原文：<https://developers.redhat.com/blog/2021/04/30/detecting-memory-management-bugs-with-gcc-11-part-1-understanding-dynamic-allocation>

在 C 和 C++ 程序中，内存管理错误是最难发现的，也是最容易被利用的目标。这些错误很难调试，因为它们涉及到程序中的三个不同的位置，这三个位置通常相距很远，并且由于使用指针而变得模糊不清:内存分配、已分配内存的使用以及通过取消分配将内存释放回系统。在这篇由两部分组成的文章中，我们将关注 [GNU 编译器集合(GCC) 11](https://gcc.gnu.org/) 的增强，这些增强有助于检测这些影响动态分配内存的错误子集。这里讨论的增强已经对 GCC 核心进行了改进。David Malcolm 在他的文章[GCC 11](/blog/2021/01/28/static-analysis-updates-in-gcc-11)中的静态分析更新中介绍了 GCC 静态分析器的相关改进。

在整篇文章中，我为那些想尝试的人提供了到[编译器浏览器](https://godbolt.org)上的代码示例的链接。您将在每个示例的源代码上方找到链接。

**注**:阅读本文后半部分:[用 GCC 11 检测内存管理 bug，第 2 部分:解除分配函数](/blog/2021/05/05/detecting-memory-management-bugs-with-gcc-11-part-2-deallocation-functions/)。

## 内存分配策略概述

让我们先快速分析一下主要的内存管理错误。C 和 C++概述了四大类存储分配策略:

*   **自动**:这个策略在函数的堆栈上分配对象。除了不标准的、不鼓励的、但仍被广泛使用的`alloca()`函数，自动对象在声明时就被分配了。然后，它们通常通过名称引用，除非通过引用传递给其他函数，或者使用指针指向数组元素。顾名思义，自动对象在声明它们的块的末尾被自动释放。由`alloca()`分配的对象在函数返回时被释放。
*   **动态**:这个策略通过显式调用分配函数来分配堆上的对象。为了避免内存耗尽，动态分配的对象必须通过显式调用相应的解除分配函数来解除分配。
*   静态的(Static):这个策略分配在程序运行期间持续的命名对象。它们永远不会超出作用域，因此在程序执行期间它们永远不会被释放。
*   **Thread** :类似于 static，但是在持续时间上仅限于单个执行线程。该策略中的对象会在创建它们的线程终止时自动释放。

在这四种策略中，最常见但也是最隐蔽的一类问题是动态分配。这种分配形式是本文的主题。可以肯定的是，大量的错误也与自动存储有关(想想未初始化的读取，或者在一个局部变量超出作用域后通过一个在它还活着时获得的指针来访问它)，但是我们将在另一个时间讨论这些。

## GCC 11 中的新命令行选项

在深入研究 GCC 11 可以检测的动态内存管理错误的细节之前，让我们快速总结一下控制检测的命令行选项。默认情况下，所有选项都是启用的。尽管它们在启用优化的情况下性能最佳，但它们并不需要这样做。

GCC 11 提供了两个新选项，并显著增强了几个版本中已有的一个选项:

*   控制对一般内存分配和释放函数调用之间不匹配的警告。这个选项是 GCC 11 中的新功能。
*   在 C++中，`[-Wmismatched-new-delete](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-Wmismatched-new-delete)`控制对`operator new()`和`operator delete()`的调用不匹配的警告。这个选项在 GCC 11 中也是新的。
*   控制对未被动态分配函数返回的指针的无效释放尝试的警告。该选项在 GCC 11 中得到增强。

## 动态内存管理功能

C 语言中最著名的动态内存管理函数是`calloc()`、`malloc()`、`realloc()`和`free()`。但是他们不是唯一的。除了 C89 的这些功能，C99 还引入了`aligned_alloc()`。POSIX 添加了一些自己的分配函数，包括`strdup()`、`strndup()`和`tempnam()`等等。c 库实现通常提供自己的扩展。例如，FreeBSD、Linux 和 Solaris 都定义了一个名为 [`reallocarray()`](https://www.freebsd.org/cgi/man.cgi?query=reallocarray) 的函数，它是`calloc()`和`realloc()`的混合体。所有这些分配函数返回的指针都必须传递给`free()`进行释放。

除了动态分配原始内存的函数之外，其他一些标准 API 也分配和释放其他资源。例如，`fopen()`、`fdopen()`和 POSIX [、`open_memstream()`、](https://pubs.opengroup.org/onlinepubs/9699919799/functions/open_memstream.html)创建并初始化`FILE`对象，然后必须通过调用`fclose()`来处理这些对象；`popen()`函数也创建`FILE` s，但是这些必须通过调用`pclose()`来关闭。类似地，POSIX [`newlocale()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/newlocale.html) 和 [`duplocale()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/duplocale.html) 函数创建必须通过调用 [`freelocale()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/freelocale.html) 销毁的语言环境。

最后，许多第三方库和程序定义了自己的函数来分配原始内存或初始化驻留在已分配内存中的各种类型的对象。这些函数通常将指向对象的指针返回给它们的客户端。其中最简单的可以通过调用`free()`直接释放，但是大多数 API 依赖它们的客户端通过将对象“返回”到适当的释放函数来销毁和释放对象。

所有这些 API 组都有一个共同的主题:每个组中的分配函数返回一个用于访问对象的指针，重要的是，这个指针最终必须传递给同一个组中适当的解除分配函数。`malloc()`的结果必须传递给`free()`，`fopen()`的结果必须传递给`fclose()`。因此，将`fopen()`的结果传递给`free()`是一个 bug，就像在从`malloc()`返回的指针上调用`fclose()`一样。此外，在 C++中，给定形式的`operator new()`的结果——无论是普通的还是数组的——都必须由相应形式的`operator delete()`释放，而不是通过调用`free()`或`realloc()`。

## 将分配与取消分配相匹配

调用错误的 deallocation 函数来释放由不同组的分配函数分配的资源通常会导致内存损坏。呼叫可能会立即崩溃，有时甚至会出现有用的消息，或者可能会在稍后返回给呼叫者，并在与无效呼叫无关的区域崩溃。或者，解除分配功能可能根本不会崩溃，而是会覆盖一些数据，从而导致以后出现不可预知的行为。当然，我们希望检测并防止这些错误，不仅仅是在它们进入产品发布之前，而且最好是在代码开发期间，在它们被提交到代码库之前。挑战在于如何让我们的工具——编译器或静态分析器——知道必须使用哪些函数来释放由其他函数分配的每个对象。

对于标准功能的子集来说，语义和关联可以并且经常被嵌入到工具本身中。例如，GCC 知道标准 C 和 C++动态内存管理函数的效果，以及哪一个与哪一个相匹配，但是它不知道像`fopen()`和`fclose()`这样的`<stdio.h>`函数，也不知道实现定义的扩展。另外，GCC 对用户定义的函数一无所知。

## 属性 malloc

输入[属性`malloc`](https://gcc.gnu.org/onlinedocs/gcc/Common-Function-Attributes.html#index-malloc-function-attribute) ，或者更准确地说，是在 GCC 11 中实现的对它的增强。在传统形式中，该属性不带参数，只是让 GCC 知道它所应用的函数会像`malloc()`一样返回动态分配的内存。GCC 使用该属性对返回内存的内容进行别名假设，并发出更高效的代码。GCC 11 扩展了属性`malloc`,以接受一个或两个参数:释放已分配对象所调用的解除分配函数的名称，以及可选地，指针必须传递到的位置参数编号。同一个分配函数可以与任意数量的解除分配函数配对。例如，下面的声明将`fclose()`指定为`fopen()`、`fdopen()`、`fmemopen()`和`tmpfile()`的释放器，并将`pclose()`指定为`popen()`的唯一释放器。

```
int fclose (FILE*);

int pclose (FILE*);

__attribute__ ((malloc (fclose, 1))))
FILE* fdopen (int);

__attribute__ ((malloc (fclose, 1))))
FILE* fopen (const char*, const char*);

__attribute__ ((malloc (fclose, 1))))
FILE* fmemopen (void *, size_t, const char *);

__attribute__ ((malloc (pclose, 1))))
FILE* popen (const char*, const char*);

__attribute__ ((malloc (fclose, 1))))
FILE* tmpfile (void);
```

理想情况下，`<stdio.h>`和其他 C 库头文件中的声明应该用属性`malloc`来修饰，就像刚才显示的那样。Linux 上 glibc 的补丁已经提交，但是还没有被批准。在此之前，您可以将前面的声明添加到自己的头中，以实现相同的检测。完整补丁也可以从 sourceware.org 下载[。](https://sourceware.org/pipermail/libc-alpha/2021-January/121527.html)

GCC proper 和集成静态分析器都利用这个新属性来发出类似的警告。静态分析器以增加编译时间为代价检测更广泛的问题。

## 检测不匹配的释放

GCC 11 中的许多警告使用新属性`malloc`来检测各种内存管理错误。`-Wmismatched-dealloc`选项控制释放调用的警告，该调用带有从不匹配的分配函数返回的参数。例如，给定上一节中的声明，在下面的函数中对`fclose()`的调用被诊断出来，因为传递给它的指针是从一个与它没有关联的分配函数返回的:`popen()`。 [`popen_pclose`](https://godbolt.org/z/Wqfx93) 的例子说明了这是如何工作的:

```
void test_popen_fclose (void)
{
   FILE *f = popen ("/bin/ls");
   // use f
   fclose (f);
}

```

编译器警告是:

```
In function 'test_popen_fclose':
**warning**: 'fclose' called on pointer returned from a mismatched allocation function [**-Wmismatched-dealloc**]
21 | fclose (f);
   | ^~~~~~~~~~
**note**: returned from 'popen'
19 | FILE *f = popen ("/bin/ls", "r");
   | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

## 第一部分的结论

寻找这篇文章的第二部分[，在那里我将描述更多检测动态分配错误的选项。我将以可能导致假阳性或假阴性鉴定的情况来结束。](https://developers.redhat.com/blog/2021/05/05/detecting-memory-management-bugs-with-gcc-11-part-2-deallocation-functions/)

*Last updated: October 14, 2022*