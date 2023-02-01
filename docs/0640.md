# 的。Linux 上的. NET 进程类

> 原文：<https://developers.redhat.com/blog/2019/10/29/the-net-process-class-on-linux>

在这篇文章中，我们将看看。NET 的`Process`类。我们将回顾如何以及何时使用它的基础知识，然后讨论 Windows 和 Linux 之间的用法差异，并指出一些注意事项。本文涵盖了[中的行为。网芯 3.0](https://developers.redhat.com/blog/2019/10/17/new-features-in-net-core-3-0-on-linux/) 。

## 基础知识

[进程](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=netcore-3.0)类表示一个正在运行的进程的实例。您可以使用它来启动使用`Process.Start`的新进程，或者通过静态的`GetProcessById`、`GetProcesses`、`GetProcessesByName`方法来获得正在运行的进程。

当启动一个新的`Process`时，启动该过程的所有信息都设置在一个`ProcessStartInfo`实例(PSI)上。PSI 有类似于`FileName`和`Arguments`的属性来设置要执行的程序及其参数。`UseShellExecute`允许你打开文档。`RedirectStandard{Input/Output/Error}`允许您写入/读取标准 I/O 流。`Environment` / `EnvironmentVariables`和`WorkingDirectory`允许你控制环境变量和工作目录。

。NET Core 2.1 (/netstandard 2.1)增加了一个`ArgumentList`属性。`Arguments`属性是一个字符串，要求用户使用 [Windows 命令行转义规则](https://docs.microsoft.com/en-us/cpp/cpp/parsing-cpp-command-line-arguments?view=vs-2019)(例如，使用双引号来分隔参数)。属性`ArgumentsList`是一个保存独立参数的`Collection`。`Process.Start`会负责将这些传递到底层平台。瞄准时建议使用`ArgumentList`而不是`Arguments`。NET Core 2.1+/netstandard2.1+。

下面的例子显示了使用单个参数 *hello world* 启动`echo`应用程序，然后等待进程终止。

```
using var process = Process.Start(
    new ProcessStartInfo
    {
        FileName = "echo",
        ArgumentList = { "hello world" }
    });
process.WaitForExit();

```

## Linux 上不支持

Linux 和 throw `PlatformNotSupportedException`不支持`ProcessStartInfo`的以下属性:`PasswordInClearText`、`Domain`、`LoadUserProfile`、`Password`。

也不支持使用`machineName`过载`GetProcessById`、`GetProcesses`、`GetProcessesByName`从远程机器检索进程。

在`Process`上，不支持设定工作设定限制(`MinWorkingSet`、`MaxWorkingSet`)。在`ProcessThread`(通过`Process.Threads`获得)上，不能设置`PriorityLevel` / `ProcessorAffinity`。

## 杀戮过程

可以通过调用`Process.Kill`来停止进程。在 Linux 上，这是通过发送`SIGKILL`信号来实现的，它告诉内核立即终止应用程序。不可能发送一个`SIGTERM`信号，请求应用程序优雅地终止。

自从。NET Core 3.0，如果进程正在终止或者已经终止，`Process.Kill`不再抛出`Win32Exception` / `InvalidOperationException`。如果你的目标更早。NET 核心版本(或。NET Framework)，您应该添加一个`try/catch`块来处理这些异常。

。NET Core 3.0 给接受 bool `entireProcessTree`的`Process.Kill`方法增加了一个重载。当设置为`true`时，进程的后代也将被杀死。

## 使用外壳执行

属性可以用来打开文档。 *shell* 是指用户的图形化 Shell，而不是像`bash`那样的命令行 Shell。将此设置为`true`意味着如同用户双击文件一样。当`ProcessStartInfo.FileName`引用一个可执行文件时，它将被执行。当它引用一个文档时，它将使用默认程序打开。例如，一个`.ods`文件将用 *LibreOffice Calc* 打开。你可以将`FileName`设置为 http-uri(比如*https://redhatloves.net*)来打开浏览器并显示网站。

Unix shell 脚本被操作系统(OS)视为真正的可执行文件。这意味着不需要设置`UseShellExecute`。这种方法不同于 Windows `.bat`文件，后者需要 Windows shell 来找到解释器。

在 Windows 上，`UseShellExecute`允许通过设置`ProcessStartInfo.Verb`对文档进行其他操作(如打印)。在其他操作系统上，该属性被忽略。

## 文件名解析

在`Process.FileName`上设置相对文件名时，文件将被解析。解决步骤取决于`UseShellExecute`是否被设置。在非 Windows 平台上实现的解析行为与 Windows 类似。

当`UseShellExecute`设置为`false`时:

*   在本机应用程序目录中找到该文件。
*   在进程的工作目录中找到该文件。
*   在路径中查找文件。

本机应用目录是执行 *dotnet* 时的`dotnet`安装目录。当使用本机 apphost 时，它是 apphost 目录。

当`UseShellExecute`设置为`true`时:

*   在`ProcessStartInfo.WorkingDirectory`中找到一个可执行文件。如果没有设置，则使用进程工作目录。
*   在路径中查找可执行文件。
*   使用 shell `open`程序，并向其传递相对路径。

重要的是要知道，在这两种情况下，目录被搜索，这可能是不安全的(可执行目录，工作目录)。您可能希望实现自己的解决方案，并总是将`FileName`设置为绝对路径。

## 重定向流

当`UseShellExecute`设置为`false`时，您可以使用`ProcessStartInfo.RedirectStandard{Input/Output/Error}`重定向标准输入、输出和错误。相应的`Encoding`属性(如`StandardOutputEncoding`)允许您设置流的编码。

除非您正在运行或启动一个交互式应用程序(比如启动`vi`)，否则您应该重定向流并处理它们。

注意，像`Process.StandardOutput.ReadAsync`和`Process.BeginOutputReadLine`这样的异步方法使用`ThreadPool`来进行异步读取，这意味着它们在等待应用程序输出时会阻塞一个`ThreadPool`线程。如果你有很多产出不多的`Processes`，这种方法会导致`ThreadPool`饥荒。这也是 Windows 上的一个问题。

如果您使用`Begin{Output/Error}ReadLine`并调用`WaitForExit`，该方法将等待，直到所有标准输出/错误被读取(并且相应的`{Output/Error}DataReceived`事件被发出)。当存在保持重定向流打开的后代时，这种方法会导致对已退出进程的调用被阻止。

## 进程名

在 Linux 上，当执行一个 shell 脚本时，`Process.ProcessName`保存脚本的名称。类似地，`Process.GetProcessesByName`将匹配脚本名称。无论进程是本机可执行文件、脚本还是包装本机可执行文件的脚本，此功能都有助于识别进程。

## 进程退出

进程占用了内核中的一些资源。在 Windows 上，这个信息是被引用计数的，这允许多个用户使用一个[进程句柄](https://docs.microsoft.com/en-us/windows/win32/procthread/process-handles-and-identifiers)来保存信息。在 Unix 上，该信息只有一个所有者。首先，它是父进程，当父进程死亡时，它是`init` (pid 1)进程(或者使用 [PR_SET_CHILD_SUBREAPER](http://man7.org/linux/man-pages/man2/prctl.2.html) 承担了这个责任的进程)。拥有进程是负责清理内核资源的进程(又名收割子进程)。。NET Core 在子进程终止时立即获取它们。

当资源被清理后，关于进程的信息将不再被检索。返回运行时信息的属性在该点抛出`InvalidOperationException`。在 Windows 上，您可以在进程退出后检索`StartTime`、`{Privileged,Total,User}ProcessorTime`。在 Linux 上，这些属性也会抛出`InvalidOperationException`。

在 Linux 上，`Process.ExitCode`只对直系子女有效。对于其他进程，它根据`Process`的状态返回`0`或抛出`InvalidOperationException`。

如果在容器中运行，通常没有初始化进程。这意味着没有人会成为孤儿。这样的子进程会持续消耗内核资源。网络核心永远不会认为他们退出。当应用程序的后代寿命超过其父代时，会出现此问题。如果您是这种情况，您应该在容器中添加一个 init 进程。使用`docker/podman run`时，可以使用`--init`标志添加一个。

## 结论

在本文中，我们解释了。NET 在 Linux 上的进程类。我们讨论了基本用法、不支持的行为、与 Windows 的区别以及其他需要注意的事情。

*Last updated: July 1, 2020*