# 扩大编译器对 _FORTIFY_SOURCE 中缓冲区溢出的检查

> 原文：<https://developers.redhat.com/blog/2021/04/16/broadening-compiler-checks-for-buffer-overflows-in-_fortify_source>

到目前为止，缓冲区溢出是 [C 或 C++](/topics/c/) 程序中最常见的漏洞，多年来已经出现了许多技术来早期检测溢出并中止执行。由 GNU C 库提供的`_FORTIFY_SOURCE`宏有助于减少这些溢出，并在 [Red Hat Enterprise Linux](/products/rhel/overview) 中广泛部署。[红帽安全博客](https://access.redhat.com/blogs/766093/posts/1976213)上的这篇文章很好地介绍了`_FORTIFY_SOURCE`。

在 GNU C 库的 2.33 版本中，我们为`_FORTIFY_SOURCE`增加了一个新的级别，以提高宏提供的保护。在这里，我们仔细看看 GCC 中`_FORTIFY_SOURCE`的内部结构，并探索对这个新级别的需求。

## _ FORTIFY _ 引擎盖下的来源

`_FORTIFY_SOURCE`宏用强化的包装器替换了一些常用函数，这些包装器检查这些函数中的目标缓冲区的大小是否足够处理输入数据。如果检查失败，程序将中止，用户将避免可能被利用的缓冲区溢出。强化包装器函数的实际实现有很多 glibc-ism，但它在概念上是一个`wmemcpy`的包装器，如下所示:

```
extern inline wchar_t *
__attribute__ ((always_inline))
wmemcpy (wchar_t *__restrict __s1, const wchar_t *__restrict __s2, size_t __n)
{
  if (__builtin_object_size (__s1, 0) != (size_t) -1)
    return __wmemcpy_chk (__s1, __s2, __n,
                          __builtin_object_size (__s1, 0) / sizeof (wchar_t));
  return __original_wmemcpy (__s1, __s2, __n);
}

```

GNU C 库为许多函数实现了类似的强化包装器。一些包装器看起来略有不同；例如，`memcpy`包装器无条件地调用一个名为`__builtin___memcpy_chk`的内置。这些内置函数还粗略地评估了刚才显示的包装函数。

### 基石:`__builtin_object_size`

如示例源代码所示，`_FORTIFY_SOURCE`的核心功能是基于`__builtin_object_size`内置的。这个内置函数评估第一个参数(它是一个指针)指向的对象，并返回对象大小的估计值。根据内置的第二个参数，大小可能是最小估计值或最大估计值。如果内置程序无法推断出对象的大小，那么它或者返回`(size_t)0`或者`(size_t)-1`，这取决于请求的是最小还是最大估计值。

编译器将包装器翻译成对`__wmmcpy_chk`的调用或者对原始`wmemcpy`的调用，在上面的例子中用`__original_wmemcpy`表示。最重要的是，编译器优化掉了`__builtin_object_size`比较，因为它是一个常量表达式:`__builtin_object_size`总是返回一个常量。如果`__builtin_object_size`不能推导出一个恒定的对象大小，编译器就退回到默认实现`wmemcpy`。

### 三思而后行

`wmemcpy`的校验变量——即`__wmemcpy_chk`——一般实现如下:

```
wchar_t *
__wmemcpy_chk (wchar_t *s1, const wchar_t *s2, size_t n, size_t ns1)
{
  if (__glibc_unlikely (ns1 < n))
    __chk_fail ();
  return (wchar_t *) memcpy ((char *) s1, (char *) s2, n * sizeof (wchar_t));
}

```

其中`__chk_fail`中止程序。乍一看，这看起来很慢，因为它为每个调用添加了一个条件。然而，`glibc`以一种运行时开销是恒定的，并且实际上可以忽略不计的方式来布置对性能敏感的函数。

## 从长远来看，一切都是动态的

对该功能的明显扩展是允许`__builtin_object_size`返回非常数结果。这个想法最终以名为`__builtin_dynamic_object_size`的新内置的形式出现，在 LLVM 9 中实现。尽管有一些不同，这个内置的是`__builtin_object_size`的替代者。像`__builtin_object_size`一样，新的内置试图将对象的大小评估为一个常数。如果对象大小不恒定，内置将尝试推导出一个计算对象大小的表达式。如果连这都不可能，它就退出并返回`(size_t)-1`或`(size_t)0`。总的来说，这允许在更多的情况下，可以很好地知道对象的大小，以便在运行时用它做一些有用的事情。

### 第三级设防

在 glibc 2.33 中，这种支持以一种新的强化级别:`_FORTIFY_SOURCE=3`的形式具体化为一种实际的强化特性。这一强化级别扩大了`_FORTIFY_SOURCE`可以捕捉的病例的覆盖范围。例如，下面的函数:

```
size_t add_size;
size_t multiplier;

void *
do_something (void *in, size_t insz, size_t sz)
{
  void *buf = malloc ((sz + add_size) * multiplier);

  memcpy (buf, in, insz);

  return buf;
}

```

当用`_FORTIFY_SOURCE=2`构建时，不会导致调用`memcpy`的强化版本(即`__memcpy_chk`)。结果，`insz`可能比`buf`的尺寸大的情况就溜走了。

有了`_FORTIFY_SOURCE=3`，虽然`__builtin_dynamic_object_size`可以评价到`(sz + add_size) * multiplier`。有了这个表达式，编译器可以生成对`__memcpy_chk`的调用，并允许强化。这有望显著扩大强化覆盖范围，以包括编译器可以看到对象大小的非常数表达式的情况。这是一个很大的改进，但是我们决定不在`_FORTIFY_SOURCE`级别`2`实现它。

### `_FORTIFY_SOURCE=3`中的运行时间开销

早期的`_FORTIFY_SOURCE`级别依赖于恒定的对象大小；因此，运行时开销可以忽略不计。`_FORTIFY_SOURCE=3,`然而，变化在于，因为用于计算物体尺寸的表达式可以是任意复杂的。复杂表达式会增加任意多的运行时开销。进一步，考虑前面例子中的`do_something`被循环调用的可能性；开销被放大了。这是一个足够好的理由不偷偷在引擎盖下这个新功能。新的级别允许开发人员修改它，并决定开销对于他们的用例是否可接受。

## _FORTIFY_SOURCE 的下一步是什么

GCC 对`__builtin_dynamic_object_size`或等效功能的支持正在进行中。目前，这仅在使用 LLVM 构建应用程序时可用。使用`__builtin_dynamic_object_size`可能会导致一些不可避免的性能开销。我们希望通过 GCC 实现解决这些问题，并将其反馈到 LLVM 中，从而使两种实现保持一致和高性能。

`_FORTIFY_SOURCE=3`随着我们接受检查的可变性能开销，开始转变放射源强化的设计。随着编译器在推断对象大小方面变得更加智能，这也增加了未来额外覆盖的可能性。希望这只是开始！

*Last updated: October 14, 2022*