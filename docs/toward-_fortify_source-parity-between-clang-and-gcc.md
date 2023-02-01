# Clang 和 GCC 之间的奇偶校验

> 原文：<https://developers.redhat.com/blog/2020/02/11/toward-_fortify_source-parity-between-clang-and-gcc>

GCC 结合 glibc 可以通过标准 C 库函数检测缓冲区溢出的实例。当用户通过`-D_FORTIFY_SOURCE={1,2}` *预处理器*标志和大于或等于`-O1`的优化级别时，在调用例如`strcpy`时，使用函数的替代*增强*实现。根据函数及其输入，这种行为可能会导致编译时错误，或者在执行时触发运行时错误。(关于这个特性的更多信息，这里有一篇关于这个主题的[优秀博客文章。](https://access.redhat.com/blogs/766093/posts/1976213)

Clang 加 glibc 二人组呢？本文深入探讨了`-D_FORTIFY_SOURCE`的用法，并讨论了应用于 Clang 以实现特性对等的补丁。

# 第一眼

让我们通过编译器可移植性棱镜来看看`-D_FORTIFY_SOURCE`。由于该特性依赖于预处理器定义，因此它应该只涉及预处理器选择。兼容性应该是免费的，对吗？

不完全是。按照`glibc`头文件中的定义，人们可以很快发现实现安全检查所依赖的编译器特定的函数调用。对标题的[仔细研究](https://gatherer.wizards.com/Pages/Card/Details.aspx?multiverseid=29727)导致在标题中的某个点使用了以下内置，仅在`-D_FORTIFY_SOURCE`开启时包含:

*   _ _ 内置常量 _p
*   _ _ 内置对象大小
*   -= =破烂熊乐园倾情奉献= =-本字幕仅供学习交流，严禁用于商业用途
*   __builtin___memmove_chk
*   __builtin___mempcpy_chk
*   __builtin___memset_chk
*   __builtin___snprintf_chk
*   __builtin___sprintf_chk
*   __builtin___stpcpy_chk
*   __builtin___strcat_chk
*   __builtin___strcpy_chk
*   __builtin___strncat_chk
*   -= =破烂熊乐园倾情奉献= =-本字幕仅供学习交流，严禁用于商业用途
*   __builtin___vsnprintf_chk
*   __builtin___vsprintf_chk

这些是编译器*内置*，正如前缀`__builtin__`所暗示的，这意味着要么编译器知道它们并提供自己的实现/处理，要么编译(或链接)过程将失败。所以为了支持`-D_FORTIFY_SOURCE`，编译器必须支持这些内置。所有这些函数(除了`__builtin_constant_p`和`__builtin_object_size`)的后缀都是`_chk`，这表明它们是 libc 中相应函数的强化版本。

让我们更深入地看看这些函数。

# 必需的编译器内置

以下编译器内置是`-D_FORTIFY_SOURCE`所必需的。

### `__builtin_object_size(obj, type)`

这个内置函数很复杂。感兴趣的读者可能想看看它的[在线文档](https://gcc.gnu.org/onlinedocs/gcc/Object-Size-Checking.html)。作为一个简短的总结，我们假设这个函数试图在编译时计算`obj`的分配大小，然后返回它。如果这个过程失败，它返回`-1`。

`type`参数控制这个函数语义的细节。以下定义在`cdefs.h`中可用:

```
#define __bos(ptr) __builtin_object_size (ptr, __USE_FORTIFY_LEVEL > 1)
```

### `__builtin_constant_p(obj)`

如果`obj`的值在编译时(优化后)已知，该函数返回`1`，否则返回`0`。以下代码摘自`glibc`版本 2.30 中的`stdio2.h`，展示了一个使用示例:

```
__fortify_function __wur char *
fgets (char *__restrict __s, int __n, FILE *__restrict __stream)
{
  if (__bos (__s) != (size_t) -1)
    {
      if (!__builtin_constant_p (__n) || __n <= 0)
  return __fgets_chk (__s, __bos (__s), __n, __stream);

      if ((size_t) __n > __bos (__s))
  return __fgets_chk_warn (__s, __bos (__s), __n, __stream);
    }
  return __fgets_alias (__s, __n, __stream);
} //*

```

这段代码读作:

如果我们可以计算出`fgets`的第一个参数的基本对象大小(bos ),那么如果第二个参数在编译时未知，就使用`__fgets_chk`函数。如果第二个参数大于对象尺寸，那么使用`__fgets_chk_warn`。否则，我们知道(在编译时)调用是安全的，并且通过`__fgets_alias`调用原始函数。

### `__builtin___memcpy_chk(dest, src, n, dest_size)`

额外的`dest_size`参数用于与`n`进行比较。`dest_size`可以是`-1`，这意味着它的值在编译时是未知的。它可以有一个正值，在这种情况下，它意味着“在由`dest`指向的位置之后剩余的分配字节数是`dest_size`当`dest_size`为正值且小于`n`时，在编译时或运行时会发出一个错误。

其他`__bultin__*_chk`内建基于目标缓冲区的编译器计算的对象大小和实际的副本大小进行类似的检查。

# 金属撞击兼容性

快速查看了一下 Clang 支持的内置，结果发现`-D_FORTIFY_SOURCE=2`需要的内置都是 Clang 支持的。这是一个很好的特性:这意味着在编译 C(或 C++)应用程序时，您可以将预处理器标志传递给 Clang，它可以很好地编译。事实上，火狐已经用 Clang 和那面旗构建了[，所以它确实*编译*很好。](https://searchfox.org/mozilla-central/source/build/moz.configure/toolchain.configure#1540-1551)

但是我们得到额外的保护了吗？深入查看 Clang 的源代码后，答案更加细致入微。基于`[Sema::checkFortifiedBuiltinMemoryFunction](https://github.com/llvm/llvm-project/blob/release/9.x/clang/lib/Sema/SemaChecking.cpp#L315)`的主体，只有在编译时 size 参数和对象大小都已知的情况下才执行检查。否则，不执行任何检查。这个序列不同于 GCC 行为，在 GCC 行为中，在这种情况下会生成对`__memcpy_chk`的调用。

看看[这个片段](https://godbolt.org/z/xXfpNZ)说明了 GCC 的行为。`memcpy`调用的 size 参数是一个运行时值，但是 destination 和 source 参数在编译时都有一个已知的大小。GCC 内部表示显示对`__builtin___memcpy_chk`的调用降低到`__memcpy_chk`。另一方面，Clang 只是[发出对`memcpy`](https://godbolt.org/z/TBDbY6) 的常规调用。

# 修补叮当声

深入研究 Clang 的代码可以发现，每当它遇到对`memcpy`的调用时，这个调用就会被对 [LLVM 的内置`llvm.memcpy`T5 的调用所取代。不幸的是，`-D_FORTIFY_SOURCE={1,2}`所做的是用加强的实现来保护`memcpy`的内联定义。这就是 Clang 应该使用的实现。这个](http://llvm.org/docs/LangRef.html#llvm-memcpy-intrinsic)[补丁](https://reviews.llvm.org/D71082)通过强制的额外测试实现了这个额外的行为。

为了验证整个方法，我写了一个[最小测试套件](https://github.com/serge-sans-paille/fortify-test-suite/)来增强编译器。GCC 通过了设计的*，而 Clang 9 没有。然而，使用 Clang 的树顶版本(`346de9b6`在撰写本文时)，测试套件现在通过得很好:*

```
(sh) make check-gcc
[...]
===== GCC OK =====
(sh) PATH=/path/to/clang:$PATH make check-clang
[...]
===== CLANG OK =====

```

# 结论

当以功能对等为目标时，魔鬼就在细节中。在`-D_FORTIFY_SOURCE`的案例中，Clang 貌似支持了这个功能。我们现在离功能对等又进了一步。

*Last updated: June 29, 2020*