# 正在执行。NET 核心功能在一个单独的进程中

> 原文：<https://developers.redhat.com/blog/2019/07/17/executing-net-core-functions-in-a-separate-process>

在本文中，我们将看看`[Tmds.ExecFunction](https://github.com/tmds/Tmds.ExecFunction)`，这是一个允许开发人员在单独的进程中轻松执行. NET 核心函数的库。

## 用例

在我们开始编写代码之前，让我们先来看几个场景，在这些场景中，将一个函数作为一个独立的进程来执行是有意义的。进程有一些全局状态，比如环境变量和工作目录。大多数时候，这种全局状态不会引起问题:当它有意义时，库提供一个 API，允许用户覆盖这些值。不过，对于测试来说，情况就不同了。为了验证应用程序是否正确地使用全局状态作为默认状态，我们需要修改它。这个因素是一个问题，因为状态是由测试宿主进程和在该进程中运行的其他测试共享的。我们可以通过在我们可以完全控制的独立进程中运行代码来解决这个问题。

第二类用例发生在我们希望代码以不同于父进程的生存期运行时。例如，假设我们想要启动一个进程，并确保它不会比。NET parent(即使在崩溃时)。我们可以通过在两者之间放置一个小进程来监视父进程，并在父进程终止时杀死子进程。另一个例子是实现[双分叉](http://thelinuxjedi.blogspot.com/2014/02/why-use-double-fork-to-daemonize.html)，它在基于脚本的`init`管理器中使用，以确保守护进程比启动它的 shell 存活得更久。大多数发行版(像 [Fedora](https://getfedora.org/) 和 [Red Hat Enterprise Linux](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) )现在都使用`systemd`作为它们的`init`管理器，不需要守护进程来双叉。

## `Tmds.ExecFunction`

`Tmds.ExecFunction`是一个受启发的图书馆。NET Core 的`[RemoteExecutor](https://github.com/dotnet/arcade/tree/6a34948f7bdbc3ceb2fb16441b49f7748a462646/src/Microsoft.DotNet.RemoteExecutor/src/Microsoft.DotNet.RemoteExecutor)`，用在[。NET Core 的类库](https://github.com/dotnet/corefx/)用于编写需要单独进程的测试。这个库可以在 NuGet.org 的[上获得，所以我们可以使用:](https://www.nuget.org)

```
dotnet add package Tmds.ExecFunction
```

现在，让我们使用它:

```
ExecFunction.Run(() => Console.WriteLine("Hello world!"));
```

我们使用了`ExecFunction.Run`并传递给它一个 lambda，它打印了`Hello world!`lambda 在一个单独的进程中执行。当子进程终止时，`Run`方法返回。`ExecFunction`还提供了一个`Start`方法，该方法返回一个`System.Diagnostics.Process`，然后在流程开始时立即返回。还提供了一个`RunAsync`方法，该方法返回一个`Task`，当进程终止时该方法完成。

我们传递的函数可以有一个. NET `Main`签名:一个`void` / `string[]`参数和一个`void` / `int` / `Task` / `Task`返回类型。

例如，我们可以传递一些参数:

```
ExecFunction.Run(
    (string[] args) => Console.WriteLine($"Hello {args[0]}"),
    new string[] { "world!" });

```

我们还可以控制环境变量、工作目录等。通过传递配置函数:

```
ExecFunction.Run(
    () => Console.WriteLine($"HOME={Environment.GetEnvironmentVariable("HOME")}"),
    o => o.StartInfo.Environment["HOME"] = "/tmp");

```

`FunctionExecutor`类也使得在多次调用中重用相同的配置变得容易。例如，我们可以在 [xUnit](https://xunit.net/) 测试中如下使用它:

```
private FunctionExecutor Executor = new FunctionExecutor(
    o =>
    {
            o.StartInfo.RedirectStandardError = true;
            o.OnExit = p =>
            {
                    if (p.ExitCode != 0)
                    {
                string message = $"Function execution failed with exit code: {p.ExitCode}" + Environment.NewLine +
                                p.StandardError.ReadToEnd();
                throw new Xunit.Sdk.XunitException(message);
                    }
            };
    });

[Fact]
public void Test()
{
    Executor.Run(
        () =>
        {
                Assert.Equal("/tmp", Environment.GetEnvironmentVariable("HOME"));
        },
        o => o.StartInfo.Environment["HOME"] = "/tmp"
    );
}

```

到目前为止，我们一直假设您使用`dotnet`可执行文件来托管您的应用程序；例如，通过使用`dotnet test`启动 xUnit 测试。为了让`ExecuteFunction`在应用程序主机上工作(也就是说，您将应用程序构建到本机二进制文件中)，我们需要在`Main`函数中添加一个钩子:

```
static int Main(string[] args)
{
    if (ExecFunction.IsExecFunctionCommand(args))
    {
            return ExecFunction.Program.Main(args);
    }
    else
    {
            ExecFunction.Run(() => System.Console.WriteLine("Hello world!"));
            return 0;
    }
}

```

## 结论

在本文中，我们展示了如何执行。NET 核心代码很容易在一个单独的进程中使用`[Tmds.ExecFunction](https://github.com/tmds/Tmds.ExecFunction)`，而且这样做的时候会很有用。现在，探索你能让这个库做什么。

*Last updated: September 3, 2019*