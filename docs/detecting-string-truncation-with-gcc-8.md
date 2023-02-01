# 用 GCC 8 检测字符串截断

> 原文：<https://developers.redhat.com/blog/2018/05/24/detecting-string-truncation-with-gcc-8>

继续努力检测常见的编程错误，刚刚发布的 GCC 8 包含许多新的警告以及对现有检查器的增强，以帮助找到 C 和 C++代码中不明显的错误。本文主要关注那些处理意外字符串截断的方法，并讨论一些避免潜在问题的方法。如果你还没有读过，你可能也想读一读大卫·马尔科姆的文章[*GCC 8*](https://developers.redhat.com/blog/2018/03/15/gcc-8-usability-improvements/)中的可用性改进。

要在 RHEL 上使用 GCC 8，参见[如何在红帽企业 Linux 7 上安装 GCC 8 和 Clang/LLVM 6](https://developers.redhat.com/blog/2019/03/05/yum-install-gcc-8-clang-6/)。GCC 8 是 Red Hat Enterprise Linux 8 Beta 中的默认编译器。

## 为什么字符串截断是一个问题？

众所周知，缓冲区溢出是危险的:超过对象末尾的写入会覆盖相邻存储中的数据，从而导致数据损坏。在最良性的情况下，这种破坏只会导致程序的不正确行为。如果相邻数据是可执行文本段中的一个地址，该损坏可能被利用来获得对受影响进程的控制，这可能导致安全漏洞。(有关缓冲区溢出的更多信息，请参见 [CWE-119](https://cwe.mitre.org/data/definitions/119.html) 。)

但是字符串截断不会覆盖任何数据，为什么会有问题呢？无意中截断一个字符串可以被认为是数据损坏:这是创建了一个字符序列，其中一些尾随字符无意中丢失了。字符串截断有两种常见形式。一种方法产生的 NUL 终止字符串比串联字符串的长度之和短。另一种方法产生不以 NUL 字符结尾的字节序列:也就是说，结果不是字符串。在需要字符串的地方使用这样的结果是未定义的。(参见 [CWE-170](https://cwe.mitre.org/data/definitions/170.html) 了解更多关于不适当的字符串终止导致的弱点。)不同种类的截断由不同的功能引起，它们的检测由不同的警告选项控制，两者都由`[-Wall](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wall)`启用。

## GCC 字符串截断检查器

GCC 有两个检测字符串截断错误的检查器:`[-Wformat-truncation](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wformat_002dtruncation)`(GCC 7 中首次引入)和`[-Wstringop-truncation](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wstringop_002dtruncation)`(GCC 8 中新增)。`-Wformat-truncation`通过`snprintf`系列标准输入/输出函数检测截断，`-Wstringop-truncation`通过`strncat`和`strncpy`函数检测同样的问题。这些警告与`[-Wformat-overflow](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wformat_002doverflow)`和`[-Wstringop-overflow](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wstringop_002doverflow)`密切相关，但又有所不同，它们分别通过相应的无边界标准 I/O 函数和`<string.h>`中声明的字符串修改函数来检测缓冲区溢出。所有这些警告，尽管在概念上很简单，但在很大程度上依赖于高级数据和控制流分析，这些分析是由 GCC 中的许多优化过程执行的，以最大限度地提高效率。

## 用 snprintf 形成截断的字符串

形成比预期更短的字符串是最常见的截断方式。它通常是由调用诸如`snprintf`和`strncat`之类的函数产生的。结果是一个有效的字符串，因为它由 NUL 字符正确终止，但它的长度小于预期，因为它缺少一个或多个尾随字符。例如，如果一个字符串表示一个名称，截断它可能会导致它匹配一个不同的名称，如下例所示:

```
  char dirname[256];
  char filename[256];

  FILE* open_file (void)
  {
    char pathname[256];

    snprintf (pathname, sizeof pathname, "%s/%i/%s", dirname, getpid (), filename);

    return fopen (pathname, "w");
  }
```

如果路径名的五个组成部分的连接不适合 256 个字节，则结果将不会指向预期的文件。接近 256 个字符长的`dirname`意味着 PID 可能会被截断，创建的文件不仅会有错误的名称，还会在错误的目录中。如果文件包含敏感数据，另一个进程(可能是由黑客控制的进程)可能会以非法的方式读取或操纵它。为了帮助检测这个问题，GCC 用类似下面的消息来诊断上面的`snprintf`调用:

```
warning: '%i' directive output may be truncated writing between 1 and 11 bytes into a region of size between 0 and 255 [-Wformat-truncation=]
   snprintf (pathname, sizeof pathname, "%s/%i/%s", dirname, getpid (), filename);
                                            ^~
note: 'snprintf' output between 4 and 524 bytes into a destination of size 256
   snprintf (pathname, sizeof pathname, "%s/%i/%s", dirname, getpid (), filename);
   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

该警告使用字符串长度和整数参数范围来确定何时可以截断。当无法确定字符串参数的长度，并且该参数是一个已知大小的数组时，警告将使用数组的大小作为字符串长度的最坏估计值。在上面的例子中，警告文本指出，如果`dirname`只有 244 个字符长，那么在`INT_MAX`附近追加斜杠和一个非常大的 PID 值将导致 PID 被截断，更不用说最后的斜杠和`filename`了。短语“*输出可能被截断*”表示截断是可能的，但不是不可避免的。如果截断是确定的，将使用短语“*输出截断的*”来代替。最后的注释给出了 GCC 确定的函数将写入目的地的最小和最大字符数。

`-Wformat-truncation`警告在 GCC 8 中并不新鲜，但是由于一些增强，它能够比 GCC 7 检测到更多的问题实例。它检测到如此多的警告，以至于用户往往会对他们代码的警告数量感到惊讶。一些用户抱怨说，他们觉得警告太吵了。最常见的抱怨是，它指出代码中截断的结果要么没有以会导致其行为不当的方式使用(例如在终端上打印一条消息或打印到一个日志文件中，其中一部分字符串被截断并不重要)，要么截断是在以后处理的(例如通过`fopen`函数无法打开一个具有截断名称的文件)。

这种观点忽略了一点，即不完整或截断的消息会使程序输出对用户或(在日志文件的情况下)操作员来说很难理解。在更严重的情况下(比如依靠`fopen`无法打开文件)，他们往往会低估下游的安全风险。

然而，尽管尽了最大努力，`-Wformat-truncation`也不是没有真正的误报(根据设计，警告不应该发出，但却是 GCC 错误或限制的结果)。尽管它们被证明比我们希望的更难避免，但没有一个是由于 GCC 架构或设计中的固有缺陷，而是由于各种优化过程中的限制。对于用户和 GCC 开发人员来说，误报都是令人沮丧的(也是耗时的),在这种情况下，它们有助于突出可能被忽视的代码生成改进机会。GCC 开发人员正在跟踪这些误报以及优化机会，并努力寻找解决方案。

不管给定的`-Wformat-truncation`警告实例是否表明程序中可能存在错误或者是误报，为了获得最佳结果，最好避免截断。由于`snprintf`的目的是防止缓冲区溢出，我们建议开发人员假设每个对`snprintf`的非平凡调用都会导致截断(否则，使用该函数将是不必要的),并适当地处理它。当 GCC 检测到截断不能发生时，它将优化处理，消除任何可能导致的开销。

### 安全使用 snprintf

避免截断的一种方法是在存储输出之前使用`snprintf`来确定目标缓冲区的大小，并动态分配它，使它刚好足够大。这是通过调用该函数两次来实现的:一次使用空目标指针，然后再次使用指向已分配缓冲区的指针。缓冲区可以由`malloc`分配，或者当已知其大小足够小时，作为可变长度数组(使用`[-Wvla-larger-than](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Warning-Options.html#index-Wvla)`检测过大的 vla)，例如:

```
  FILE* open_file (const char *dirname, const char *filename)
  {
    errno = 0;
    int n = snprintf (0, 0, "%s/%i/%s", dirname, getpid (), filename);
    if (n < 0)
      {
        perror ("snprintf failed");
        abort ();
      }

    errno = 0; 
    char *pathname = (char *) malloc (n + 1);
    if (!pathname)
      {
        perror ("malloc failed");
        abort ();
      }

    errno = 0;
    n = snprintf (pathname, n + 1, "%s/%i/%s", dirname, getpid (), filename);
    if (n < 0)
      {
        perror ("snprintf failed");
        abort ();
      }

    FILE *fp = fopen (pathname, "w");
    free (pathname);
    return fp;
  }
```

### 使用 sprintf 和 open_memstream 避免截断

另一个解决方案是使用 POSIX `open_memstream`函数创建一个`FILE`对象，当与其他 I/O 函数一起使用时，该对象管理一个动态分配的缓冲区，该缓冲区根据需要增长以适应所有输出。使用这种方法，当然有必要处理内存不足的问题，例如:

```
  FILE* open_file (const char *dirname, const char *filename)
  {
    char *pathname;
    size_t pathsize;

    FILE *pathfp = open_memstream (&pathname, &pathsize);
    if (!pathfp)
      {
        perror ("open_memstream failed");
        abort ();
      }

    fprintf (pathfp, "%s/%i/%s", dirname, getpid (), filename);
    if (fclose (pathfp))
      {
        // Likely out of memory.
        perror ("fclose failed");
        abort ();
      }

    FILE *fp = fopen (pathname, "w");
    free (pathname);
    return fp;
  }
```

### 使用 asprintf 避免截断

最后，BSD 和 GNU 函数`asprintf`可以安全地用于在一次调用中动态分配缓冲区和格式化输出。与`open_memstream`方法一样，`asnprintf`的代价是调用者必须准备好处理函数分配内存失败的情况(也就是检测和处理`ENOMEM`)。

### 发生截断时的处理

当避免截断不可行时，需要进行处理。为了处理`snprintf`截断，从函数返回的值必须用于采取一些行动。GCC 查看该值是否以有意义的方式使用，并避免在有意义的情况下发出警告。请注意，将返回值赋给一个未使用的变量是不够的。最简单但不一定是最合适的处理截断的方法是中止，例如:

```
  FILE* open_file (const char *dirname, const char *filename)
  {
    char pathname[256];

    int n = snprintf (pathname, sizeof pathname, "%s/%i/%s", dirname, getpid (), filename);

    if (n < 0)
      {
        perror ("snprintf failed");
        abort ();
      }

    if ((size_t)n > sizeof pathname)
      {
        perror ("pathname too long");
        abort ();
      }

    return fopen (pathname, "w");
  }
```

## 用 strcat 形成截断的字符串

调用`strncat`也会导致字符串截断。`strncat`(和`strncpy`)的起源可以追溯到 1979 年发布的[版本 7 UNIX](https://en.wikipedia.org/wiki/Version_7_Unix) ，其中引入了一些函数来操作不一定以 NUL 字符结束的二进制数据数组，比如目录条目或加密密钥。与本文中讨论的其他函数不同，`strncat`即使在最初的预期情况下也不可能安全使用。为了安全使用，该函数不仅需要将目标中剩余空间的大小作为参数，还需要将从非字符串中复制的最大字符数作为参数。但是，通过只提供一个 size 参数，不可能同时避免缓冲区溢出和字符串截断。由于防止缓冲区溢出往往被视为比防止字符串截断更重要，GCC 假设 size 参数指的是目标缓冲区中的剩余空间，并期望安全调用符合以下模式(另请参见 US-CERT 文章中的 [`strncpy()`和`strncat()`](https://www.us-cert.gov/bsi/articles/knowledge/coding-practices/strncpy-and-strncat) ):

```
    strncat (dest, src, dest_size - strlen (dest) - 1);
```

不诊断具有这种形式的呼叫。其他调用，比如那些从源字符串的大小或长度中以某种方式获得大小的调用，由`-Wstringop-overflow`进行诊断。这包括不安全的呼叫，如

```
    strncat (dest, src, strlen (src));   // incorrect - warning
```

和

```
    strncat (dest, src, sizeof src);   // incorrect - warning
```

## 形成非 NUL 终止的序列

一种完全不同的字符串截断形式是调用`strncpy`产生的。不像像`snprintf`和`strncat`这样的函数总是附加一个终止 NUL，当传递给`strncpy`的源字符串比第三个参数指定的长度长时，该函数会截断副本，而不会在末尾附加一个 NUL。结果不是 C 或 C++意义上的字符串(在两种语言中都被定义为以 NUL 终止的字节序列)，因此，它不适合作为需要字符串的函数的参数。调用字符串处理函数(如`strlen`)时，一个常见的错误是参数不是以 NUL 结尾的字符串，例如:

```
  FILE* open_file (const char *dirname, const char *filename)
  {
    char pathname[256];

    strncpy (pathname, dirname, sizeof pathname);
    strncat (pathname, "/", sizeof pathname);
    strncat (pathname, filename, sizeof pathname);

    return fopen (pathname, "w");
  }
```

如果`dirname`长于 255 个字符，那么对`strncpy`的调用会将前 256 个字符复制到`pathname`中，而不会添加终止 NUL。随后对 strncat 的调用将试图在超过`pathname`缓冲区末尾的地方写入路径分隔符和`filename`的内容。具体发生在哪里取决于缓冲区末尾以外的内存内容(第一个 NUL 字节的位置)。GCC 通过诊断如下代码来帮助检测这些错误:

```
warning: 'strncpy' specified bound 256 equals destination size [-Wstringop-truncation]
   strncpy (pathname, dirname, sizeof pathname);
   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
warning: 'strncat' specified bound 256 equals destination size [-Wstringop-overflow=]
   strncat (pathname, filename, sizeof pathname);
   ^~~~~~~
```

第一个警告与我们正在讨论的案例有关。(请注意，此警告可能由于 GCC 错误 [82944](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=82944) 而被抑制。)第二个警告是由于不正确地限制了`strncat`复制的字符数。

### 安全使用 strncpy

一般来说，除了将目的地的大小设置为至少比源字符串的长度大一个字节之外，不可能通过`strncpy`来避免字符串截断。然而，使用这种方法，使用`strncpy`就变得没有必要了，可以避免使用这个函数，而使用其他 API，比如`strcpy`或(不太理想的)`memcpy`。关于`strncpy`的问题已经写了很多，我们建议尽可能避免它。然而，值得记住的是，与其他标准的字符串处理函数不同，`strncpy`总是按照第三个参数指定的数量写入字符；如果源字符串较短，函数用 NULs 填充剩余的字节。

### 减轻 strncpy 截断

由于无法避免被`strncpy`截断，当使用其他函数不可行时，有必要确保`strncpy`的结果以 NUL 终止，并且在`strncpy`返回后，必须显式插入 NUL:

```
    char pathname[256];
    strncpy (pathname, dirname, sizeof pathname);
    pathname[sizeof pathname - 1] = '\0';
```

GCC 会尝试检测这些使用，并在确定 NUL 是在字符串处理函数使用数组之前插入的时候避免发出警告。然而，上面概述的简单方法遇到了与忽略`snprintf`截断相同的问题，因此，为了安全起见，截断应该如上所述被检测和处理。在这种情况下，GCC 8 不会检测到缺失的处理，但未来的版本可能会。

示例中避免后续`strncat`调用中可能的缓冲区溢出留给读者练习。

与本文中讨论的其他警告相比，`-Wstringop-truncation`可能不太容易出现假阴性，但可能比`-Wformat-truncation`更容易出现假阳性。这是因为该函数最初的安全用途并不总是能与不安全用途区分开来。这是一个必要的判断电话，以决定是否在这些情况下发布诊断。GCC 开发人员决定谨慎行事，并发出警告，因为在正确和安全的使用中，假阳性很容易被抑制，尤其是由有经验的程序员抑制，而假阴性会让缺乏经验或不太细心的程序员的错误不被注意到。为了帮助区分两组用例并避免误报，GCC 8 引入了一个新的属性来修饰数组和指针，该属性不需要以 NUL 终止。该属性的名称是`[nonstring](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Common-Variable-Attributes.html#index-nonstring-variable-attribute)`，GCC 使用它来隐藏缺少 NUL 的 select `-Wformat-truncation`实例。值得注意的是，由于非 NUL 终止的字符数组对于需要字符串的函数(如`strlen`或`strcpy`)来说不是有效的参数，因此对使用`nonstring`数组的函数进行诊断是很重要的。

*Last updated: March 8, 2019*