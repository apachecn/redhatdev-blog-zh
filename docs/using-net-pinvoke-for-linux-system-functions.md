# 使用。NET PInvoke for Linux 系统函数

> 原文：<https://developers.redhat.com/blog/2019/03/25/using-net-pinvoke-for-linux-system-functions>

如果你已经用。NET，您可能会发现自己处于这样一种情况:框架没有提供您需要的 API。当这种情况发生时，您首先需要识别系统 API，然后使用 PInvoke 使它们可用。像[pinvoke.net](https://pinvoke.net)这样的网站为许多 Win32 API 函数提供了可复制和粘贴的代码片段。

。NET 平台调用(PInvoke)使得使用本地库变得容易。在本文中，我们将看看如何使用 PInvoke 实现 [Linux](https://developers.redhat.com/topics/linux/) 系统功能。

## PInvoking Linux

如果您不熟悉 PInvoke，我们来看一个简单的例子:

```
[DllImport("mylibrary")]
public static extern int foo();

```

这使得来自本地库`mylibrary`的函数`foo`可用。该函数不接受任何参数，并返回一个`int`。。NET 负责封送参数类型。可以在这些签名中使用托管类型(如`strings`)，这将被自动封送。

PInvoke 工作在应用程序二进制接口(ABI)级别:它遵守运行它的平台的二进制调用约定。

Linux 是一个 UNIX 风格的操作系统。它提供了许多由 [POSIX](https://en.wikipedia.org/wiki/POSIX) 和 [SUS](https://en.wikipedia.org/wiki/Single_UNIX_Specification) 规定的 APIs 然而，这些 API 是使用 C 编程语言定义的。它们不是二进制级别的 API(ABIs)。

例如，标准描述了应该有正值来指示某些类型的错误。这些值被命名(例如，`EAGAIN`)，但是它们的值没有被定义。这意味着不同的平台可以(也将会)对这些名称有不同的值。这使得不可能向 C#文件添加一个对所有平台都具有正确值的`const int`。

再比如`struct`，在 API 中用来定义若干字段；但是顺序不固定，允许平台添加额外的字段。所以，我们不能在 C#中描述一个与所有平台上的原生`struct`匹配的`struct`。

解决这个问题的一般方法是引入一个本机 shim 库。该库提供了自己的一组函数、常数和结构，这些函数、常数和结构与平台无关，具有固定的 ABI。这个 API 被. gets PInvoked。在库中，这个 API 调用本地函数。

如果您想 PInvoke system C 库而不使用 shim，您可以使用`libc`作为库名。的。NET Core runtime 将其映射到 system C 库。例如，我们可以如下公开 [kill](http://man7.org/linux/man-pages/man2/kill.2.html) 函数:

```
[DllImport("libc", SetLastError = true))]
public static extern int kill(int pid, int sig);

```

`SetLastError`表示该函数使用 [errno](http://man7.org/linux/man-pages/man3/errno.3.html) 来指示哪里出错了。[元帅。GetLastWin32Error](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.getlastwin32error?view=netcore-2.1) 可以用来检索`errno`。

## 单声道。Posix 标准

单声道。Posix.NETStandard 包提供系统功能。它使用本机垫片在许多平台上工作。如果我们从 nuget.org 的[下载这个包，我们可以看看里面:](https://www.nuget.org)

```
$ unzip -l mono.posix.netstandard.1.0.0.nupkg
Archive:  mono.posix.netstandard.1.0.0.nupkg
  Length  	Date	Time	Name
---------  ---------- -----   ----
  	516  02-28-2018 15:18   _rels/.rels
  	814  02-28-2018 15:18   Mono.Posix.NETStandard.nuspec
	13376  02-28-2018 15:18   lib/net40/Mono.Posix.NETStandard.dll
	13376  02-28-2018 15:18   ref/net40/Mono.Posix.NETStandard.dll
   189504  02-28-2018 15:18   ref/netstandard2.0/Mono.Posix.NETStandard.dll
   189504  02-28-2018 15:18   runtimes/linux-arm/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   183240  02-28-2018 15:18   runtimes/linux-arm/native/libMonoPosixHelper.so
   189504  02-28-2018 15:18   runtimes/linux-arm64/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   246736  02-28-2018 15:18   runtimes/linux-arm64/native/libMonoPosixHelper.so
   189504  02-28-2018 15:18   runtimes/linux-armel/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   236480  02-28-2018 15:18   runtimes/linux-armel/native/libMonoPosixHelper.so
   189504  02-28-2018 15:18   runtimes/linux-x64/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   920882  02-28-2018 15:18   runtimes/linux-x64/native/libMonoPosixHelper.so
   189504  02-28-2018 15:18   runtimes/linux-x86/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   766620  02-28-2018 15:18   runtimes/linux-x86/native/libMonoPosixHelper.so
   189504  02-28-2018 15:18   runtimes/osx/lib/netstandard2.0/Mono.Posix.NETStandard.dll
   535888  02-28-2018 15:18   runtimes/osx/native/libMonoPosixHelper.dylib
   189504  02-28-2018 15:18   runtimes/win-x64/lib/netstandard2.0/Mono.Posix.NETStandard.dll
  1495800  02-28-2018 15:18   runtimes/win-x64/native/libMonoPosixHelper.dll
	87600  02-28-2018 15:18   runtimes/win-x64/native/MonoPosixHelper.dll
   189504  02-28-2018 15:18   runtimes/win-x86/lib/netstandard2.0/Mono.Posix.NETStandard.dll
  1256296  02-28-2018 15:18   runtimes/win-x86/native/libMonoPosixHelper.dll
   400120  02-28-2018 15:18   runtimes/win-x86/native/MonoPosixHelper.dll
  	592  02-28-2018 15:18   [Content_Types].xml
  	692  02-28-2018 15:18   package/services/metadata/core-properties/4b5c471cadba4490965015071f90289f.psmdcp
 	9475  10-11-2018 20:23   .signature.p7s
---------                 	-------
  7874039                 	26 files

```

`libMonoPosixHelper.*`文件是本机 shim 库。`runtimes`下的目录是支持的平台。有 Linux 对`arm`、`arm64`、`armel`、`x64`、`x86`的支持。该软件包还提供了对 Windows 和 OS X 的支持。Linux 库是基于 *glibc* (GNU C 库)的 Linux 版本，如 Fedora 和[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux/)(RHEL)。有些东西不能在基于 *musl、*的 Linux 版本上工作，比如 Alpine。。使用这个包的. NET 核心应用程序将根据它们运行的平台在`runtimes`文件夹下选择合适的变体。这基于[运行时标识符(RID)图](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog)。

# Tmds。LibC

不久前，我在博客上写了一篇关于 Kestrel 的 Linux 专用传输的文章。该库使用的特定于 Linux 的 API 不是由提供的。到目前为止，该库已经使用了一个本机填充程序来实现这些功能。NuGet 包中有一个为 *glibc* x64 编译的版本。这意味着传输不能在其他架构或其他 C 库上工作。

如前所述，解决这个问题的一种方法是为不同的平台重新编译 shim。虽然这在理论上很容易做到，但在实践中，需要做相当多的工作。在每个平台上，都需要构建本地库。那么这些库需要被收集在一个 NuGet 包中。我想尝试一种不同的方法:完全托管的实现。

托管实现在托管代码中捕获平台 ABI。对于每个平台，我们构建一个与该平台的 ABI 相匹配的程序集。平台之间的差异很小，所以我们可以在它们之间共享大量代码。

捕获平台 ABIs 的一个好处是我们的。NET API 可以非常接近 C API。我们不需要引入填充结构或填充常数。

例如，用于更改套接字选项的`setsockopt`选项有这样的 C API:

```
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);

```

C# API 看起来像这样:

```
public static int setsockopt(int socket, int level, int optname, void* optval, socklen_t optlen);

```

如你所见，签名几乎是一样的。

因为这些系统函数可能对其他开发人员有用，所以我将它们收集在一个单独的库中，名为 *Tmds。LibC* 。你可以在 [GitHub](https://github.com/tmds/Tmds.LibC) 上找到源代码，包发布到[NuGet.org](https://www.nuget.org/packages/Tmds.LibC/)。Kestrel Linux Transport 已更新为使用该库。传输不需要处理平台细节，它现在运行在 arm32、arm64 和 x64 上。

# 结论

在本文中，我们研究了。NET PInvoke 以及 PInvoke 在 Linux 和 Windows 上的不同之处。然后我们探索了单声道。Posix.NETStandard 使用本机垫片在各种平台上提供了一组 Posix 函数。最后，我们看了一下 *Tmds。LibC，*，它使用完全托管的实现为许多架构提供 Linux 系统功能。

*Last updated: September 3, 2019*