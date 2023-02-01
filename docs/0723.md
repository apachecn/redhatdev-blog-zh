# C 语言中有效的字符串复制和连接

> 原文：<https://developers.redhat.com/blog/2019/08/12/efficient-string-copying-and-concatenation-in-c>

在标准 C <string.h>头文件中声明的最常用的字符串处理函数是那些复制和连接字符串的函数。两组函数都将字符从一个对象复制到另一个对象，并且都返回它们的第一个参数:指向目标对象开头的指针。返回值的选择是低效率的一个来源，这是本文的主题。</string.h>

本文中显示的代码示例仅用于说明。它们不应该被视为推荐的实践，并且可能包含细微的错误。

## 标准溶液

返回函数的第一个参数的设计有时会被想知道其目的的用户质疑——例如参见 [strcpy()返回值](https://stackoverflow.com/questions/3561427)或[C:strcpy 为什么返回它的参数？](https://stackoverflow.com/questions/16386238)简单的答案是，这是由于一个历史事故。第一个函数子集是在 1979 年 UNIX 的[第七版](https://minnie.tuhs.org/cgi-bin/utree.pl?file=V7)中引入的，由 strcat、strncat、strcpy 和 strncpy 组成。尽管在 UNIX 的实现中使用了所有这四个函数，其中一些被广泛使用，但是它们的调用都没有利用它们的返回值。这些函数可以很容易地被定义为返回一个指向最后一个被复制的字符的指针，或者返回一个刚刚经过的字符。

连接两个或更多字符串的最佳复杂度与字符数成线性关系。但是，如上所述，让函数返回目标指针会导致操作效率大大低于最佳效率。这些函数遍历源序列和目标序列，并获得指向这两个序列末尾的指针。指针指向函数(strncpy 除外)附加到目的地的终止 NUL ('\0 ')字符或刚过该字符。然而，通过返回一个指向第一个字符而不是最后一个字符(或者刚刚过去的一个)的指针，NUL 字符的位置丢失了，需要时必须重新计算。这种低效率可以通过将两个字符串 s1 和 s2 连接到目标缓冲区 d 的示例来说明。附加两个字符串的惯用方法(尽管远非理想)是调用 strcpy 和 strcat 函数，如下所示。

```
	strcat (strcpy (d, s1), s2);
```

为了执行串联，除了同时发生的对应的对 d 的传递之外，对 s1 的一次传递和对 s2 的一次传递是所有必要的，但是上面的调用对 s1 进行了两次传递。让我们把电话分成两个陈述。

```
        char *d1 = strcpy (d, s1); // pass 1 over s1
        strcat (d1, s2); // pass 2 over the copy of s1 in d
```

因为 strcpy 返回其第一个参数 d 的值，所以 d1 的值与 d 相同，为了简单起见，下面的示例使用 d，而不是将返回值存储在 d1 中并使用它。在 strcat 调用中，确定最后一个字符的位置包括遍历刚刚复制到 d1 的字符。这样做的成本与第一个字符串 s1 的长度成线性关系。成本随着每个附加的字符串而增加，因此倾向于串联数量乘以所有串联字符串的长度的平方。这种低效率是如此的臭名昭著，以至于为自己赢得了一个名字:画家的算法。(参见 [1](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2349.htm#sad-string) 。)

需要指出的是，除了效率低之外，strcat 和 strcpy 还因其缓冲区溢出的倾向而臭名昭著，因为它们都没有提供对复制字符数量的限制。

## 克服局限性的尝试

当字符串的长度未知并且目标大小固定时，遵循一些流行的安全编码准则来将连接的结果限制为目标大小实际上会导致两次冗余传递。例如，按照 CERT 关于安全使用 [strncpy()和 strncat()](https://www.us-cert.gov/bsi/articles/knowledge/coding-practices/strncpy-and-strncat) 的建议，目的地的大小是 dsize 字节，我们可能以下面的代码结束。

```
	strncpy (d, s1, dsize - 1);      // pass 1 over s1 plus over d up to dsize - 1
	d[dsize - 1] = '\0';             // remember to nul-terminate
	size_t n = strlen (d);           // pass 2 over copy of s1 in d
	strncat (d, s2, dsize - n - 1);  // pass 3 over copy of s1 in d
```

请注意，与对 strncat 的调用不同，上面对 strncpy 的调用不会在 s1 大于 d 的大小时将终止 NUL 字符附加到 d 上。假设它是，这是一个常见的错误。此外，当 s1 比 dsize - 1 短时，strncpy 函数会将所有剩余的字符设置为 NUL，这也被认为是浪费，因为对 strncat 的后续调用最终会覆盖它们。

为了避免冗余，程序员有时会选择首先计算字符串长度，然后使用 memcpy，如下所示，但这是徒劳的。这种方法虽然效率仍然不够理想，但更容易出错，并且难以阅读和维护。

```
	size_t s1len = strlen (s1);      // pass 1 over s1
	if (dsize <= s1len)
          s1len = dsize - 1;            // no need to nul-terminate
	memcpy (d, s1, s1len);           // pass 2 over s1
	size_t s2len = strlen (s2);      // pass 1 over s2
	if (dsize - s1len <= s2len)
          s2len = dsize - s1len - 1;
	memcpy (d + s1len, s2, s2len);   // pass 2, over s2
	d[s1len + s1len] = '\0';         // nul-terminate result
```

### 使用 sprintf 和 snprintf 进行连接

关心代码复杂性和可读性的程序员有时会使用 snprintf 函数。

```
	snprintf (d, dsize, "%s%s", s1, s2);
```

这使得代码非常易读，但是由于 snprintf 的开销相当大，即使效率很低，也比使用字符串函数慢几个数量级。开销不仅是由于解析格式字符串，还由于格式化 I/O 函数实现中固有的复杂性。

GCC 和 Clang 等编译器试图通过将非常简单的 sprintf 和 snprintf 调用转换为 strcpy 或 memcpy 调用来提高效率，从而避免一些 I/O 函数调用的开销。(在线看一个现场[例子](https://godbolt.org/z/RaWkyd)。)但是对于 snprintf 很少执行相应的转换，因为 C 库中没有等价的字符串函数(只有在可以证明 snprintf 调用不会导致输出截断的情况下才进行转换)。单独使用 memcpy 是不合适的，因为它复制的字节数正好与指定的一样多，strncpy 也不合适，因为它甚至会在最后一个 NUL 字符结束后覆盖目标。

由于字符串上的冗余传递，将 snprintf 调用转换为 strlen 和 memcpy 调用序列的开销被认为没有足够的收益。标题为[更好的内置字符串函数](https://www.gnu.org/software/gcc/projects/optimize.html#better_builtin_string_functions)的部分列出了 GCC 优化器在这方面的一些限制，以及改进它所涉及的一些权衡。

### POSIX stpcpy 和 stpncpy

多年来，出现了许多 C 标准之外的库解决方案来帮助解决这个问题。POSIX 标准包括 stpcpy 和 stpncpy 函数，如果找到 NUL 字符，则返回指向它的指针。这些功能可用于减轻上述不便和低效。

```
	const char* **stpcpy** (char* restrict, const char* restrict);
	const char* **stpncpy** (char* restrict, const char* restrict, size_t);
```

特别是，在不考虑缓冲区溢出的情况下，可以这样调用 stpcpy 来连接字符串:

```
	stpcpy (stpcpy (d, s1), s2);
```

但是，当副本必须由目标的大小限制时，等效地使用 stpncpy 并不能消除在第一个 NUL 字符之后以及限制所指定的最大字符数之前将目标的其余部分清零的开销。

```
	char *ret = stpncpy (d, dsize, s1);   // zeroes out d beyond the end of s1
	dsize -= (ret - d);
	stpncpy (d, dsize, s2);               // again zeroes out d beyond the end
```

结果，这个函数仍然是低效的，因为对它的每一次调用都将目标中剩余的空间清零，并且超过了复制的字符串的结尾。因此，这个操作的复杂度仍然是二次的。低效率的严重程度与目的地的大小成比例增加，与连接字符串的长度成反比。

### OpenBSD strlcpy 和 strlcat

为了应对利用 strcpy 和 strcat 函数的弱点以及上面讨论的 strncpy 和 strncat 的一些缺点的缓冲区溢出攻击，OpenBSD 项目在 20 世纪 90 年代末引入了一对替代 API，旨在使字符串复制和连接更加安全[ [2](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2349.htm#strlcpy) ]。

```
	size_t **strlcpy** (char* restrict, const char* restrict, size_t);
	size_t **strlcat** (char* restrict, const char* restrict, size_t);
```

strncpy 和 strlcpy 的主要区别在于返回值:前者返回指向目标的指针，而后者返回复制的字符数。另一个区别是 strlcpy 总是在目的地存储恰好一个 NUL。要连接 s1 和 s2，strlcpy 函数的用法如下。

```
	size_t n = strlcpy (d, s1, dsize);
	dsize -= n;
	d += n;
	strlcpy (d, s2, dsize);
```

这使得 strlcpy 在使用和复杂性方面与 snprintf 不相上下(当然，snprintf 的开销虽然不变，但要大得多)。

strlcpy 和 strlcat 函数在 OpenBSD 之外的其他系统上也可用，包括 Solaris 和 Linux(在 BSD 兼容性库中),但是因为它们不是由 POSIX 指定的，所以它们不是普遍存在的。

### posix 内存

POSIX 还定义了另一个函数，它具有上面讨论的所有理想属性，可以用来解决这个问题。

```
	void* **memccpy** (void* restrict dst, const void* restrict src, int c, size_t n);
```

该函数结合了 memcpy、memchr 的属性以及上面讨论的 API 的最佳方面。

*   像 memchr 一样，它扫描源序列，寻找由它的一个参数指定的字符的第一次出现。该字符可以有任何值，包括零。
*   像 strlcpy 一样，它将指定数量的字符从源序列复制(最多)到目标序列，而不超出它。这解决了关于 strncpy 和 stpncpy 的低效抱怨。
*   与 stpcpy 和 stpncpy 类似(尽管不完全相同),它返回一个指针，该指针刚好经过指定字符的副本(如果存在的话)。(回想一下，stpcpy 和 stpncpy 返回一个指向复制的 nul 的指针。)这样就避免了 strcpy 和 strncpy 固有的低效率。

因此，上面的第一个例子( **strcat** ( **strcpy** (d，s1)，s2))可以使用 memccpy 重写，以避免字符串上的任何冗余传递，如下所示。请注意，通过使用 SIZE_MAX 作为界限，这种重写没有避免原始示例中出现的目的地溢出的风险，应该避免这种风险。

```
	memccpy (memccpy (d, s1, '\0', SIZE_MAX) - 1, s2, '\0', SIZE_MAX);
```

为了避免缓冲区溢出的风险，需要为每个调用确定适当的界限，并作为参数提供。因此，像在 **snprintf** (d，dsize，“%s%s”，s1，s2)调用中那样，受目标大小约束的串联可能会按如下方式计算目标大小。

```
	char *p = memccpy (d, s1, '\0', dsize);
	dsize -= (p - d - 1);
	memccpy (p - 1, s2, '\0', dsize);
```

## 选择解决方案

如果字符串函数不返回第一个参数的值，而是返回一个指向最后一个存储的字符或刚刚超过最后一个存储的字符的指针，那么上面讨论的效率问题就可以解决。然而，在使用了近半个世纪之后改变现有的功能是不可行的。

尽管用现有的 C 标准字符串函数来解决这个问题是不可行的，但是可以通过添加一个或多个不受相同限制的函数来在新代码中减轻这个问题。因为 C 标准的章程正在编纂现有的实践，所以标准化委员会有责任调查这种功能是否已经存在于流行的实现中，如果是，考虑采用它。如上所示，存在几种这样的解决方案。

在上面描述的解决方案中，memccpy 函数是最通用的、效率最优的、受 ISO 标准支持的、甚至在 POSIX 实现之外最广泛可用的，并且争议最小。

相比之下，stpcpy 和 stpncpy 函数不太通用，而且 stpncpy 承受了不必要的开销，因此不符合概述的目标。这些功能可能仍然值得考虑在 C2X 采用，以提高便携性。请参见 [N2352 -将 stpcpy 和 stpncpy 添加到 C2X](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2352.htm) 以获得建议。

OpenBSD strlcpy 和 strlcat 函数虽然是最佳的，但是不够通用，支持范围也不够广泛，并且没有被 ISO 标准指定。

memccpy 函数不仅存在于 UNIX 实现的子集中，它还由另一个 ISO 标准指定，即 ISO/IEC 9945，也称为 IEEE Std 1003.1，2017 Edition，或简称为 POSIX: [memccpy](http://pubs.opengroup.org/onlinepubs/9699919799/functions/memccpy.html) ，它是作为 c 的 XSI 扩展提供的。该函数源自 System V Interface Definition，Issue 1 (SVID 1)，最初发布于 1985 年。

memccpy 甚至可以超越 UNIX 和 POSIX 的实现，例如:

*   [Android 中的 memccpy](https://android.googlesource.com/platform/bionic/+/ics-mr0/libc/string/memccpy.c) ，
*   [在苹果 Mac OS X 的 memccpy](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man3/memccpy.3.html) ，
*   [memccpy](http://developer.blackberry.com/native/reference/core/com.qnx.doc.neutrino.lib_ref/topic/m/memccpy.html) 在黑莓原生 SDK 中，
*   VAX 康柏运行库中的 memccpy ，
*   [memccpy](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/memccpy) 在微软 Visual Studio C 运行时库中，
*   IBM z/OS 中的 [memccpy](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.bpxbd00/rmccpy.htm) 。

下面提供了 memccpy 的一个简单(但效率不高)的参考实现。

```
      void* memccpy (void* restrict dst, const void* restrict src, int c, size_t n)
      {
        void *pc = memchr (src, c, n);
        void *ret;

        if (pc)
        {
          n = (char*)pc - (char*)src + 1;
          ret = (char*)dst + n;
        }
        else
          ret = 0;

        memcpy (dst, src, n);
        return ret;
      }
```

该函数的一个更优化的实现可能如下所示。

```
      void* memccpy (void* restrict dst, const void* restrict src, int c, size_t n)
      {
        const char *s = src;
        for (char *ret = dst; n; ++ret, ++s, --n)
        {
          *ret = *s;
          if ((unsigned char)*ret == (unsigned char)c)
            return ret + 1;
        }
        return 0;
      }
```

通过依赖 memccpy，优化编译器将能够把简单的 **snprintf** (d，dsize，“%s”，s)调用转换成对 **memccpy** (d，s，' \0 '，dsize)的效率最优的调用。以代码大小换取速度，激进的优化器甚至可以将 snprintf 调用转换成一系列这样的 memccpy 调用，如下所示:格式字符串由多个%s 指令组成，中间散布着普通字符，如" %s/%s ":

```
      char *p = memccpy (d, s1, '\0', dsize);
      if (p)
      {
        --p;
        p = memccpy (p, "/", '\0', dsize - (p - d));
        if (p)
        {
          --p;
          p = memccpy (p, s2, '\0', dsize - (p - d));
        }
      }
      if (!p)
        d[dsize - 1] = '\0';
```

## WG14 2019 年 4 月会议后的更新

将 memccpy 和本文中讨论的其他标准函数(除 strlcpy 和 strlcat 之外的所有函数)以及其他两个函数包含在 C 编程语言的下一个版本中的提案已于 2019 年 4 月提交给 C 标准化委员会(见 [3](#N2349) 、 [4](#N2351) 、 [5](#N2352) 和 [6](#N2353) )。委员会决定采纳备忘录，但拒绝了其余的建议。

## 参考

*   【1】[C 弦的悲情状态](https://symas.com/the-sad-state-of-c-strings/)朱德华，2016。
*   【2】[strlcpy 和 strlcat——一致、安全的字符串复制和连接](https://www.usenix.org/legacy/event/usenix99/full_papers/millert/millert.pdf)作者 Todd C. Miller，OpenBSD 项目。
*   【3】[n 2349——更高效的字符串复制和连接](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2349.htm)。
*   n 2351–将 strnlen 添加到 C2X 。
*   将 stpcpy 和 stpncpy 添加到 C2X。
*   n 2353–将 strdup 和 strdnup 添加到 C2X。

*Last updated: August 11, 2019*