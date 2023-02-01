# 中的新功能。Linux 上的 NET Core 3.0

> 原文：<https://developers.redhat.com/blog/2019/10/17/new-features-in-net-core-3-0-on-linux>

。NET Core 3.0 带来了许多令人兴奋的[新特性](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-core-3-0)，包括一个[新的 C#](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8) 、[改进的性能](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)以及对[构建 Windows 桌面应用](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-preview-1-and-open-sourcing-windows-desktop-frameworks/)(在 Windows 上)的支持。在本文中，我们将为 Linux 和 Linux 容器用户介绍一些有趣的新特性。

### 更快更小的 SDK

SDK 更小更快。以前版本的。NET Core 在构建应用程序时使用了 NuGet 包。这些 NuGet 包包含引用程序集(描述 API)和实现程序集。。NET Core 3.0 使用 SDK 附带的参考包。因为这些包不包含实现，所以它们比 NuGet 包小。这使得 SDK 更小(如果 NuGet 包包含在 SDK 中)或更快(如果 NuGet 包必须从 nuget.org 下载)。

[源构建。NET Core](https://github.com/dotnet/source-build/) (像红帽企业版 Linux，红帽 OpenShift，还有 [Fedora。NET SIG](https://fedoraloves.net/) )现在包含了 ASP.NET 核心框架。这减少了构建 ASP.NET 核心应用程序的时间(不再有 nuget.org 下载)，加速了这些应用程序(AOT 编译)，并通过包更新提供了安全修复(而不是必须重新构建应用程序)。

### 依赖于框架的可执行文件、单个文件发布、修整

在以前的版本中，只有在发布自包含应用程序时才会包含本机可执行文件。现在，依赖于框架的应用程序中也包含了本机可执行文件:

```
$ dotnet new console -o console
$ cd console
$ dotnet publish
$ bin/Debug/netcoreapp3.0/publish/console
Hello World!

```

默认情况下，这个本机可执行文件适用于您正在运行的平台。通过指定运行时 id ( `-r`)，可以为不同的平台(如 Windows)构建本地可执行文件。

```
$ dotnet publish -r win-x64 --no-self-contained
$ ls /tmp/console3/bin/Debug/netcoreapp3.0/win-x64/publish/
console3.deps.json  console3.dll  console3.exe  console3.pdb  console3.runtimeconfig.json

```

注意，为了构建一个依赖于框架的应用程序，我们传递了`--no-self-contained`标志。没有它，应用程序将是自包含的(这意味着它包括运行时)。

如果您想要一个本机可执行文件，它可以跨一系列 Linux 发行版工作，那么您可以将`linux-x64`指定为 rid。这个可执行文件依赖于 GNU C 库，这个库在很多发行版中都有使用(包括 Fedora 和 RHEL)。如果您的可执行文件是基于 musl 的发行版，比如 Alpine，您可以指定`linux-musl-x64` rid。

独立的和依赖于框架的应用程序都支持将应用程序打包到单个本机可执行文件中。为此，您可以设置`PublishSingleFile`属性。

```
$ dotnet publish -r win-x64 /p:PublishSingleFile=true
$ ls -lh bin/Debug/netcoreapp3.0/win-x64/publish/*.exe
-rwxrw-r--. 1 tmds tmds 66M Sep 13 09:08 bin/Debug/netcoreapp3.0/win-x64/publish/console.exe

```

前面的命令将整个应用程序打包到一个独立的 Windows 可执行文件中。从大小(66M)可以看出包含了运行时。

SDK 现在也可以利用 [mono 的链接器](https://github.com/mono/linker)来检测未使用的程序集。当我们添加`PublishTrimmed`属性时，我们的自包含应用程序缩小到 26M。

```
$ dotnet publish -r win-x64 /p:PublishSingleFile=true /p:PublishTrimmed=true
$ ls -lh bin/Debug/netcoreapp3.0/win-x64/publish/*.exe
-rwxrw-r--. 1 tmds tmds 26M Sep 13 09:09 bin/Debug/netcoreapp3.0/win-x64/publish/console.exe

```

### ARM64

。NET Core 增加了对 Linux ARM64 的支持。主要用例是物联网(IoT)场景。

### 串行端口支持

SerialPort 类现在也可以在 Linux 上工作。例如，您可以在运行时使用它。在 Raspberry Pi 或其他嵌入式 Linux 平台上的 NET Core。

### TLS 1.3

什么时候。NET Core 运行在装有 [OpenSSL 1.1.1](https://www.openssl.org/blog/blog/2018/09/11/release111/) (像最近版本的 Fedora，和 RHEL8) [SslStream](https://docs.microsoft.com/en-us/dotnet/api/system.net.security.sslstream?view=netcore-3.0) 和 [HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=netcore-3.0) 的系统上，当对等体支持 TLS 1.3 时使用它。TLS 1.3 改进了连接时间和安全性。

### 建筑系统 d 服务

。NET Core 附带了构建工人的模板，这些模板是长期运行的服务。worker 模板(`dotnet new worker`)有用于构建 [Windows 服务](https://devblogs.microsoft.com/aspnet/net-core-workers-as-windows-services/)和 [systemd 服务](https://devblogs.microsoft.com/dotnet/net-core-and-systemd/)的扩展包。

### readytorun

SDK 现在允许提前编译应用程序。这将在程序集中添加本机代码。这也意味着代码不再需要在第一次执行时进行实时编译，从而减少启动时间。

要发布一个准备运行的应用程序，您需要指定一个运行时标识符(它决定了您将生成的本机代码)和`/p:PublishReadyToRun=true`参数。

```
$ dotnet build -r linux-x64 /p:PublishReadyToRun=true

```

默认情况下，这将发布自包含的应用程序。要发布依赖于框架的应用程序，可以添加`--no-self-contained`。

OpenShift 的。NET Core builder([s2i-dotnetcore](https://github.com/redhat-developer/s2i-dotnetcore))可以通过将新的`DOTNET_PUBLISH_READYTORUN`设置为`true`来构建就绪运行映像。

### 低内存容器中的 GC

。NET Core 3.0 在低内存分配的容器中工作得更好。以前的版本为每个 CPU 分配一个大堆，并根据已用内存和可用内存来执行垃圾收集(GC)。这可能会导致应用程序内存不足(OOM)。[。NET Core 3.0 在创建堆时考虑了内存限制](https://devblogs.microsoft.com/dotnet/using-net-and-docker-together-dockercon-2019-update/)。这意味着堆变小了，堆的数量受到分配给容器的内存的限制。

将 ASP.NET 核心应用程序更改为使用工作站垃圾收集器(GC)是解决这个问题的一种方法(工作站垃圾收集器使用一个较小的堆)。对于，不再需要此解决方法。网芯 3.0。

如果您想知道您的应用程序正在使用哪种类型的 GC:默认情况下，ASP.NET 核心应用程序(使用`csproj`文件中的`Microsoft.NET.Sdk.Web`)使用服务器 GC。控制台应用程序(`Microsoft.NET.Sdk`)默认为工作站 GC。当应用程序在单个处理器上运行时(比如 CPU 分配为 1 或更少的容器)，运行时将自动切换到工作站 GC。

### 超过 64 个 CPU 的机器上的 GC

Windows APIs 基于多达 64 个处理器的处理器组，而 Linux APIs 支持任意数量的处理器。在的早期版本中。NET Core 中，GC 会人为地将 Linux 上的处理器分组，形成处理器组。[默认情况下](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/runtime/gccpugroup-element)，服务器 GC 会将其线程数量限制在单个组中的处理器数量之内(最大 64)。与。NET Core 3.0 中，处理器组仿真从 GC 中移除，服务器 GC 将匹配分配给进程的处理器数量(不限于 64)。

### 巨大的页面支持

在配置了[大页面支持](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt)的系统上。通过将环境变量`COMPlus_GCLargePages`设置为`1`，可以配置 NET GC 来分配巨大的页面。因为应用程序启动时内存被保留，所以 GC 需要知道它可以使用多少内存。在容器中运行时，运行时会受到一些限制。如果你在容器外运行，你需要使用[COMPlus _ GCHeapHardLimit/COMPlus _ GCHeapHardLimitPercent](https://devblogs.microsoft.com/dotnet/running-with-server-gc-in-a-small-container-scenario-part-1-hard-limit-for-the-gc-heap/)来提供这些。

### 诊断工具

与。NET Core 3.0，微软正在[提供用于诊断的](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/)跨平台命令行工具: [dotnet-dump](https://github.com/dotnet/diagnostics/blob/master/documentation/dotnet-dump-instructions.md) (用于收集和分析转储)、 [dotnet-trace](https://github.com/dotnet/diagnostics/blob/master/documentation/dotnet-trace-instructions.md) (用于收集跟踪)、 [dotnet-counters](https://github.com/dotnet/diagnostics/blob/master/documentation/dotnet-counters-instructions.md) (用于实时查看性能计数器)。

例如，下面的命令显示了如何安装`dotnet-dump`，用它从运行中收集转储。NET 核心应用程序`console`，加载转储，并显示托管堆栈跟踪。

```
$ dotnet tool install -g dotnet-dump
$ dotnet dump collect -p $(pidof console)
Writing minidump with heap to /tmp/core_20190911_104217
Complete
$ dotnet dump analyze /tmp/core_20190911_104217                                                                                                                                                                                                                                                                
Loading core dump: /tmp/core_20190911_104217 ...
Ready to process analysis commands. Type 'help' to list available commands or 'help [command]' to get detailed help on a command.
Type 'quit' or 'exit' to exit the session.
> clrstack                                                                                                                                                                                                                                                                                                                   
OS Thread Id: 0x3f4f (0)
        Child SP               IP Call Site
00007FFE05DB3068 00007fe0f046c83c [HelperMethodFrame: 00007ffe05db3068] System.Threading.Thread.SleepInternal(Int32)
00007FFE05DB31B0 00007FE0760B580B System.Threading.Thread.Sleep(Int32)
00007FFE05DB31C0 00007FE0765C007D console.Program.Main(System.String[]) [/tmp/console/Program.cs @ 9]
00007FFE05DB34A8 00007fe0ef98df83 [GCFrame: 00007ffe05db34a8] 
00007FFE05DB39A0 00007fe0ef98df83 [GCFrame: 00007ffe05db39a0] 
>                                                                           

```

如您所见，`console`正从`Program.Main`呼叫`Thread.Sleep`。

### 结论

这里我们讨论了的几个有趣的特性。Linux 和 Linux 容器上的 NET Core 3.0。要了解更多信息，请查看中的[新功能。NET Core 3.0](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-core-3-0) 文档。

*Last updated: July 1, 2020*