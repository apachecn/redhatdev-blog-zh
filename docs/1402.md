# 连接中。网络核心至 D 总线

> 原文：<https://developers.redhat.com/blog/2017/09/18/connecting-net-core-d-bus>

D-Bus 是一个 Linux 消息总线系统。许多系统守护程序(如 systemd、PulseAudio、bluez)和桌面服务都可以通过 D-Bus 进行控制。有些应用程序可以通过全局系统总线访问，而其他应用程序则通过每个用户的登录会话总线访问。

各种流行的框架和语言都有更高级别的绑定。 *Tmds。DBus* 是一个. NET 实现。该库基于针对单声道/的 *dbus-sharp/ndesk-dbus，*。NET 2.0。Tmds。DBus 利用中引入的 async/await(基于任务的异步模式)。NET 4.5/C# 5.0。该库面向 netstandard1.5，这意味着它运行在。NET 4.6.1+和。网芯 1.0+。

要使用 D-Bus 服务，必须将其功能描述为 C#接口。因为这是一个繁琐且容易出错的任务，Tmds。DBus 自带了一个 cli 工具来为我们做这件事。该工具查询服务并自动生成接口代码。。需要 NET Core 2.0 作为开发依赖项来运行该工具。

如果你没有。NET Core，你可以在 www.dot.net/core 找到安装说明。或者，在 Fedora 上，你可以安装 [DotNet SIG 包](https://developer.fedoraproject.org/tech/languages/csharp/dotnet-installation.html)。

在这篇文章中，我们将构建一个小应用程序，当网络接口改变状态时，它会写一个控制台消息。为了检测状态变化，我们使用 NetworkManager 守护进程的 D-Bus 服务。我们将执行代码生成的步骤，然后增强生成的代码。

我们首先使用 dotnet cli 创建一个新的控制台应用程序:

```
$ dotnet new console -o netmon
$ cd netmon
```

现在我们添加对*TMD 的引用。DBus* 和*TMD。 *netmon.csproj* 中的 DBus.Tool* 。我们还设置了语言版本，这样我们就可以使用异步 Main (C# 7.1)。

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <LangVersion>7.1</LangVersion>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Tmds.DBus" Version="0.5.0" />
    <DotNetCliToolReference Include="Tmds.DBus.Tool" Version="0.5.0" />
  </ItemGroup>
</Project>
```

现在我们*恢复*来获取这些依赖关系:

```
$ dotnet restore
```

接下来，我们使用 *list* 命令来查找关于网络管理器服务的一些信息。*列出服务*显示可用的服务。使用*列表对象*我们可以看到服务公开的对象。该命令打印每个对象的路径，后跟它实现的接口。

```
$ dotnet dbus list services --bus system | grep NetworkManager
org.freedesktop.NetworkManager
$ dotnet dbus list objects --bus system --service org.freedesktop.NetworkManager | head -2
/org/freedesktop : org.freedesktop.DBus.ObjectManager
/org/freedesktop/NetworkManager : org.freedesktop.NetworkManager
```

命令的输出向我们展示了*org . free desktop . network manager*服务在系统总线上，并且在*/org/free desktop/network manager*处有一个入口点对象，它实现了*org . free desktop . network manager*。

现在我们调用 *codegen* 命令为 NetworkManager 服务生成 C#接口。

```
$ dotnet dbus codegen --bus system --service org.freedesktop.NetworkManager
```

这将生成一个*网络管理器。本地文件夹中的 DBus.cs* 文件。

我们更新了 *Program.cs* 来使用一个*异步 Main* 并实例化一个*in network manager*代理对象。该代理是在*连接上创建的。系统*。这个单体使得在整个应用程序中共享同一个*连接*变得很容易。

```
using System;
using Tmds.DBus;
using NetworkManager.DBus;
using System.Threading.Tasks;
namespace DBusExample
{
  class Program
  {
    static async Task Main(string[] args)
    {
      Console.WriteLine("Monitoring network state changes. Press Ctrl-C to stop.");
      var systemConnection = Connection.System;
      var networkManager = systemConnection.CreateProxy<INetworkManager>("org.freedesktop.NetworkManager", "/org/freedesktop/NetworkManager");
      // TODO: watch state changes
      await Task.Delay(int.MaxValue);
    }
  }
}
```

当我们在*网络管理器中查看*网络管理器*界面时。* DBus.cs，我们看到它有一个 *GetDevicesAsync* 方法。

```
Task<ObjectPath[]> GetDevicesAsync();
```

该方法正在返回*对象路径[]* 。这些路径引用了 D-Bus 服务的其他对象。我们可以将它们与传递了 *IDevice* 类型的 *CreateProxy* 一起使用。相反，我们将更新该方法，以反映它正在返回 *IDevice* 对象。

```
Task<IDevice[]> GetDevicesAsync();
```

现在，我们可以添加代码来迭代设备，并添加状态更改的信号处理程序:

```
foreach (var device in await networkManager.GetDevicesAsync())
{
  var interfaceName = await device.GetInterfaceAsync();
  await device.WatchStateChangedAsync(
  change => Console.WriteLine($"{interfaceName}: {change.oldState} -> {change.newState}"));
}
```

当我们运行程序并更改网络接口(例如打开/关闭 WiFi)时，会出现通知:

```
$ dotnet run
Press any key to close the application.
wlp4s0: 100 -> 20
```

如果我们查阅 *StateChanged* 信号的文档，我们会发现神奇常数的含义: [NMDeviceState](https://developer.gnome.org/NetworkManager/stable/nm-dbus-types.html#NMDeviceState) 。

我们可以用 C#来模拟这种枚举:

```
enum DeviceState : uint
{
  Unknown = 0,
  Unmanaged = 10,
  Unavailable = 20,
  Disconnected = 30,
  Prepare = 40,
  Config = 50,
  NeedAuth = 60,
  IpConfig = 70,
  IpCheck = 80,
  Secondaries = 90,
  Activated = 100,
  Deactivating = 110,
  Failed = 120
}
```

我们将 enum 添加到*网络管理器。DBus.cs* 然后更新 *WatchStateChangedAsync* 的签名，这样它就使用 *DeviceState* 而不是 *uint* 。

```
Task<IDisposable> WatchStateChangedAsync(Action<(DeviceState newState, DeviceState oldState, uint reason)> action);
```

当我们再次运行应用程序时，我们会看到更多有意义的消息:

```
$ dotnet run
Press any key to close the application.
wlp4s0: Activated -> Unavailable
```

这个例子到此结束。我们已经介绍了 Tmds 的基本用法。DBus 和 Tmds.DBus.Tool。了解更多关于 Tmds 的信息。查看 [GitHub 项目](https://github.com/tmds/Tmds.DBus)和[freedesktop.org 网站](https://www.freedesktop.org/wiki/Software/dbus/)。

*Last updated: September 3, 2019*