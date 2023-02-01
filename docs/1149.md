# 红帽峰会:发展中。Red Hat OpenShift 上的 NET 核心应用程序

> 原文：<https://developers.redhat.com/blog/2018/05/08/red-hat-summit-developing-net-core-apps-on-red-hat-openshift>

在 2018 年红帽峰会上，红帽的约翰·奥斯本和微软的 Harold Wong 做了一个演讲:*发展。红帽 OpenShift* 上的 NET 核心应用。

。NET Core 1.0 对 Linux 的可用性是两年前宣布的，但是许多开发人员仍然对。NET 框架和。网芯。会议首先概述了不同之处。简单地说，。NET Framework 是 Windows 开发人员已经使用多年的一组 API 和库，它与 Microsoft Windows 和 Windows GUI APIs 有着非常紧密的联系。另一方面，。NET Core 是一组跨平台的 API，可用于构建可以通过 Xamarin 在 Linux、macOS 或移动设备上运行的应用程序。。网芯 2.0 于去年 8 月发布；参见[唐·申克的文章](https://developers.redhat.com/blog/2017/08/22/quick-introduction-of-net-core-2-0/)。

一个关键的问题是什么时候使用一个而不是另一个。以下是 Harold Wong 的总结:

。NET 框架:

*   最适合 Windows 桌面应用程序
*   覆盖
    *   Windows 窗体和 Windows 演示框架应用程序
    *   ASP.NET Web 窗体
    *   Windows 通信基础框架
    *   特定于 Windows 的 API
*   完全支持 VB.NET 和 F#
*   是在系统范围内安装的，所以每个系统只有一个版本
*   缺乏…的特色。网络核心:
    *   包括第三方 NuGet 包
    *   APICompat 库

。网络核心:

*   适用于跨平台运行的应用程序
*   旨在满足以下需求
    *   容器
    *   微服务
*   可以在 Linux 上开发，但目标是其他平台，如 macOS
*   在规模上具有最佳性能
*   可以并行运行多个版本
*   非常适合以 CLI 为中心的环境

。NET Core 是当你想编写一个在容器中运行的应用程序或服务时可以使用的平台。趁还能跑。NET 框架应用程序放在 Windows 容器中，这通常是不可取的，尤其是在云平台上。Harold 给出的粗略标准是，Linux 容器从大约 30 MB 开始，而 Windows 容器目前从大约 480 MB 开始。当添加到的所有库中时。NET Framework 和应用程序本身，您可能会增加额外的 GB，因此对于一个最小的应用程序来说，这大约是 1.5 GB 的容器。在 Linux 容器中运行的. NET 核心应用程序可以更小、更轻，大约 100-200 MB。

中可用 API 的数量。Net Core 已经增加，从其移植变得越来越容易。NET Famework to。网芯。然而，Harold 提出了一个观点，那就是仅仅因为你可以移植某些东西并不意味着你应该移植。在许多情况下，从整体服务到微服务的重构对许多应用程序来说更有意义。

一个特别甜蜜的地方。NET 是 ASP.NET 应用程序。ASP.NET 被认为是最快的 MVC 框架之一。这是一个创建 RESTful web 服务的简单环境。即使 ASP.NET 应用程序最初是使用。NET 框架，它通常是非常容易移植的。网芯。这为 Windows 开发人员提供了一个很好的途径来扩展 ASP.NET 应用程序。NET 核心并在 Red Hat OpenShift 上部署到云。

在演讲的演示部分，约翰·奥斯本演示了构建和部署。OpenShift 上的 NET Core 2.0 应用。他描述了两种发展模式。一种是容器即服务(CaaS)方法，容器在本地开发环境中构建，然后部署到 OpenShift/Kubernetes 上运行。另一种方法是平台即服务(PaaS)模型，在这种模型中，容器是使用源到映像(S2I)构建在 OpenShift 上构建的，该构建可以是 CI/CD 管道的一部分，该管道通过 GitHub 的 webhooks 从 git commits 启动。这样做的好处是，当堆栈的任何组件发生变化时，构建可以自动开始。

如果你选择 CaaS 路线，在本地构建容器，John 推荐使用 [metaparticle.io](https://metaparticle.io/about/) 。Metaparticle.io 可以减少构建分布式应用程序的一些困难。

对于许多组织来说，部署在 OpenShift 上的优势是能够避免创建与单一云基础设施平台提供的 API 和服务紧密耦合的应用程序代码。通过 OpenShift 中的 Kubernetes 功能，可以将服务位置和安全凭证存储的配置信息从应用程序中分解出来，以便从运行时环境中获取。这允许跨环境使用相同的容器，因为它们没有受到特定于环境的配置细节的影响。

在演示中，John 展示了一个正在部署的应用程序，该应用程序使用了通过 OpenShift 公开的 Azure Service Broker 找到的 Microsoft SQL Server 实例。该应用程序通过 Kubernetes 配置和秘密获得数据库绑定信息，这些信息是通过 OpenShift web sonsole 创建的，无需创建和编辑 YAML 文件。

演示文稿中的其他几点毫无价值:

*   。网芯 3.0 前几天刚公布。正在向添加其他 API。净核心，这将使它更容易移动。NET 框架代码转移到。网芯。
*   微软的 VS 代码——可以在 Windows、macOS 和 Linux 上运行——已经成为构建人员的首选开发环境。NET 核心应用程序。
*   。NET Core 是构建从 Windows 到移动设备再到 Linux 容器的跨平台应用的关键，而且它足够轻量，可以在云平台上轻松部署，最重要的是可以在云平台上进行扩展。
*   OpenShift 可以使用 Azure Service Broker，让你在云中以服务的形式消费微软 SQL Server 等云服务，而不必自己打包管理。
*   约翰·奥斯本是新书《行动中的开放转变》的合著者。红帽峰会上将提供数量有限的预发本。

*Last updated: September 3, 2019*