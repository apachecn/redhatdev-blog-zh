# 开始使用。红帽企业版 Linux 8.1 中的 NET Core

> 原文：<https://developers.redhat.com/blog/2019/11/25/getting-started-with-net-core-in-red-hat-enterprise-linux-8-1>

最近发布的 Red Hat Enterprise Linux 8.1 中一个令人兴奋的特性是[。网芯 3.0](https://developers.redhat.com/blog/2019/10/17/new-features-in-net-core-3-0-on-linux/) 。在本文中，我们将快速了解如何使用？NET Core on [红帽企业 Linux 8](https://developers.redhat.com/rhel8/) 。我们将涵盖安装。NET 核心 rpm 并使用基于 RHEL 的[通用基础映像](//developers.redhat.com/blog/tag/ubi/”)容器映像。

## 正在安装。RHEL 8 上的网络核心包

与 [RHEL 8](https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-developer-cheat-sheet/) ，。NET Core 包含在 [AppStream 库](//developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)中，默认情况下在 RHEL 8 系统上启用。至少有两个版本的[。NET Core](https://developers.redhat.com/topics/dotnet/) 已经可以在 RHEL 8 上使用，更多的将会随着他们的发布而增加。

的多个版本。NET Core 可以并行安装(并排)。您可以挑选的组件。NET Core (SDK，Runtime)你只需要安装这些。安装一个组件将安装它的所有依赖项。例如，安装一个. NET 核心 SDK 也会安装相应的。NET 核心运行时以及任何其他附加的 SDK 依赖项。

您可以安装特定版本的。NET Core SDK:

```
dnf install dotnet-sdk-2.1

```

或者

```
dnf install dotnet-sdk-3.0

```

通常，您可以安装。NET Core SDK 版本`x.y`使用:

```
dnf install dotnet-sdk-x.y

```

如果你对开发不感兴趣。NET 核心应用程序，而不只是运行它们，您可以跳过 SDK，只安装特定版本的。NET 核心运行时。例如:

```
	dnf install dotnet-runtime-2.1

```

或者

```
	dnf install dotnet-runtime-3.0

```

通常，您可以安装。NET Core 运行时版本`x.y`使用:

```
dnf install dotnet-runtime-x.y

```

从……开始。NET Core 3.0，您还可以安装 ASP.NET 核心运行时，它允许您运行依赖于框架的 ASP.NET 核心应用程序:

```
dnf install aspnetcore-runtime-3.0

```

## 跑步。网络核心

一旦你安装了。在 RHEL 8 上，你可以简单地开始使用`dotnet`命令。为了确保。NET Core 已安装，请尝试:

```
	dotnet --info

```

这应该会显示更多关于。NET Core，包括安装的特定组件:

```
.NET Core SDK (reflecting any global.json):
 Version:   3.0.100
 Commit:    04339c3a26

Runtime Environment:
 OS Name:     rhel
 OS Version:  8
 OS Platform: Linux
 RID:         rhel.8-x64
 Base Path:   /usr/lib64/dotnet/sdk/3.0.100/

Host (useful for support):
  Version: 3.0.0
  Commit:  7d57652f33

.NET Core SDKs installed:
  2.1.509 [/usr/lib64/dotnet/sdk]
  3.0.100 [/usr/lib64/dotnet/sdk]

.NET Core runtimes installed:
  Microsoft.AspNetCore.App 3.0.0 [/usr/lib64/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 2.1.13 [/usr/lib64/dotnet/shared/Microsoft.NETCore.App]
  Microsoft.NETCore.App 3.0.0 [/usr/lib64/dotnet/shared/Microsoft.NETCore.App]

To install additional .NET Core runtimes or SDKs:
  https://aka.ms/dotnet-download

```

我们现在可以使用。NET Core SDK 来创建、构建、发布和运行一个简单的 Hello World 应用程序:

```
$ mkdir HelloWorld
$ cd HelloWorld/
$ dotnet new console

Welcome to .NET Core 3.0!
---------------------
SDK Version: 3.0.100

----------------
Explore documentation: https://aka.ms/dotnet-docs
Report issues and find source on GitHub: https://github.com/dotnet/core
Find out what's new: https://aka.ms/dotnet-whats-new
Learn about the installed HTTPS developer cert: https://aka.ms/aspnet-core-https
Use 'dotnet --help' to see available commands or visit: https://aka.ms/dotnet-cli-docs
Write your first app: https://aka.ms/first-net-core-app
--------------------------------------------------------------------------------------
Getting ready...
The template "Console Application" was created successfully.

Processing post-creation actions...
Running 'dotnet restore' on /HelloWorld/HelloWorld.csproj...
  Restore completed in 50.91 ms for /HelloWorld/HelloWorld.csproj.

Restore succeeded.

$ dotnet publish --configuration Release --runtime rhel.8-x64 --self-contained false
Microsoft (R) Build Engine version 16.3.0+0f4c62fea for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restore completed in 49.95 ms for /HelloWorld/HelloWorld.csproj.
  HelloWorld -> /HelloWorld/bin/Release/netcoreapp3.0/rhel.8-x64/HelloWorld.dll
  HelloWorld -> /HelloWorld/bin/Release/netcoreapp3.0/rhel.8-x64/publish/
$ dotnet bin/Release/netcoreapp3.0/rhel.8-x64/publish/HelloWorld.dll  
Hello World!

```

请参见。NET 核心文档以获取更多信息，包括参考资料、示例和教程。

## 使用。NET Core 基于 RHEL 8 的容器映像

用[红帽企业 Linux 8](https://developers.redhat.com/rhel8/) ，。NET Core 也可用于基于 RHEL 8 的容器映像，称为通用基本映像。您可以使用容器映像来开发和部署您的。容器化环境下的 NET 核心应用，比如 [OpenShift](//developers.redhat.com/products/openshift/overview) 和 [Kubernetes](//developers.redhat.com/topics/kubernetes/) 。

要获得. NET Core 2.1 SDK 容器，请使用:

```
registry.access.redhat.com/ubi8/dotnet-21

```

要获得. NET Core 3.0 SDK 容器，请使用:

```
registry.access.redhat.com/ubi8/dotnet-30

```

要获取. NET Core 2.1 运行时容器，请使用:

```
registry.access.redhat.com/ubi8/dotnet-21-runtime

```

要获取. NET Core 3.0 运行时容器，请使用:

```
registry.access.redhat.com/ubi8/dotnet-30-runtime

```

## 跑步。净核心集装箱

您可以使用。你的容器管道中基于 RHEL 8 的容器。您可以使用它来部署到云环境。您还可以使用它来构建源到图像(也称为 s2i)应用程序。

例如，让我们在一个容器中创建、构建和运行一个 Hello World 风格的应用程序。创建一个包含以下内容的`Dockerfile`:

```
FROM registry.access.redhat.com/ubi8/dotnet-30

RUN dotnet --info && \
	dotnet new console -o HelloWorld && \
	cd HelloWorld && \
	dotnet publish --configuration Release

ENTRYPOINT dotnet HelloWorld/bin/Release/netcoreapp3.0/publish/HelloWorld.dll

```

您可以使用 podman 或 docker 命令来构建和运行它:

```
$ podman build -t hello .
STEP 1: FROM registry.access.redhat.com/ubi8/dotnet-30
Getting image source signatures
Copying blob d4639cd2c710 done
Copying blob 1fa946e8e839 done
Copying blob 340ff6d7f58c done
Copying blob 0e8ea260d026 done               	 
Copying config ca818f6208 done   
Writing manifest to image destination
Storing signatures
STEP 2: RUN dotnet --info && 	dotnet new console -o HelloWorld && 	cd HelloWorld && 	dotnet publish --configuration Release
.NET Core SDK (reflecting any global.json):
 Version:   3.0.100
 Commit:	04339c3a26

Runtime Environment:
 OS Name: 	rhel
 OS Version:  8
 OS Platform: Linux                                  	 
 RID:     	rhel.8-x64
 Base Path:   /usr/lib64/dotnet/sdk/3.0.100/

Host (useful for support):
  Version: 3.0.0
  Commit:  7d57652f33

.NET Core SDKs installed:
  3.0.100 [/usr/lib64/dotnet/sdk]

.NET Core runtimes installed:
  Microsoft.AspNetCore.App 3.0.0 [/usr/lib64/dotnet/shared/Microsoft.AspNetCore.App]
  Microsoft.NETCore.App 3.0.0 [/usr/lib64/dotnet/shared/Microsoft.NETCore.App]

To install additional .NET Core runtimes or SDKs:
  https://aka.ms/dotnet-download
Getting ready...
The template "Console Application" was created successfully.

Processing post-creation actions...
Running 'dotnet restore' on HelloWorld/HelloWorld.csproj...  
  Restore completed in 51.76 ms for /opt/app-root/src/HelloWorld/HelloWorld.csproj.

Restore succeeded.

Microsoft (R) Build Engine version 16.3.0+0f4c62fea for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restore completed in 14.11 ms for /opt/app-root/src/HelloWorld/HelloWorld.csproj.
  HelloWorld -> /opt/app-root/src/HelloWorld/bin/Release/netcoreapp3.0/HelloWorld.dll
  HelloWorld -> /opt/app-root/src/HelloWorld/bin/Release/netcoreapp3.0/publish/
164ce90cf7892154ab5a6a00f8d5f890dd26ffaabe0b6dd58c4005603fe325d4
STEP 3: ENTRYPOINT dotnet HelloWorld/bin/Release/netcoreapp3.0/publish/HelloWorld.dll
STEP 4: COMMIT hello
1bd4ceedbcec15de4d99ee79ad5860d6f97031da1f4db32f7d408274262f8bec

$ podman run -it hello
Hello World!

```

## 支持和生命周期

总的来说，。Red Hat Enterprise Linux 上的 NET Core 试图遵循[。微软](//dotnet.microsoft.com/platform/support/policy/dotnet-core)建立的 NET Core 生命周期。去了解更多。NET 核心生命周期关于 RHEL 8，访问[红帽企业 Linux 8 应用流生命周期](//access.redhat.com/support/policy/updates/rhel8-app-streams-life-cycle)页面。

如果您遇到任何错误，请[在 Bugzilla](//bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%208&component=dotnet”) 中将其报告为 bug。

*Last updated: January 13, 2022*