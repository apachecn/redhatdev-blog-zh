# 建筑。使用 S2I 的. NET 核心容器映像

> 原文：<https://developers.redhat.com/blog/2018/12/13/building-net-core-container-images-using-s2i>

[红帽 OpenShift](https://openshift.com) 实现[。NET Core](https://developers.redhat.com/blog/category/dot-net/) 通过源到映像(S2I)构建器提供支持。在本文中，我们将进一步了解如何直接使用这个构建器。使用 S2I，你可以建立。NET 核心应用程序映像，而不必编写定制的构建脚本或 docker 文件。这在你的开发机器上或者作为 [CI/CD](https://developers.redhat.com/blog/category/ci-cd/) 管道的一部分会很有用。

## 构建的容器

容器映像提供了一种高效的机制，以可移植的方式跨云和操作系统分布部署自包含的应用程序。

通过使用构建器映像构建应用程序映像本身，可以以可移植、可再现的方式构建应用程序映像。

S2I 是一个构建可复制应用程序映像的工具包。S2I 使用构建器映像从源代码或预编译的应用程序中生成应用程序映像。

```
application source/        +-------+
prebuilt application  +--> |  s2i  | +-->  application image
                           +-------+
                               ^
                        builder image

```

在 Fedora 和[红帽企业版 Linux](https://developers.redhat.com/products/rhel/overview/) (RHEL)上，安装`source-to-image`包(`yum install source-to-image`)即可获得 S2I。如果你使用的是另一个操作系统，包括 Windows 和 macOS，你可以从 github.com/openshift/source-to-image/releases[下载 S2I。你还需要安装`s2i`使用的`docker`和`git`。](https://github.com/openshift/source-to-image/releases)

## 。网络核心生成器图像

每个都有一个构建器映像。NET 核心版。这些图片是根据 github.com/redhat-developer/s2i-dotnetcore 的文档制作的。

因为。NET Core 2.1，这些图像如下所示:

| 基本的 | 图像类型 | 映像名 |
| --- | --- | --- |
| RHEL 7 | 运行时间 | registry.redhat.io/dotnet/dotnet-21-runtime-rhel7:2.1 |
| RHEL 7 | 软件开发工具包(Software Development Kit) | registry.redhat.io/dotnet/dotnet-21-rhel7:2.1 |
| CentOS 7 | 运行时间 | registry.centos.org/dotnet/dotnet-21-runtime-centos7:latest |
| CentOS 7 | 软件开发工具包(Software Development Kit) | registry.centos.org/dotnet/dotnet-21-centos7:latest |

如果你想找一个不同的。NET 核心版，可以查看[红帽容器目录](https://access.redhat.com/containers/)或者 [CentOS 容器注册表](https://registry.centos.org/)。

要使用 RHEL 7 图像，您需要订阅红帽。对于开发，您可以使用[免费开发订阅](https://developers.redhat.com/register)。使用`docker login registry.redhat.io`命令来配置您的凭证。

## 建筑图像

以下部分描述了三种创建图像的方法。

### 从 git 存储库构建应用程序映像

作为我们的第一个例子，我们将在[github.com/redhat-developer/s2i-dotnetcore-ex](https://github.com/redhat-developer/s2i-dotnetcore-ex)为 ASP.NET 核心应用程序构建一个名为`mywebapp`的应用程序映像。

```
$ s2i build https://github.com/redhat-developer/s2i-dotnetcore-ex registry.redhat.io/dotnet/dotnet-21-rhel7:2.1 mywebapp -r dotnetcore-2.1 -e DOTNET_STARTUP_PROJECT=app -p always

```

当我们运行这个命令时，我们将看到`s2i`检查源代码并构建。NET 核心应用程序映像。命令完成后，我们可以运行新创建的映像:

```
$ docker run --rm -p 8080:8080 mywebapp

```

让我们看看我们传递给`s2i`命令的参数:

| 参数 | 描述 |
| --- | --- |
| registry.redhat.io/dotnet-21-rhel7:2.1 | S2I 建筑形象 |
| https://.../s2i-dotnetcore-ex | git 存储库 |
| -r 点网络核心-2.1 | 存储库中的分支 |
| -e DOTNET_STARTUP_PROJECT=app | 存储库中包含应用程序 csproj 文件的文件夹 |
| -p 始终 | 始终获取最新的构建器图像 |

`-e DOTNET_STARTUP_PROJECT`是传递给 S2I 生成器的环境变量。该构建器支持许多环境变量，允许您自定义其行为。完整列表见[环境变量文档](https://access.redhat.com/documentation/en-us/net_core/2.1/html/getting_started_guide/gs_dotnet_on_openshift#gs_env-var)。

### 从本地资源构建应用程序映像

`s2i`命令的 source 参数也可以引用包含源代码的本地文件夹。在下面的例子中，我们在本地克隆存储库，并将适当的分支签出到`s2i-dotnetcore-ex`文件夹中。然后我们从本地文件夹开始构建。

```
$ git clone -b dotnetcore-2.1 https://github.com/redhat-developer/s2i-dotnetcore-ex
$ s2i build s2i-dotnetcore-ex registry.redhat.io/dotnet/dotnet-21-rhel7:2.1 mywebapp -r dotnetcore-2.1 -e DOTNET_STARTUP_PROJECT=app -p always

```

我们可以像以前一样使用相同的`docker run`命令运行映像。

### 从预构建的应用程序构建应用程序映像

的。NET Core builder 也可用于从预构建的应用程序构建应用程序映像。

为了探索这一点，我们首先发布了`s2i-dotnetcore-ex`应用程序。为此，你需要在你的系统上安装一个. NET SDK([rhel 7](https://access.redhat.com/documentation/en-us/net_core/2.1/html/getting_started_guide/gs_install_dotnet#install_dotnet21)/[centos 7](https://access.redhat.com/documentation/en-us/net_core/2.1/html/getting_started_guide/gs_install_dotnet#install_dotnet21)， [Fedora](https://fedoraloves.net/) ，[其他](https://www.microsoft.com/net/download))。

```
$ git clone -b dotnetcore-2.1 https://github.com/redhat-developer/s2i-dotnetcore-ex
$ cd s2i-dotnetcore-ex/app
$ dotnet publish -c Release /p:MicrosoftNETPlatformLibrary=Microsoft.NETCore.App

```

我们指定参数`MicrosoftNETPlatformLibrary`来使发布的应用程序包含 ASP.NET 核心共享框架组件。这是必要的，因为图像不包含共享框架。

为了创建图像，我们将`publish`文件夹作为`s2i build`命令的源参数进行传递。我们可以像以前一样使用 SDK 构建器映像，但是因为应用程序已经构建好了，所以我们可以使用较小的运行时映像。

```
$ s2i build bin/Release/netcoreapp2.1/publish mywebapp registry.redhat.io/dotnet/dotnet-21-runtime-rhel7:2.1 -p always

```

## 结论

在这篇文章中，你已经学会了如何创造。NET 核心应用程序映像使用`s2i`。这些映像可以直接从 git 存储库、本地资源或预构建的应用程序中构建。

*Last updated: September 3, 2019*