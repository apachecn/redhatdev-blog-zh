# NuGet 简介。RHEL 的网络核心

> 原文：<https://developers.redhat.com/blog/2017/08/16/introduction-to-nuget-with-net-core-on-rhel>

## NuGet 简介。网络核心

NuGet 是一个开源的包管理器。网核生态系统。对于那些熟悉 Red Hat Enterprise Linux (RHEL)的人来说，您可以把它看作是将库引入您的。NET 核心项目。在中使用 NuGet 包。NET 核心应用程序主要是通过项目的`.csproj`文件和 dotnet 命令行界面来完成的。

### 仓库

就像 RHEL 一样，NuGet 有自己的库来获取包。默认情况下，当。NET 核心运行时被安装，nuget.org 库被添加到你的系统中。你可以通过查看`~/.nuget/NuGet/NuGet.Config`来了解这一点。

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  </packageSources>
</configuration> 
```

关于各种`NuGet.Config`文件以及如何应用它们的更多信息可以在微软关于[配置 NuGet 行为](https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior)的文档中找到。

### 寻找包裹

找到一个包是一个有机的过程，无疑会随着时间的推移而改变。换句话说，你会找到自己的方法来做这件事，所以我会给你指出正确的方向，并假设你会从那里开始。

我先找了两个地方找包裹。我使用哪一个依赖于我已经知道的关于包的信息，也就是说，我是否知道包的名字或者只是包中的一个类的名字。

如果我已经知道了包的名字，只是想知道当前版本和它所针对的框架版本，我将使用[nuget.org](https://nuget.org)来按名字搜索。例如， [Newtonsoft。Json 包页面](https://www.nuget.org/packages/Newtonsoft.Json)显示当前版本是 10.0.3，目标是。网络标准 1.3。

有时候我只知道一个类或者方法的名字。例如，我可能在 Stack Overflow 上看到过一个代码片段，它引用了`JsonConvert.DeserializeObject`方法。nuget.org 的搜索在这里不太好用，所以微软创造了他们的[反向打包搜索工具](https://packagesearch.azurewebsites.net/)。您可以在这里输入任何类或方法名，搜索结果将链接到[nuget.org](http://nuget.org)上的项目。

请注意，任何开发人员都可以创建一个帐户并将软件包上传到 NuGet。这意味着软件包在用于生产系统之前需要经过审查。如果下载量很高并且软件包正在开发中，通常可以信任它。如有疑问，请务必访问该项目的主页，如果它存在的话。

### 向您的项目添加包

有两种方法可以将包添加到项目中:

1.  使用`dotnet add`命令
2.  手动编辑您的`.csproj`文件

使用`dotnet add`是最简单的，因为它会自动运行包恢复并下载包。为了使用上面的例子，让我们添加 **Newtonsoft。Json** 包给一个项目。在项目的根目录(包含您的`.csproj`文件的目录)中，执行以下命令。

```
$ dotnet add package Newtonsoft.Json --version 10.0.3
Microsoft (R) Build Engine version 15.1.1012.6693
Copyright (C) Microsoft Corporation. All rights reserved.

  Writing /tmp/tmpgud8AG.tmp
info : Adding PackageReference for package 'Newtonsoft.Json' into project '/home/redhat/myproject/myproject.csproj'.
log  : Restoring packages for /home/redhat/myproject/myproject.csproj...
info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project '/home/redhat/myproject/myproject.csproj'.
info : PackageReference for package 'Newtonsoft.Json' version '10.0.3' updated in file '/home/redhat/myproject/myproject.csproj'. 
```

要手动添加项目，您需要编辑项目的`.csproj`文件，然后运行`dotnet restore`。下面是添加 Newtonsoft 后项目文件的样子。Json 包。

```
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp1.1</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="10.0.3"/>
  </ItemGroup>
</Project> 
```

保存文件，然后在项目文件夹中运行`dotnet restore`。这将检查包的兼容性并下载包以供使用。我发现使用第一种方法要容易得多。

### 本地包

如果您正在开发自己的库以便在多个项目中使用，您最终会想要引用这些库。由于 NuGet 是添加对. NET 核心项目的引用的唯一方式，您有两种选择:

1.  您可以创建一个帐户并将您的包裹上传到[nuget.org](http://nuget.org)。
2.  您可以托管一个内部 NuGet 存储库。

在写这篇文章的时候，还没有运行在 Linux 上的 NuGet 托管软件。所以唯一的选择是使用一个目录来存放您的 NuGet 包。注意，这可以是本地机器上的一个目录，也可以是像 NFS 这样的网络文件共享。为了简单起见，下面我将重点介绍一个本地目录。

首先，为您的包创建一个目录。这可以是您的用户可以访问的任何地方，但我选择了/usr/share/nuget。

```
$ sudo mkdir -p /usr/share/nuget
$ sudo chmod 644 /usr/share/nuget 
```

这篇文章并没有介绍如何创建一个 NuGet 包，而是将它们复制到上面创建的目录中。现在你必须说出来。NET 关于这个目录。在您的项目文件夹中，用以下内容创建一个新的`NuGet.Config`。

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="mynuget" value="/usr/share/nuget" />
  </packageSources>
</configuration> 
```

现在，使用您在上一节中选择的方法将一个包添加到您的项目中。

### 结论

我希望这篇文章能帮助你开始使用 NuGet。净核心项目。请在下面评论任何提示或建议！

*Last updated: August 14, 2017*