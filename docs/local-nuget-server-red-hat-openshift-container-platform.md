# 通过 Red Hat OpenShift 使用本地 NuGet 服务器

> 原文：<https://developers.redhat.com/blog/2019/01/08/local-nuget-server-red-hat-openshift-container-platform>

NuGet 是。NET 包管理器。默认情况下。NET Core SDK 将使用来自 nuget.org 网站的软件包。

在本文中，您将了解如何在 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/) (RHOCP)上部署 NuGet 服务器。我们将使用它作为缓存服务器，并看到它加快了我们的构建。在此之前，我们将探索一些通用的 NuGet 概念，并了解为什么使用本地 NuGet 服务器是有意义的。

## 努格特

NuGet 包是。NET 的库:它们允许我们打包编译好的代码，并在不同的应用程序中使用。为了共享这些包，我们让它们可以通过 HTTP 访问。例如，当我们消费来自 NuGet 的包裹时，我们使用来自 https://api.nuget.org/v3/index.json. NuGet feed 的包裹

SDK `dotnet restore`命令检索构建项目所需的包。为了覆盖使用 NuGet.config 的默认设置，我们可以使用一个 [NuGet.config 文件](https://docs.microsoft.com/en-us/nuget/reference/nuget-config-file)，或者为`restore`命令指定`--source`参数。

执行恢复时，我们可以使用多个馈送。例如，一个提要可以是 NuGet 的公共存储库，而另一个提要可以指向一个本地 NuGet 服务器，该服务器托管内部开发的包。

这是第一个用例:*托管私有包*。注意，一个 NuGet 服务器可以托管多个不同的提要。这允许不同的团队/项目有他们自己的提要。您还可以使用这个功能为开发构建和发布构建提供单独的提要。

第二个用例是*缓存包*。在这种情况下，提要是上游提要的缓存实例。通过在本地缓存包，我们可以减少恢复项目的时间。我们也不再依赖上游服务器的可用性。如果上游服务器允许删除包，我们仍会将它们保存在缓存中。缓存服务器还减少了从互联网上获取的包的数量。

## 本地托管选项

*微软的 NuGet 服务器*是开源的，[但是不能在 Linux 上运行](https://github.com/NuGet/NuGetGallery/issues/3390)。像 *JFrog Artifactory* 和 *Sonatype Nexus* 这样的库管理器是功能丰富的包管理器，也支持 NuGet 提要。

如果我们正在寻找一个可以在 Linux 容器中运行的轻量级、仅支持 NuGet 的选项，那么 *BaGet* 是一个有趣的选择。这是一个支持 v3 协议的[开源](https://github.com/loic-sharma/BaGet) NuGet 服务器，它是使用 ASP.NET 核心实现的。

## RHOCP 和 NuGet

RHOCP 可以建立我们的。NET 核心应用程序。的。NET Core builder 接受许多环境变量来控制它的行为。我们对`DOTNET_RESTORE_SOURCES`感兴趣，它做了以下事情:

*指定在恢复操作过程中使用的以空格分隔的 NuGet 软件包源列表。这将覆盖 NuGet.config 文件中指定的所有源。*

因此，要使用本地 NuGet 服务器，我们可以将这个变量设置为提要 URL。如果您使用的是 Microsoft NuGet Server、JFrog Artifactory 或 Sonatype Nexus，请查看创建 NuGet 提要的产品文档。在下一节中，我们将解释如何在 RHOCP 上部署 BaGet，并将其用作 NuGet 的缓存 NuGet 服务器。

## 将 BaGet 与 RHOCP 一起使用

对于接下来的步骤，我假设。NET Core 支持已经添加到您的 RHOCP 安装中，如[安装图像流](https://access.redhat.com/documentation/en-us/net_core/2.2/html/getting_started_guide/gs_dotnet_on_openshift#install_imagestreams)中所述。步骤中使用的`DOTNET_NAMESPACE`变量应该设置为包含。NET 核心生成器图像。

使用 RHOCP CLI ( `oc`)，在您的 RHOCP 项目中导入 BaGet 模板:

```
$ oc create -f https://raw.githubusercontent.com/redhat-developer/s2i-dotnetcore/master/templates/community/dotnet-baget.json
template.template.openshift.io/dotnet-baget-persistent created
template.template.openshift.io/dotnet-baget-ephemeral created

```

正如我们所看到的，这创建了两个模板:一个用于持久的，另一个用于短暂的 BaGet 部署。

这些模板接受许多参数。例如，要查看来自`dotnet-baget-persistent`的参数，请执行:

```
$ oc process --parameters dotnet-baget-persistent

```

| 参数名称 | 描述 | 默认 |
| --- | --- | --- |
| `NAME` | 分配给此模板中定义的所有前端对象的名称。 | `nuget` |
| `MIRROR_PACKAGESOURCE` | 将从此服务器中检索在本地找不到的包。 | `https://api.nuget.org/v3/index.json` |
| `NUGET_API_KEY` | 将此设置为推送包所需的密码。 |  |
| `DELETION_BEHAVIOR` | 将此项设置为- `Unlist`使包裹不可被发现，或设置为`HardDelete`从存储中移除包裹。 | `Unlist` |
| `MEMORY_LIMIT` | 最大内存量。NET 核心容器可以使用。 | `512Mi` |
| `VOLUME_CAPACITY` | 可用于存储数据的卷空间，例如 512 兆字节、21 兆字节 | `512Mi` |
| `DOTNET_IMAGE_STREAM_TAG` | 用于构建代码的图像流标记。 | `dotnet:2.2` |
| `NAMESPACE` | RHOCP 命名空间。NET builder ImageStream 驻留在。 | `openshift` |
| `SOURCE_REPOSITORY_URL` | 包含应用程序源代码的存储库的 URL。 | `https://github.com/loic-sharma/BaGet.git` |
| `SOURCE_REPOSITORY_REF` | 如果您不使用默认分支，请将其设置为分支名称、标记或存储库的其他引用。 | `v0.1.29-prerelease` |
| `DOTNET_STARTUP_PROJECT` | 将其设置为项目文件(例如，`csproj`)或包含单个项目文件的文件夹。 | `src/BaGet` |

我们看到，默认情况下，该服务将镜像来自 nuget.org 的包(`MIRROR_PACKAGESOURCE`)。我们的容器被分配了 512 兆字节的内存(`MEMORY_LIMIT`)。我们提供 512MiB ( `VOLUME_CAPACITY`)用于永久存储。NuGet 服务将从`https://github.com/loic-sharma/BaGet.git v0.1.29-prerelease` ( `SOURCE_REPOSITORY_URL`和`SOURCE_REPOSITORY_REF`)开始构建。`NUGET_API_KEY`为空:将包推送到这个服务器不需要任何键。我们服务的主机名是`nuget`。

我们现在将在项目中部署服务器:

```
$ oc new-app dotnet-baget-ephemeral -p NAMESPACE=$DOTNET_NAMESPACE

```

RHOCP 将从源代码构建 BaGet。我们可以看到构建的进度:

```
$ oc logs -f bc/nuget

```

当构建完成时，NuGet 服务将被部署，并可以通过 URL `http://nuget:8080/v3/index.json`在项目内部使用。

让我们看看这个本地 NuGet 服务如何影响构建时间。

我们将部署 [dotnet-example](https://access.redhat.com/documentation/en-us/net_core/2.2/html/getting_started_guide/gs_dotnet_on_openshift#sample_apps) 模板并触发许多构建:

```
$ oc new-app dotnet-example -p NAMESPACE=$DOTNET_NAMESPACE
$ oc start-build dotnet-example;oc start-build dotnet-example; oc start-build dotnet-example

```

在 OpenShift 控制台中，我们得到了没有本地 NuGet 服务器的构建时间的概述:

[![Overview of the build times without the local NuGet server](img/a01deae32655c4aef4ec18468d19b35d.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/non_cached.png)

现在，我们将部署相同的应用程序，并将本地 NuGet 服务器用作缓存:

```
$ oc new-app dotnet-example -p NAMESPACE=$DOTNET_NAMESPACE -p NAME=dotnet-example-nuget -p DOTNET_RESTORE_SOURCES=http://nuget:8080/v3/index.json
$ oc start-build dotnet-example-nuget; oc start-build dotnet-example-nuget; oc start-build dotnet-example-nuget; oc start-build dotnet-example-nuget

```

[![Deploying the same application and use the local NuGet server as a cache](img/7ccd1b2f825a8891bfa088731e6163ae.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/12/cached.png)

正如我们从图表中看到的，使用本地 NuGet 服务器显著减少了构建时间。直接使用 nuget.org 的构建在构建时间上有很大的差异，这与本地服务器无关。我们看到我们第一次构建本地服务器花了一点时间。在这个构建过程中，从 nuget.org 检索包并缓存在本地。这些包随后在后续的构建中被重用。

## 结论

在本文中，我们研究了在本地部署 NuGet 服务器的选项。然后，我们看到了如何使用本地 NuGet 服务器来提高速度。NET 核心构建在 Red Hat OpenShift 容器平台上。

## 附加。Red Hat 开发人员博客上的 NET 核心文章

*   [建筑。使用 S2I 的. NET 核心容器映像](https://developers.redhat.com/blog/2018/12/13/building-net-core-container-images-using-s2i/)
*   [跨平台定位特殊文件夹。网络应用](https://developers.redhat.com/blog/2018/11/07/dotnet-special-folder-api-linux/)
*   [在 OpenShift 上使用 Kubernetes 就绪性和活性探针对 ASP.NET Core 2.2 进行健康检查](https://developers.redhat.com/blog/2018/12/21/asp_dotnet_core_kubernetes_health_check_openshift/)
*   [宣布。面向红帽平台的 NET Core 2.2](https://developers.redhat.com/blog/2018/12/05/announcing-net-core-2-2-for-red-hat-platforms/)
*   [在 Red Hat OpenShift 上运行微软 SQL Server](https://developers.redhat.com/blog/2018/09/25/sql-server-on-openshift/)
*   [固定。使用 HTTPS 的 OpenShift 上的 NET Core](https://developers.redhat.com/blog/2018/10/12/securing-net-core-on-openshift-using-https/)
*   [提高。NET Core Kestrel 使用 Linux 特定传输的性能](https://developers.redhat.com/blog/2018/07/24/improv-net-core-kestrel-performance-linux/)

*Last updated: September 3, 2019*