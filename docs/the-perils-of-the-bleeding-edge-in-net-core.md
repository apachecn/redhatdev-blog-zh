# 流血边缘的危险。网络核心

> 原文：<https://developers.redhat.com/blog/2017/08/01/the-perils-of-the-bleeding-edge-in-net-core>

让我们面对现实吧:作为开发人员，我们中的许多人都喜欢站在技术的前沿，或者更好地说，站在技术的前沿。无论是因为学习新东西很有趣，还是为了在本地用户群中吹嘘自己，或者是因为我们希望保持我们的“职业之剑”锋利，前沿保证会给我们的日子带来兴奋。肯定比维护 VB6 代码强。

但是伴随这种兴奋而来的是“出血边缘”这个术语的由来；千刀万剐。

我仍然感到最近被割伤的刺痛。

## 目标

我决定写一篇关于使用 Visual Studio 2017 (VS 2017)创建一个应用程序的博文(并制作一个视频)，然后在 [Red Hat OpenShift](https://www.openshift.com/) 中运行它。我想演示如何在 Visual Studio 中运行代码，同时在 OpenShift 中的 Linux 容器中运行代码，实现代码的零停机滚动更新。

## 安装

我做的第一件事是在 GitHub 中创建一个库来存放我的代码。这将允许 OpenShift 从 GitHub 中提取代码，并自动构建和部署它。我所要做的就是将代码从 Visual Studio 推送到 repo，OpenShift 将由 webhook 触发开始构建。

在创建了一个空的 repo(包含 README 和许可证文件)之后，我将它克隆到我的 Windows 机器和 Red Hat 虚拟机之间的一个共享目录中。目录名为“shared”，所以我将其克隆到“/shared/locationms”中。

## 阻塞物

然后我意识到我正在使用。NET Core 2.0 在我的 Red Hat Enterprise Linux (RHEL)虚拟机(VM)上预览 2 位，因为它不是普遍可用的，所以 OpenShift.com 不会支持。网芯 2.0 还没。没问题；我将简单地在 Visual Studio 中编写代码，然后在我的 RHEL 虚拟机的命令行中运行它。这至少会让我*更接近*我想要的体验:在 Visual Studio 中工作，在 OpenShift 中运行。

当我意识到 VS 2017 只支持. NET Core 2.0 web API 应用程序时，我启动了我的 IDE 并开始创建它。网芯 1.0 和 1.1。2.0 版没有选项。

一些网络搜索发现了对 Visual Studio 预览版的早期访问。谈论出血边缘...

## 在 Windows 上运行

我安装了 VS 2017 预览版 15.3，启动了。就在那里。用于创建 web API 的 NET Core 2.0 选项。因为我已经有了。NET Core 2 Preview 2 安装在我的 RHEL 虚拟机上，这将很容易:我会在 VS 2017 中创建新项目，将其保存到我在 Windows 机器和虚拟机之间共享的目录中，然后切换到虚拟机来运行它。我创建了 web API 应用程序，并从 IDE 中运行它；没问题。

## 在 RHEL 奔跑

我关闭了 Visual Studio 并热键切换到我的虚拟机。导航到/shared/locationms 目录时，我看到了一些意想不到的东西。提醒你，没什么不好，只是出乎意料。我的应用程序的实际代码不在/shared/locationms；它在/shared/locationms/locationms 中。第二个目录是 Visual Studio 默认行为的结果。注意到了。

这是事情走下坡路的时候。我运行`dotnet restore`来确保依赖关系是好的，但它们并不好。我得到了以下错误:

奇怪。我知道我的两个系统不是完全相同的版本，但我认为它们足够接近:我的 Windows 系统是 2.0.0-preview2-006900，我的 RHEL 虚拟机是 2.0.0-preview2-006497。

## 狐狸

所以这是一种脱节。Visual Studio 使用的是较新版本的。网芯。虽然这不是一个完美的解决方案，但我编辑了 locationms.csproj 文件，更改了下面两行。在这两种情况下，我都把版本改成了“2.0.0-*”。

![](img/a2371b4902d3b85aa2452fa984ff6611.png)

之后，我又跑了一次`dotnet restore`，这次成功了。使用`dotnet run`在我的 RHEL 虚拟机上取得了成功:

![](img/8f42dba6fa3baedfcd14e0ff9b9e1b27.png)

## 交易场所

当我回到 Windows 中的 Visual Studio 时，现在它*没有*运行。我必须恢复到原始的 locationms.csproj 文件才能使它工作。

## 外卖

当您使用预览代码或测试版时，有时会出现问题。请放心，这些都将在最终版本中工作。在此之前，请注意动态开发工作可能会带来挑战。和微小的伤口。

*Last updated: July 31, 2017*