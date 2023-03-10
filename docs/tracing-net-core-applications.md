# 追踪。网络核心应用

> 原文：<https://developers.redhat.com/blog/2019/12/23/tracing-net-core-applications>

在本文中，我们将看看从[开始收集和检查事件的不同方式。NET 核心运行时](https://developers.redhat.com/products/dotnet/overview)和基础类库(BCL)。

## 事件监听器

`[EventListener](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventlistener?view=netcore-3.0)`类允许我们获取正在运行的应用程序的事件。让我们通过一个示例应用程序来学习如何使用它。我们的应用程序执行 HTTP get 并打印接收到的响应的长度。

```
using var httpClient = new HttpClient();
string response = await httpClient.GetStringAsync("http://redhatloves.net/");
Console.WriteLine($"Received response with length {response.Length}");

```

的。NET Core 基类(如`HttpClient`)使用`EventSource`类记录事件。每个`[EventSource](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.0)`都有一个名字。`EventSources`是在我们使用某些功能时产生的。为了知道哪些资源是可用的，我们将创建一个`EventListener`并覆盖`OnEventSourceCreated`。

```
sealed class EventSourceCreatedListener : EventListener
{
    protected override void OnEventSourceCreated(EventSource eventSource)
    {
        base.OnEventSourceCreated(eventSource);
        Console.WriteLine($"New event source: {eventSource.Name}");
    }
}

```

当我们实例化这个类时，基本构造函数将为所有现有的源调用`OnEventSourceCreated`。当添加新源时(例如，当我们第一次使用`HttpClient`)，也会为它们调用`OnEventSourceCreated`。

我们在应用程序中添加了一个实例:

```
using var eventSourceListener = new EventSourceCreatedListener();
using var httpClient = new HttpClient();
...

```

当我们运行应用程序时，输出如下:

```
New event source: Microsoft-Windows-DotNETRuntime
New event source: System.Runtime
New event source: Microsoft-System-Net-Http
New event source: System.Diagnostics.Eventing.FrameworkEventSource
New event source: Microsoft-Diagnostics-DiagnosticSource
New event source: Microsoft-System-Net-Sockets
New event source: Microsoft-System-Net-NameResolution
New event source: System.Threading.Tasks.TplEventSource
New event source: System.Buffers.ArrayPoolEventSource
New event source: Microsoft-System-Net-Security
New event source: System.Collections.Concurrent.ConcurrentCollectionsEventSource

```

当我们创建`EventSourceCreatedListener`时，`Microsoft-Windows-DotNETRuntime`和`System.Runtime`就出现了。其他事件源是在我们使用`HttpClient`时创建的。HTTP (Microsoft-System-Net-Http)有一个事件源，我们还看到了名称解析(Microsoft-System-Net-name resolution)、安全(Microsoft-System-Net-Security)和套接字(Microsoft-System-Net-Sockets)的事件源。让我们用另一个监听器将事件从 HTTP 源打印到`Console`。

我们创建一个`EventListener`来打印特定`EventSource`的事件:

```
sealed class EventSourceListener : EventListener
{
    private readonly string _eventSourceName;
    private readonly StringBuilder _messageBuilder = new StringBuilder();

    public EventSourceListener(string name)
    {
        _eventSourceName = name;
    }

    protected override void OnEventSourceCreated(EventSource eventSource)
    {
        base.OnEventSourceCreated(eventSource);

        if (eventSource.Name == _eventSourceName)
        {
            EnableEvents(eventSource, EventLevel.LogAlways, EventKeywords.All);
        }
    }

    protected override void OnEventWritten(EventWrittenEventArgs eventData)
    {
        base.OnEventWritten(eventData);

        string message;
        lock (_messageBuilder)
        {
            _messageBuilder.Append("<- Event ");
            _messageBuilder.Append(eventData.EventSource.Name);
            _messageBuilder.Append(" - ");
            _messageBuilder.Append(eventData.EventName);
            _messageBuilder.Append(" : ");
            _messageBuilder.AppendJoin(',', eventData.Payload);
            _messageBuilder.AppendLine(" ->");
            message = _messageBuilder.ToString();
            _messageBuilder.Clear();
        }
        Console.WriteLine(message);
    }
}

```

在`OnEventSourceCreated`中，我们允许接收特定源的事件。我们不是针对特定的日志级别(EventLevel)进行筛选。LogAlways)或特定关键字(EventKeywords。全部)。在`OnEventWritten`中，我们将事件打印到`Console`。

现在我们为`Microsoft-System-Net-Http`事件添加一个实例。

```
using var httpEventListener = new EventSourceListener("Microsoft-System-Net-Http");
...

```

当我们再次运行我们的程序时，我们从 HTTP `EventSource`中得到许多事件。例如，我们看到我们的请求被重定向，我们看到关于 SSL 协商的信息。

```
…
Event Microsoft-System-Net-Http - HandlerMessage : 55530882,6044116,0,SendAsyncCore,Received response: StatusCode: 301, ReasonPhrase: 'Moved Permanently'
…
<- Event Microsoft-System-Net-Http - HandlerMessage : 16294043,41622463,0,TraceConnection,HttpConnection(HttpConnectionPool https://developers.redhat.com:443). SslProtocol:Tls12, NegotiatedApplicationProtocol:, NegotiatedCipherSuite:TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, CipherAlgorithm:Aes256, CipherStrength:256, HashAlgorithm:None, HashStrength:0, KeyExchangeAlgorithm:DiffieHellman, KeyExchangeStrength:0, LocalCertificate:, RemoteCertificate:[Subject]
  CN=openshift.redhat.com, O=Red Hat, L=Raleigh, S=North Carolina, C=US

[Issuer]
  CN=GeoTrust RSA CA 2018, OU=www.digicert.com, O=DigiCert Inc, C=US
…

```

名为`Microsoft-Windows-DotNETRuntime`的`EventSource`提供来自运行时的事件，如垃圾收集器(GC)、实时编译器(JIT)和`ThreadPool`。

## 点网跟踪

`[dotnet-trace](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)`是一款全新的全球工具。网芯 3.0。使用`dotnet-trace`,我们可以从正在运行的应用程序中创建一个跟踪。

要安装 dotnet-trace 运行:

```
dotnet tool install --global dotnet-trace

```

可以使用以下命令创建跟踪:

```
dotnet trace collect -p <pid> [--buffersize 256] [-o <outputpath>] [--providers <providerlist>] [--profile <profilename>] [--format NetTrace]

```

我们需要指定我们想要跟踪的程序的`<pid>`。或者，我们可以指定一个缓冲区大小(默认值:256MB)。输出路径是跟踪文件的文件名。`providers`和`profile`允许我们指定我们感兴趣的事件。A `profile`是预定义数量的提供商。您可以使用`list-profiles`子命令查看可用的概要文件:

```
$ dotnet trace list-profiles
cpu-sampling     - Useful for tracking CPU usage and general .NET runtime information. This is the default option if no profile or providers are specified.
gc-verbose       - Tracks GC collections and samples object allocations.
gc-collect       - Tracks GC collections only at very low overhead.

```

如输出所示，如果您没有指定`providers`和`profile`，那么默认使用`cpu-samping`概要文件。

可以使用逗号分隔来指定多个提供者。默认情况下，将为任何关键字和详细级别捕获事件。我们可以通过在提供者名称后面添加一个`:[<keyword_hex_nr>]:<loglevel_nr>]`后缀来改变这一点。可以省略`keyword_hex_nr`来表示所有关键字。例如，在[事件级别跟踪`Microsoft-System-Net-Http`T3。信息性](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventlevel?view=netcore-3.0)，我们可以指定:`--providers Microsoft-System-Net-Http::4`。星号可用于跟踪所有事件(即`--providers '*'`)。

有些提供者可以配置额外的键值对。这些被添加到提供者名称`:[key1=value1][;key2=value2]`中。当`EventSource`关联了`[EventCounters](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventcounter?view=netcore-3.0)`时，可以使用`EventCounterIntervalSec`键设置报告间隔。

默认格式(nettrace)适用于 Windows [PerfView](https://github.com/microsoft/perfview) 工具。如果你在做性能分析，你可以使用`speedscope`格式，它可以在任何平台上使用[https://www.speedscope.app/](https://www.speedscope.app/)网站打开，或者离线使用 npm [speedscopeapp](https://www.npmjs.com/package/speedscope) 包打开。当您以`speedscope`格式登录时，轨迹首先被捕获为`nettrace`格式，然后被转换为`speedscope`格式。这种转换也可以在命令行上使用`dotnet trace convert`来完成。

例如，这就是如何跟踪名为`web`的. NET 核心应用程序的默认配置文件(cpu 采样)并输出一个`speedscope`文件。

```
$ dotnet trace collect -p $(pidof web) --format speedscope

Provider Name                           Keywords            Level               Enabled By
Microsoft-System-Net-Http               0xFFFFFFFFFFFFFFFF  Verbose(5)          --providers

Process        : /tmp/web/bin/Debug/netcoreapp3.0/web
Output File    : /tmp/trace/trace.nettrace

[00:00:00:08]   Recording trace 2.3301   (MB)
Press  or  to exit...

Trace completed.
Writing:        /tmp/trace/trace.speedscope.json
Conversion complete

```

如工具所示，您可以使用 Ctrl+C 或 Enter 停止追踪。关于使用 speedscope 的更多信息，请查看 Adam Sitnick 的博客文章。

## EventPipe 环境变量

dotnet-trace 在应用程序运行时启用跟踪输出。通过使用环境变量，我们还可以从应用程序开始就启用跟踪。这也让我们能够追踪流程树。

| 名字 | 描述 |
| COMPlus_EnableEventPipe | 启用/禁用事件管道。 |
| COMPlus_EventPipeOutputPath | 跟踪文件的完整路径，不包括文件名。 |
| COMPlus_EventPipeCircularMB | 以兆字节为单位的循环缓冲区大小(默认值:1024 MB)。 |
| compelles _ event pipeconfig | 事件管道的配置。 |

`EventPipeConfig`与`dotnet trace` `providers`参数的格式相同。`EventPipeOutputPath`是指一个文件夹。这允许多个进程通过在文件名中包含进程 pid 来写入跟踪文件。`EventPipe`使用的是`nettrace`格式。

例如，要从控制台应用程序中捕获所有事件，我们可以运行:

```
COMPlus_EnableEventPipe=1 COMPlus_EventPipeConfig=* dotnet console.dll

```

## TraceEvent 库

如果你想编写处理这些跟踪文件的工具，你可以使用微软的[。diagnostics . tracing . trace event](https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent/)库。这个库还允许我们读取 Linux 上的 nettrace 文件:

```
# add the package:
#      dotnet add package Microsoft.Diagnostics.Tracing.TraceEvent

using System;
using Microsoft.Diagnostics.Tracing;
using Microsoft.Diagnostics.Tracing.Etlx;
using Microsoft.Diagnostics.Tracing.Parsers;

namespace console
{
    class Program
    {

        static void Main(string[] args)
        {
            if (args.Length < 1)
            {
                Console.WriteLine("You must specify a nettrace file.");
                return;
            }

            using (var traceLog = new TraceLog(TraceLog.CreateFromEventPipeDataFile(args[0])))
            {
                var traceSource = traceLog.Events.GetSource();

                traceSource.AllEvents += delegate (TraceEvent data) {
                    Console.WriteLine("{0}", data);
                };

                traceSource.Process();

           }
        }
    }
}

```

## 微软。扩展.日志记录

ASP.NET 岩心使用`Microsoft.Extensions.Logging`包进行测井。你可以在[登录中找到更多相关信息。网芯和 ASP.NET 芯](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.0)。多个接收器可以被配置为[日志提供者](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.0#built-in-logging-providers)。`EventSource`日志记录器提供程序(默认包含)允许在跟踪输出中包含 ASP.NET 核心日志输出。 [EventSource 部分](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.0#event-source-provider)描述了如何将`Microsoft-Extensions-Logging` EventSource 与`dotnet-trace`一起使用，并控制各个记录器的格式和日志级别。

## 网络计数器

[点网计数器](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)是一个新的全球性工具。网芯 3.0。用`dotnet-counters`我们可以实时监控[运行中的事件计数器](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventcounter?view=netcore-3.0)。NET 核心应用程序。
安装计数器运行:

```
dotnet tool install --global dotnet-counters

```

可以使用以下命令执行监控。

```
dotnet-counters monitor -p <pid> [--refreshInterval <intervalSec>] [<counter_list>]

```

当未指定计数器列表时，默认的。NET 运行时计数器显示有关 GC、`ThreadPool`、锁争用、活动计时器和异常计数的信息。

`<counter_list>`可以指定为`provider_name[:counter_name]`的空格分隔列表。当省略`<counter_name>`时，包括提供者的所有计数器。

## 结论

在本文中，您了解了如何从[中捕获事件。NET 核心运行时](https://developers.redhat.com/products/dotnet/docs-and-apis)和 BCL 使用`EventListener`、`[dotnet-trace](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)`和`EventPipe`环境变量。使用`LoggingEventSource`我们可以包括来自`Microsoft.Extensions.Logging`的事件(如 ASP.NET 核心)。要查看和分析这些事件，您可以使用 windows 上的 [PerfView](https://github.com/microsoft/perfview) ，跨平台 [speedscope](https://www.speedscope.app/) ，或者使用 [Microsoft 构建您自己的跨平台工具。diagnostics . tracing . trace event](https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent/)。使用`[dotnet-counters](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace)`我们可以实时查看事件计数器。

*Last updated: July 1, 2020*