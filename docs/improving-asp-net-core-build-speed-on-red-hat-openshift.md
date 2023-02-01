# 在 Red Hat OpenShift 上提高 speed 核心构建速度

> 原文：<https://developers.redhat.com/blog/2019/07/10/improving-asp-net-core-build-speed-on-red-hat-openshift>

在以前的文章中，我已经介绍了两种改进[的策略。NET Core](https://developers.redhat.com/products/dotnet/overview) 在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上的构建速度通过减少恢复依赖关系的时间:[添加本地 NuGet 服务器](https://developers.redhat.com/blog/2019/01/08/local-nuget-server-red-hat-openshift-container-platform/)和[使用增量构建](https://developers.redhat.com/blog/2019/04/08/reduce-application-image-build-times-with-net-core-incremental-builds/)。在本文中，我将研究另一种策略:使用包含依赖项的定制基础映像。

## ASP.NET 核心

ASP.NET 核心是构建 web 应用的实际框架。网芯。框架在 GitHub 上是开源的；但是，对于 1.x 和 2.x，它没有第三方(例如，Red Hat)从源代码构建它以匹配 Microsoft 版本所需的基础结构。

从源代码构建是开源项目的一个关键特性。它通过支持其他公司和社区来构建和支持软件，减少了对供应商的锁定。从源代码构建代码的能力是在 Linux 发行版(如 Fedora、Debian 和 Ubuntu)的默认包存储库中包含包的一个要求。即将到来的 ASP.NET 核心 3.0 将从源代码构建！

因为 ASP.NET Core 2 . x 不能从源代码构建，所以它不能包含在 Red Hat Enterprise Linux 包和 Red Hat OpenShift 的包中。NET Core builder ( `s2i-dotnetcore`)。这导致恢复时间很长，因为应用程序需要从 NuGet(或本地 NuGet 服务器)获取和提取这些包。

## aspnet 图像流

作为一个 Red Hat OpenShift 用户，要解决这个限制，您可以构建一个包含 ASP.NET 核心 NuGet 包的自定义映像。我已经编写了一个资源文件来完成这个任务。您可以使用以下方式将其导入 OpenShift:

```
$ oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/aspnet-2.x.json

```

如果你看一下这个文件，你会看到它定义了一个`aspnet:2.1`和`aspnet:2.2`图像流，使用一个`Docker`策略建立在`s2i-dotnetcore`的`dotnet:2.1`和`dotnet:2.2`之上。通过执行`dotnet new web`添加 ASP.NET 核心包。因为我们是在`dotnet`映像流的基础上构建的，所以当使用安全补丁更新基础映像时，`aspnet`流将会自动重建。

导入文件后，需要几分钟来构建图像。

让我们通过使用`dotnet:2.2`和`aspnet:2.2`构建【https://github.com/redhat-developer/s2i-dotnetcore-ex】的[来比较构建速度的差异。](https://github.com/redhat-developer/s2i-dotnetcore-ex)

```
$ oc new-app --name=dotnetapp dotnet:2.2~https://github.com/redhat-developer/s2i-dotnetcore-ex#dotnetcore-2.2 --build-env DOTNET_STARTUP_PROJECT=app
$ oc new-app --name=aspnetapp aspnet:2.2~https://github.com/redhat-developer/s2i-dotnetcore-ex#dotnetcore-2.2 --build-env DOTNET_STARTUP_PROJECT=app

```

当我们进行五次构建时，我们得到这些构建时间:

| 基础图像 | 部 | 最大 | 平均 |
| 点网:2.1 | 43s | 1m30s | 47s |
| aspnet:2.1 | 23s | 31 日 | 24s |

如您所见，使用`aspnet`映像，构建时间减半。此外，构建时间差异要小得多。

## 结论

我们可以大大减少构建时间。NET 核心应用程序，方法是创建包含常见依赖项的自定义生成映像。 [aspnet-2.x.json](https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/aspnet-2.x.json) 提供了将 ASP.NET 2 . x 核心包添加到`s2i-dotnetcore` `dotnet`映像的构建配置。

*Last updated: September 3, 2019*