# 使用源代码级注释来帮助 GCC 检测缓冲区溢出

> 原文：<https://developers.redhat.com/articles/2021/06/25/use-source-level-annotations-help-gcc-detect-buffer-overflows>

缓冲区溢出漏洞等越界内存访问仍然是 2021 年最危险的软件漏洞之一(参见 [*2020 年 CWE 25 大最危险软件漏洞*](https://cwe.mitre.org/top25/archive/2020/2020_cwe_top25.html) )。事实上，越界写入( [CWE-787](https://cwe.mitre.org/data/definitions/787.html) )从 2019 年的第十二位跃升至 2020 年的第二位，而越界读取( [CWE-125](https://cwe.mitre.org/data/definitions/125.html) )从第五位升至第四位。

认识到在开发周期早期检测编码错误的重要性，最近的 GNU 编译器集合(GCC)版本通过使用诸如 [`-Warray-bounds`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Warray-bounds) 、 [`-Wformat-overflow`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wformat-overflow) 、 [`-Wstringop-overflow`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wstringop-overflow) 和(最近在 GCC 11 中) [`-Wstringop-overread`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wstringop-overread) 等警告，显著提高了编译器诊断这些危险错误的能力。然而，所有这些警告共有的一个共同限制是，它们一次只能分析单个函数中的代码。除了对一小部分内置函数的调用，比如编译器内置的`memcpy()`，警告会在函数调用边界停止。这意味着，当一个函数中分配的缓冲区溢出到从该函数调用的函数中时，除非被调用的函数被内联到调用者中，否则不会检测到问题。

本文描述了三种简单的源代码级注释，程序可以使用它们来帮助 GCC 检测跨越函数调用边界的越界访问，即使这些函数是在不同的源文件中定义的:

*   属性`access`(在 GCC 10 中首次引入，在 [C 和 C++](/topics/c/) 中都有)
*   可变长度数组(VLA)函数参数([GCC 11](https://gcc.gnu.org/gcc-11/changes.html)中新增，仅在 C 中可用)
*   数组函数参数(GCC 11 中的新功能，仅在 C 语言中可用)

## 属性访问

对于将指向缓冲区的指针作为一个参数，将其大小作为另一个参数的函数来说,`[access](https://gcc.gnu.org/onlinedocs/gcc/Common-Function-Attributes.html)`函数属性非常有用。一个例子可能是 POSIX [`read()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/read.html) 和 [`write()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/write.html) 对函数。除了让程序员关联这两个参数之外，该属性还指定了函数如何访问内容缓冲区。该属性适用于函数声明，并在调用点和分析函数定义以检测无效访问时使用。

该属性具有以下语法:

*   `access` ( `*access-mode*`，`*ref-index*`)
*   `access` ( `*access-mode*`、`*ref-index*`、`*size-index*`)

`*ref-index*`和`*size-index*`表示位置参数，并分别给出缓冲区的从 1 开始的参数号及其大小。缓冲区参数`*ref-index*`可以声明为普通对象指针，包括`void*`，也可以使用数组形式(如`T[]`或`T[N]`)。它不需要指向完整的类型。可选的`*size-index*`必须引用一个整数参数，指定函数可能访问的数组元素的数量。对于不完整类型的缓冲区，如`void*`，size 参数用于给出字节数。当未指定`*size-index*`时，假设缓冲器有一个元素。

`*access-mode*`描述函数如何访问缓冲区。在 GCC 11 中，识别了四种模式:

*   `read_only`模式表示该功能从提供的缓冲区读取数据，但不写入其中。缓冲区应该由调用者初始化。在缓冲区上，`read_only`模式意味着比`const`限定符更强的保证，因为限定符可以被丢弃，缓冲区可以在一个定义良好的程序中被修改，只要缓冲区对象本身不是`const`。应用`read_only`模式的参数可以(但不一定)是`const`限定的。声明一个参数`read_only`与在 C99 中声明一个参数同时声明`const`和`restrict`具有相同的含义(尽管 GCC 11 不认为这两者是等价的)。
*   `write_only`模式表示函数将数据写入所提供的缓冲区，但不从中读取数据。缓冲区不需要初始化。试图将`write_only`模式应用于`const`限定的参数会导致警告，该属性会被忽略。这实际上是没有相关属性`access`的参数的默认模式。
*   `read_write`模式表示该功能既读取数据又将数据写入缓冲区。缓冲区应该被初始化。试图将`read_write`模式应用于`const`限定的参数会导致警告，该属性会被忽略。
*   `none`模式意味着该函数根本不访问缓冲区。缓冲区不需要初始化。这种模式在 GCC 11 中是新的，为执行参数验证而不访问缓冲区中的数据的函数提供。

以下示例显示了如何使用属性来注释 POSIX `read()`和`write()`函数:

```
__attribute__ ((access (write_only, 2, 3))) ssize_t
read (int fd, void *buf, size_t nbytes); 
__attribute__ ((access (read_only, 2, 3))) ssize_t
write (int fd, const void *buf, size_t nbytes);
```

因为`read()`函数将数据存储在提供的缓冲区中，所以属性访问模式是`write_only`。类似地，因为`write()`从缓冲区读取数据，所以访问模式是`read_only`。

`access`属性的作用类似于使用可变长度数组符号声明函数参数，只是它更加灵活。除了访问模式之外，`*size-index*`参数还可以将一个指针与函数参数列表中它后面的一个大小相关联，这是常见的情况。我们将在下一节讨论 VLA 符号。

## VLA 函数参数

在 C 语言中(但在 GCC 11 中，而不是在 C++中)，使用数组符号声明的函数参数可以引用非常数表达式，包括同一函数的前几个参数，作为它的界限。当边界引用另一个函数参数时，该参数的声明必须在 VLA 的声明之前(GCC 提供了一个扩展来绕过语言限制；参见 GCC 手册中的[可变长度数组](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Variable-Length.html)。当只有最重要的界限使用这样的界限时，它就像任何其他数组一样衰减为普通指针。否则，它就是 VLA。由于这两种数组在上下文中的区别相当微妙，GCC 诊断将两者都称为 vla。在本文的其余部分，我们也将遵循这一简化约定。例如:

```
void init_array (int n, int a[n]);
```

该函数将一个普通数组(或者更准确地说，一个指针)作为其第二个参数，其元素数量由第一个参数给出。尽管这不是语言所必需的，但是向函数传递一个元素比第一个参数少的数组几乎肯定是一个错误。GCC 检查对此类函数的调用，并在确定数组小于预期值时发出警告。例如， [`vla_init`](https://godbolt.org/z/Tha38rd6f) 程序让 GCC 发出以下警告:

```
#define N 32

int* f (void)
{
  int *a = (int *)malloc (N);
  init_array (N, a);
  return a;
}
```

```
In function 'f':
warning: 'init_array' accessing 128 bytes in a region of size 32 [-Wstringop-overflow=]
   10 |     init_array (N, a);
      |     ^~~~~~~~~~~~~~~~~
note: referencing argument 2 of type 'int *'
note: in a call to function 'init_array'
    5 | void init_array (int n, int a[n]);
      |      ^~~~~~~~~~
```

该警告检测到(可能的)错误，即将一个小于第一个参数的数组传递给`init_array`。

正如已经提到的，声明一个只有最重要的边界是变量的数组实际上并不声明一个 VLA，而是一个普通的数组。只有不太重要的界限才重要。这意味着下面的[示例](https://godbolt.org/z/xvaeYhnsW)中的声明都是有效且等价的:

```
void init_vla (int n, int[n]);
void init_vla (int, int[32]);
void init_vla (int, int*);
void init_vla (int n, int[n + 1]);
```

然而，这带来了一个问题:哪个声明应该用于越界访问警告？GCC 11 中实现的解决方案是信任第一个声明，并发出一个单独的警告， [`-Wvla-parameter`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wvla-parameter) ，用于建议数组中不同数量元素的任何后续重新声明。上述示例中的四个声明会导致以下警告:

```
warning: argument 2 of type 'int[32]' declared as an ordinary array [-Wvla-parameter]
    2 | void init_vla (int, int[32]);
      |                     ^~~~~~~
warning: argument 2 of type 'int *' declared as a pointer [-Wvla-parameter]
    3 | void init_vla (int, int*);
      |                     ^~~~
warning: argument 2 of type 'int[n + 1]' declared with mismatched bound [-Wvla-parameter]
    4 | void init_vla (int, int[n + 1]);
      |                     ^~~~~~~~~
note: previously declared as a variable length array 'int[n]'
    1 | void init_vla (int n, int[n]);
      |                       ^~~~~~
```

## 数组函数参数

由于对无限堆栈分配的关注，vla 在现代 C 代码中往往使用不足，即使在像函数声明这样的上下文中，vla 不仅安全，而且有助于提高分析代码的能力。在没有 vla 的情况下，一些项目使用更简单的约定来声明函数参数，期望调用者使用普通的数组符号`T[N]`来提供对某个恒定的最小数量的元素(比如 N)的访问。例如，C 标准函数 [`tmpnam()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/tmpnam.html) 期望其参数指向至少有`L_tmpnam`个元素的数组。为了明确这一点，GNU libc 2.34 将其声明为:

```
char *tmpnam (char[L_tmpnam]);
```

GCC 11 识别这个约定，当它确定对函数的调用提供了一个更小的数组时，它发出一个警告。例如，在 [Linux](/topics/linux) 上，其中`L_tmpnam`被定义为 20，对于下面显示的函数，GCC 发出以下警告:

```
void g (void)
{
  char a[16];
  if (tmpnam (a))
    puts (a);
}
```

```
In function 'g':
warning: 'tmpnam' accessing 20 bytes in a region of size 16 [-Wstringop-overflow=]
 10 | if (tmpnam (a))
    |     ^~~~~~~~~~
note: referencing argument 1 of type 'char *'
note: in a call to function 'tmpnam'
  3 | extern char* tmpnam (char[L_tmpnam]);
    |              ^~~~~~
```

除了函数调用，GCC 11 还检查用数组参数声明的函数的定义，并对给定常数边界的越界访问发出警告。例如，`init_array()`函数的这个定义触发了一个`-Warray-bounds`警告，如编译器资源管理器[示例](https://godbolt.org/z/Ybnc8T7q1)所示:

```
void init_array (int, int a[32])
{ 
  a[32] = 0;
}
```

```
In function 'init_array':
warning: array subscript 32 is outside array bounds of 'int[32]' [-Warray-bounds]
    3 |   a[32] = 0;
      |   ~^~~~
note: while referencing 'a'
    1 | void init_array (int, int a[32])
      |                       ~~~~^~~~~
```

类似于涉及 VLA 参数的函数重声明，GCC 也检查那些涉及参数数组形式的重声明，并发出不匹配的 [`-Warray-parameter`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Warray-parameter) 警告，如下面的[示例](https://godbolt.org/z/o97Pxrrxb)所示:

```
void init_array (int, int[32]);
void init_array (int, int[16]);
void init_array (int n, int[]);
void init_array (int n, int*);
```

```
warning: argument 2 of type 'int[16]' with mismatched bound [-Warray-parameter=]
    2 | void init_array (int, int[16]);
      |                       ^~~~~~~
warning: argument 2 of type 'int[]' with mismatched bound [-Warray-parameter=]
    3 | void init_array (int n, int[]);
      |                         ^~~~~
warning: argument 2 of type 'int *' declared as a pointer [-Warray-parameter=]
    4 | void init_array (int n, int*);
      |                         ^~~~
note: previously declared as an array 'int[32]'
    1 | void init_array (int, int[32]);
      |                       ^~~~~~~
```

## 警告和限制

这里讨论的特性在一个有趣的方面是独一无二的:它们涉及简单的词法分析和更复杂的流敏感分析。理论上，词法警告可以既健全又完整(也就是说，它们既不会出现误报，也不会出现漏报)。因为它们是在词法分析过程中处理的，所以`-Warray-parameter`和`-Wvla-parameters`警告实际上没有这样的问题。另一方面，基于流的警告本质上既不健全也不完整；相反，它们不可避免地容易出现假阳性和假阴性。

### 假阴性

要使用`access`属性并检测越界访问，它们所应用的函数不能是内联的。一旦一个函数被内联到它的调用者中，它的大多数属性通常都会丢失。如果不能从内联函数体容易地确定越界访问，这可以防止 GCC 检测错误。例如，下面代码清单中的`genfname()`函数使用`getpid()`在`/tmp`目录中生成一个临时文件名。因为在大多数系统上，POSIX [`gepid()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/getpid.html) 函数返回一个 32 位的 int，所以该函数可以生成的最长名称是 26 个字符(10 个用于`INT_MAX`，16 个用于`/tmp/tmpfile.txt`字符串，1 个字节用于终止 nul 字符)。当`main()`中的`genfname(a)`调用没有内联时，GCC 会发出如下警告。但是当调用被内联时，警告就消失了。在这里你可以看到两个场景并排[。](https://godbolt.org/z/Pv6qbzd1o)

```
#include <stdio.h>
#include <unistd.h>

inline void genfname (char name[27])
{
  snprintf (name, 27, "/tmp/tmpfile%u.txt", getpid ());
}

int main (void)
{
  char name[16];
  genfname (name);
  puts (name);
} 
```

```
In function 'main':
warning: 'f' accessing 27 bytes in a region of size 16 [-Wstringop-overflow=]
   11 |   f (a);
      |   ^~~~~
note: referencing argument 1 of type 'char *'
note: in a call to function 'f'
    3 | inline void f (char a[27])
      |             ^
```

顺便说一句，如果你想知道为什么`-Wformat-truncation`没有诊断出`sprintf()`呼叫，那是因为警告不能确定任何关于`getpid()`结果的事情。

### 假阳性

一般来说，基于这里讨论的注释对越界访问的检测与 GCC 中所有流敏感的警告一样，有着相同的限制和缺点。关于这些的详细讨论，请参见 [*了解 GCC 警告，第 2 部分*](/blog/2019/03/13/understanding-gcc-warnings-part-2) 。一些常见的关于函数注释机制的问题可能值得一提。

如前所述，一些项目使用带有常量绑定的数组参数表示法来提供一个直观的线索，即调用者应该提供一个至少包含同样多元素的数组。但有时约定是模糊的，这意味着函数只在另一个参数有这个或那个值时才使用数组。因为没有什么可以向 GCC 传达这种约定的“怪癖”,所以即使使用是安全的，最终也可能会发出警告。我们建议在这些情况下避免使用公约。

## 未来的工作

在 GCC 11 中，您可以使用`access`属性来检测以下内容:

*   越界访问:`-Warray-bounds`、`-Wformat-overflow`、`-Wstringop-overflow`和`-Wstringop-overread`
*   重叠访问: [`-Wrestrict`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wrestrict)
*   未初始化的访问: [`-Wuninitialized`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wuninitialized)

在未来，我们希望使用该属性来检测只被写入而不被读取的变量( [`-Wunused-but-set-parameter`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wunused-but-set-parameter) 和 [`-Wunused-but-set-variable`](https://gcc.gnu.org/onlinedocs/gcc-11.1.0/gcc/Warning-Options.html#index-Wunused-but-set-variable) )。

我们也在考虑以某种形式将`access`属性扩展到函数返回值以及变量。注释函数返回值将让 GCC 通过从 [`getenv()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/getenv.html) 或 [`localeconv()`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/localeconv.html) 等函数返回的指针来检测修改不可变对象的企图。类似地，注释全局变量将使得检测意外修改对象的内容成为可能，例如环境指针数组 [`environ`](https://pubs.opengroup.org/onlinepubs/9699919799/functions/environ.html) 。

*Last updated: August 26, 2022*