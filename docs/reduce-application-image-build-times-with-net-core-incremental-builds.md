# 使用减少应用程序映像构建时间。净核心增量构建

> 原文：<https://developers.redhat.com/blog/2019/04/08/reduce-application-image-build-times-with-net-core-incremental-builds>

在之前的一篇文章中，我们谈到了使用容器来构建。NET 核心应用程序映像，使我们的构建可移植和可复制。因为每个构建都是从头开始的，所以需要花费一些时间来下载和提取 NuGet 包。

减少构建时间的一个方法是[添加一个本地 NuGet 服务器](https://developers.redhat.com/blog/2019/01/08/local-nuget-server-red-hat-openshift-container-platform/)；这使得包离构建机器更近，从而减少了下载包的时间。在本文中，我们将了解。NET 核心 S2I 生成器可以进一步减少生成时间。

的。NET 核心 S2I 生成器现在支持增量生成。当我们进行增量构建时，构建器将重用以前构建的应用程序映像中的 NuGet 包。因此，应用程序映像的第一次构建将从 NuGet 服务器获取包，随后的构建将重用这些包。

为了使用[源到图像](https://github.com/openshift/source-to-image/) ( `s2i`)工具执行增量构建，我们需要传递`--incremental`标志。默认情况下。NET 核心 S2I 生成器将删除 NuGet 包以减小应用程序映像的大小。为了保留这些包，我们需要将`DOTNET_INCREMENTAL`环境变量设置为`true`。

在我的开发机器上，为`dotnet new mvc`-模板执行`s2i`构建给出了这些构建时间:

| S2I | 总时间 | 恢复时间 |
| 首次构建 | 35s | 16.5 秒 |
| 连续构建 | 24.5 秒 | 0.5s |

我们看到，由于恢复时间减少，构建时间减少了；然而，总时间并没有减少相同的数量。这是因为我们花费了额外的时间从以前的应用程序映像中提取 NuGet 包。注意，先前的应用程序映像已经存在于构建机器上，所以我们没有花时间从映像注册中心获取它。

[Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/)也可以配置为执行增量构建。让我们用一个免费的[红帽 OpenShift Online](https://manage.openshift.com/) 账号来看看构建时间差。

| 免费 OpenShift 在线帐户 | 总时间 | 恢复时间 |
| 非增量构建 | 2m18s | 28.5 秒 |
| 连续增量构建 | 4m45s | 2s |

在这种情况下，增量构建会比较慢。从 NuGet 服务器获取包比从集群容器映像注册表中检索以前构建的应用程序映像，然后从中提取包要快。

我们再来一次。现在我们将使用一个本地 Red Hat OpenShift 测试集群。以下是构建时间:

| OpenShift 测试集群 | 总时间 | 恢复时间 |
| 非增量构建 | 1m12s | 12.72 秒 |
| 连续增量构建 | 1m53s | 1s |

构建时间更接近，但是同样，由于 NuGet 服务器的高带宽，非增量构建比容器注册更快。

## 结论

在本文中，我们研究了 S2I 新的增量构建特性。NET Core builder。当在开发机器上使用`s2i`时，当先前构建的应用程序映像已经存在于机器上时，增量构建会更快。在 OpenShift 上构建时，情况并非如此。与 NuGet 服务器的带宽相比，增量或非增量构建是否更快取决于集群映像注册表的带宽。

*Last updated: September 3, 2019*