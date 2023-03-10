# 快速介绍。网络核心 2.0

> 原文：<https://developers.redhat.com/blog/2017/08/22/quick-introduction-of-net-core-2-0>

如果你已经在 IT 行业工作了几年以上，你可能听说过这样一句话“等到第三次发布”再开始新的技术或产品。嗯，。NET Core 有 1.0 版和 1.1 版。这是第三个版本:介绍。网芯 2.0。相信我，现在是时候跟上这股潮流了。

## 你得到一个 API，每个人都得到一个 API

有什么大不了的。网芯 2.0？首先，大约 20，000 多个 API，从大约 13，000 个增加到 32，000 个。这 32，000 代表了 NuGet.org 项目所使用的 API 的 70%的覆盖率。这对你意味着什么？这意味着有一个更好的机会。NET 框架类库将与。网芯 2.0。

想知道吗？微软有一个简单易用的工具， [APIPort](https://github.com/Microsoft/dotnet-apiport) ，你可以用它来检查你现有的代码。为了给你一个这样的例子，我选择了一个相当大的库来使用 APIPort 进行检查:[Rackspace。NET SDK for OpenStack](https://rackspace.github.io/rackspace-net-sdk/) 。结果呢？它获得了 100%的兼容评级。

![](img/f1a6da2d6921f9d9c5a37f2fa4328be7.png)

这是事情之一。网芯 2.0 带来的表。

## 我需要速度

想要性能？。网芯 2.0*最快。网曾经*。你可以在这里查看一些测试结果[，在这里](https://blogs.msdn.microsoft.com/dotnet/2017/06/07/performance-improvements-in-net-core/)查看[，在这里](https://blogs.msdn.microsoft.com/dotnet/2017/06/29/performance-improvements-in-ryujit-in-net-core-and-net-framework/)查看[。简单地说，当谈到性能改进时，业务正在蓬勃发展。](https://blogs.msdn.microsoft.com/dotnet/2017/07/20/profile-guided-optimization-in-net-core-2-0/)

## 标准和。网无处不在

。NET Core 2.0 支持。NET 标准 2.0。。NET 标准不是 API 更确切地说，这是一种规范。很像 HTML5 是一种规范，浏览器也是为了支持它而编写的。NET 标准是规范。。NET Core 2.0 兼容。净标准。

那是什么意思？这个怎么样:把你的库写到？NET 标准 2.0 规范，它们可以在任何地方运行:。网络核心，单声道。NET 框架。是的，*真正可移植的代码*可跨支持的所有设备和操作系统。网族。

你也会立即注意到一些变化。

## 节省一些击键

第一次运行`dotnet new`时，您会注意到`dotnet restore`是自动运行的。从……开始。NET Core 2.0，`dotnet restore`是一个隐式命令。也就是说，它会在必要时自动运行。可以通过使用`--disable-restore`标志将其禁用。

## Visual Basic 和微服务？

现在支持 Visual Basic。考虑到许多企业在 VB 代码上有大量投资，这对于来说是一个巨大的胜利。网芯 2.0。不管你是不是 VB 的粉丝，跟我一起说:“用 Visual Basic 写的微服务”。是的，*来了*。

## 更少的代码意味着更多

你的“项目”文件现在更小，更容易理解，尤其是对于 ASP.NET MVC 应用程序。例如，引用一个 MVC 应用程序的所有必要的位只需要*一行额外的代码*你的项目文件(提示:启动。NET Core 2.0，输入`dotnet new mvc`，看*号。csproj 文件)。

## 今天就买

这只是对所有美好事物的一瞥。网芯 2.0。现在是时候将它安装到您的 Red Hat Enterprise Linux (RHEL)机器上并开始编码了。毕竟，这是第三个版本(1.0，1.1，2.0)，第三次是一种魅力。

什么？你的 Windows PC 或 Mac 上没有运行 RHEL 虚拟机吗？跳转到 Red Hat Developers 网站，立即获得一份零成本的 Red Hat Enterprise Linux。你将编码。几分钟之内就能上 RHEL 的网。

你有 RHEL 却没有。安装了网芯 2.0？你可以在这里找到安装说明。

*Last updated: September 3, 2019*