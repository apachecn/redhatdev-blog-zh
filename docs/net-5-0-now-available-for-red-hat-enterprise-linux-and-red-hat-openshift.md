# 。NET 5.0 现在可用于 Red Hat Enterprise Linux 和 Red Hat OpenShift

> 原文：<https://developers.redhat.com/blog/2020/12/22/net-5-0-now-available-for-red-hat-enterprise-linux-and-red-hat-openshift>

我们很高兴地宣布正式推出。NET 5.0 在[红帽企业 Linux](https://developers.redhat.com/products/rhel/overview) 7、红帽企业 Linux 8、[红帽 OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)上。

## 怎么样

。NET 5.0 是。NET Core 3.1，而[取而代之。NET Framework](https://devblogs.microsoft.com/dotnet/net-core-is-the-future-of-net/) 作为构建 Windows 窗体和 WPF 应用程序的首选目标平台。

。NET 5.0 包括新的语言版本 C# 9 和 F# 5.0。对基本库、GC 和 JIT 进行了显著的性能改进。ASP.NET 核心框架有许多很棒的新特性和改进，比如改进的 gRPC 和 Blazor 性能，以及默认启用 OpenAPI。

## 安装。NET 5.0

。NET 5.0 可以安装在 RHEL 7 与通常的:

```
# yum install rh-dotnet50

```

在 RHEL 8 上，输入:

```
# dnf install dotnet-sdk-5.0

```

的。NET 5.0 SDK 和运行时容器映像可从 [Red Hat 容器注册表](https://catalog.redhat.com/software/containers/search?q=dotnet-50)获得。您可以将容器图像用作独立图像，并与 OpenShift 一起使用:

```
$ podman run --rm registry.redhat.io/ubi8/dotnet-50 dotnet --version
5.0.100

```

## 支持

。NET 5.0 是最新版本。它计划支持到 2022 年 1 月，即。NET 6.0 年 11 月发布。。NET 6.0 将是一个长期支持(LTS)版本，支持三年。以前的 LTS 版本。NET Core 2.1 和。NET Core 3.1，分别支持到 2021 年 8 月 21 日和 2022 年 12 月 3 日。

参观[。NET 概述页面](http://redhatloves.net)了解更多关于使用。NET 在红帽企业版 Linux 和 OpenShift 上的应用。

*Last updated: December 21, 2020*