# 用 GCC 11 检测内存管理错误，第 2 部分:解除分配函数

> 原文：<https://developers.redhat.com/blog/2021/05/05/detecting-memory-management-bugs-with-gcc-11-part-2-deallocation-functions>

本文的前半部分描述了在 [C 和 C++](/topics/c) 中的动态内存分配，以及一些新的 [GNU 编译器集合(GCC) 11](https://gcc.gnu.org/) 特性，这些特性可以帮助您检测动态分配中的错误。这第二部分完成了 GCC 11 在这方面的特性，并解释了检测机制可能报告误报或漏报的地方。

在整篇文章中，我为那些想尝试的人提供了到[编译器浏览器](https://godbolt.org)上的代码示例的链接。您会在每个示例的源代码上方找到链接。

**注**:阅读本文前半部分:[用 GCC 11 检测内存管理 bug，第 1 部分:了解动态分配](/blog/2021/04/30/detecting-memory-management-bugs-with-gcc-11-part-1-understanding-dynamic-allocation/)。

## new 和 delete 运算符不匹配

C++的一个特点是对特定形式和重载`operator new()`的调用必须与对应的形式和重载`operator delete()`成对出现。C++提供了许多现成的操作符，GCC 知道所有这些。但除此之外，C++程序还可以定义自己的这些操作符的重载。这样做的人必须确保在匹配对中定义它们。否则，试图释放由错误形式分配的对象很容易导致与其他不匹配相同的潜在错误。此外，声明运算符标量形式的类还应该声明一对相应的数组形式。(参见 [C++核心指南](https://isocpp.github.io/CppCoreGuidelines/)中的 [R.15:总是重载匹配的分配/解除分配对](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#r15-always-overload-matched-allocationdeallocation-pairs)。)

例如，假设我们定义了一个管理自己内存的类。我们定义了`operator new()`和`operator delete()`的成员形式，但忽略了定义运算符的数组形式。接下来，我们分配一个类的对象数组，然后我们尝试释放它，如下面的代码片段所示。GCC 11 发现了这个错误，并发出`-Wmismatched-new-delete`警告指出这一点。因为`operator new()`和`operator delete()`被认为是特殊的，所以即使没有属性`malloc`也会发生这种情况。 [`array_new_delete`](https://godbolt.org/z/s7aoGj) 测试用例展示了一个例子:

```
#include <stddef.h>

struct A
{
  void* operator new (size_t);
  void operator delete (void*);
};

void f (A *p)
{
  delete p;
}

void test_array_new_delete (void)
{
  A *p = new A[2];
  f (p + 1);
}

```

编译器警告是:

```
In function 'void f(A*)',
    inlined from 'void test_array_new_delete()':
**warning**: 'static void A::operator delete(void*)' called on pointer returned from a mismatched allocation function [**-Wmismatched-new-delete**]
11 | delete p;
   | ^
In function 'void test_array_new_delete()':
**note**: returned from 'void* operator new [](long unsigned int)'
16 | A *p = new A[2];
   |        ^

```

注意这个例子是如何用一个不指向分配数组开始的指针调用`f()`的，并且警告仍然能够检测到它不是 deallocation 函数的有效参数。即使没有优化，警告仍然有效，为了让 GCC 找到 bug，必须用`-O2`选项编译这个例子。这是因为对`new`和`delete`表达式的调用在不同的函数中，GCC 必须内联这些函数来检测不匹配。

## 取消分配未分配的对象

另一个动态内存管理错误是试图释放一个没有被动态分配的对象。例如，将局部变量等命名对象的地址传递给解除分配函数是一个错误。通常，解除分配调用会崩溃，但它不必这样做。发生什么往往取决于对象指向的内存内容。GCC 早就实现了一个警告来检测这类错误:`-Wfree-nonheap-object`。但是在 GCC 11 之前，该选项只检测到了与`free()`函数相关的最明显的错误——基本上只是将一个命名变量的地址传递给它。GCC 11 得到了增强，可以检查对每个已知的 C 或 C++释放函数的每个调用。除了`free()`，这些函数还包括`realloc()`，以及在 C++中`operator delete()`的所有非放置形式。此外，检查对用属性`malloc`标记的用户定义的释放函数的调用。例如，参见 [`free_declared`](https://godbolt.org/z/afohGK) :

```
#include <stdlib.h>

void use_it (void*);

void test_free_declared (void)
{
  char buf[32], *p = buf;
  use_it (p);
  free (p);
}

```

编译器警告是:

```
In function 'test_free_declared':
**warning**: 'free' called on unallocated object 'buf' [**-Wfree-nonheap-object**]
9 | free (p);
  | ^~~~~~~~
**note**: declared here
7 | char buf[32], *p = buf;
  |      ^~~

```

## 属性 alloc_size

除了属性`malloc`的两种形式之外，还有一种属性可以帮助 GCC 发现内存管理错误。属性`alloc_size`告诉 GCC 哪个分配函数的参数指定了分配对象的大小。例如，`malloc()`和`calloc()`函数被隐式声明如下:

```
__attribute__ ((malloc, malloc (free, 1), alloc_size (1)))
void* malloc (size_t);

__attribute__ ((malloc, malloc (free, 1), alloc_size (1, 2)))
void* calloc (size_t, size_t);

```

利用属性`alloc_size`有助于 GCC 找到对已分配内存的越界访问。 [`my_alloc_free`](https://godbolt.org/z/eafcfK) 示例展示了如何使用用户定义分配器的属性:

```
void my_free (void*);

__attribute__ ((malloc, malloc (my_free, 1), alloc_size (1)))
void* my_alloc (int);

void* f (void)
{
  int *p = (int*)my_alloc (8);
  memset (p, 0, 8 * sizeof *p);
  return p;
}

```

编译器警告是:

```
In function 'test_memset_overflow':
warning: 'memset forming offset [8, 31] is out of the bounds [0, 8] [**-Warray-bounds**]
9 | memset (p, 0, 8 * sizeof *p);
  | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

## 限制

作为新产品，GCC 11 对动态内存管理缺陷的检测并不完美。这些警告容易受到误报和漏报的影响。

### 假阳性

误报通常是由 GCC 未能确定某些代码路径不可达造成的。这往往会对代码中的`-Wfree-nonheap-object`产生特别的影响，这些代码根据某些条件使用声明的数组或动态分配的缓冲区，然后根据其他一些等价的条件释放缓冲区。当 GCC 不能证明这两个条件相等时，它可能会发出警告。GCC bug [54202](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=54202) 展示了这可能是如何发生的。值得注意的是，该 bug 是在 2012 年针对 GCC 4.7 提交的。所以这个警告已经很久了，但是最初的实现只检测到最基本的错误，所以误报很少。但是因为 GCC 11 增强了警告的实现，以检查对每个已知解除分配函数的每个调用，所以这种误报会更频繁地出现，与发现的错误数量成比例。来自 bug 54202 的测试用例可以在编译器浏览器上看到[:](https://godbolt.org/z/sPjhKr)

```
typedef struct Data
{
  int refcount;
} Data;

extern const Data shared_null;

Data *allocate()
{
  return (Data *)(&shared_null);
}

void dispose (Data *d)
{
  if (d->refcount == 0)
    free (d);
}

void f (void)
{
  Data *d = allocate();
  dispose (d);
}

```

编译器警告是:

```
In function 'dispose',
    inlined from 'f':
**warning**: attempt to free a non-heap object 'shared_null' [**-Wfree-nonheap-object**]
18 | free(d)
   | ^~~~~~~

```

我们确实希望这些情况不会太常见，但计划在未来的更新中进一步减少它们。在此之前，当它们发生时，我们建议使用`[#pragma GCC diagnostic](https://gcc.gnu.org/onlinedocs/gcc/Diagnostic-Pragmas.html)`禁用警告。

### 假阴性

类似地，但在更大程度上，有条件地尝试释放未分配对象的错误代码可能根本不会被诊断出来。由于 GCC 中分析的局限性，这些假阴性通常是很常见的，也是不可避免的。一个主要原因是分析的准确性和深度通常取决于优化，特别是内联。除了只能通过优化来实现之外，内联还受到一些约束，这些约束旨在实现速度和空间效率之间的最佳平衡。对特定函数的调用是否内联到其调用者取决于它对调用者的好处。在内部，收益性是由 GCC 伪指令中函数的大小决定的。该约束由`[-finline-limit=](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-finline-limit)`选项控制。将该选项设置为一个适当高的值，可以内联翻译单元中定义的大多数函数，并将它们的主体暴露给分析。(对于即将发布的代码，不建议更改限制。)此外，由`[-flto](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-flto)`选项启用的链接时间优化(LTO)将相同的分析应用于跨翻译单元边界内联的函数。

也就是说，我们知道有一类假阴性不受内联试探法的影响，并且有可能做得更好。我们希望在未来的版本中解决一些问题。下面的代码是一个可以解决的限制的例子，虽然不容易。因为对`f()`的调用可能会覆盖`g()`存储在`*p`中的值，所以在这些情况下发出警告会是误报。如果`f()`不修改传递给它的对象，那么声明函数来接受一个`const void*`参数似乎是一个解决方案。但是因为抛弃 constness 不是一个错误，GCC 必须保守地假设这个函数实际上可能会这样做。为了发出正确的代码，这种假设是必要的，但是警告可以合理地做出更强的假设，尽管代价是对于严格正确(尽管明显有问题)的程序会有一些误报。实现这些更严格的假设是未来版本考虑的增强之一。

```
void f (void*);

void g (int n)
{
  char a[8];
  char *p = 8 < n ? malloc (n) : a;
  *p = n;
  f (p + 1);    // might change *p
  if (*p < 8)
    free (p);   // missing warning
}
```

## 第二部分的结论

GCC 11 可以发现许多现成的内存管理错误，而无需对程序源代码进行任何更改。但是为了在定义自己的内存管理例程的代码中充分利用这些特性，您可以通过用属性`malloc`和`alloc_size`注释这些函数来获益。GCC 检查所有对内存分配和释放函数的调用，即使没有优化，但在没有优化的情况下，检测仅限于函数体的范围。通过优化，分析还包括内联到调用方的函数。提高内联限制可以改善分析，LTO 也是如此。GCC 静态分析器执行相同的检查，但是考虑相同翻译单元中的所有函数，而不考虑内联。

*Last updated: October 14, 2022*