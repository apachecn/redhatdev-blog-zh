# Kubernetes 的 4 个命令行工具:Linux 版

> 原文：<https://developers.redhat.com/blog/2019/08/08/4-command-line-tools-for-kubernetes-linux-edition>

在之前的博文中，我详细介绍了如何在你的 macOS 或 Windows 机器上安装四个非常有用的 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 工具。这些工具——kubectl、stern、kubectx 和 kubens——是高级开发人员以及任何运营人员的必备工具。然而，我之前没有做的是包含在 Linux 上安装这些工具的说明。因此...我们到了。

## 库贝特尔

使用 Kubernetes 的标准命令行工具 kubectl，您可以执行 Kubernetes 所需的所有操作。这是任何 Kubernetes 政府的起点。kubectl 是 Kubernetes 的命令行工具，由 T2 云计算基金会管理，该基金会为我们带来了 Kubernetes。

### 在 Linux 上安装

在 Linux 上安装 kubectl 有三种路径可供选择:

1.  下载并安装二进制文件。
2.  使用与您的特定 Linux 发行版相关联的包管理器(例如 yum、apt-get)。
3.  使用[抓手。](https://snapcraft.io/docs)

所有这三个路径在 kubectl 安装页面上都有详细介绍；我不会在这里重复它们，因为它们已经被很好地记录了。

## 严厉的

为了与 Kubernetes 航海主题保持一致，stern 是一艘船的尾部，它是一个显示集装箱和多个舱的日志尾部的工具。真正的开源社区精神，[斯特恩项目](https://github.com/wercker/stern/tree/master/stern)来自 wer cker(2017 年被甲骨文收购)。

Stern 是一个礼物，因为您可以使用 stern 来观看日志展开，而不是查看整个日志来查看最近发生的事情。

### 在 Linux 上安装

跳转到 stern 的 Github repo 的二进制文件页面，下载最新的二进制文件，并将其存储在您机器上的一个目录中。确保您的路径包含所述目录。这就是全部了。

## kubectx

Kubectx 对于多集群安装很有帮助，在这种情况下，您需要在一个集群和另一个集群之间切换上下文。与其键入一系列冗长的 kubectl 命令， [kubectx](https://github.com/ahmetb/kubectx) 在一个简短的命令中发挥了它的魔力。它还允许您将一个很长的集群名作为别名。例如(直接取自 kubectx 网站)，`kubectx eu=gke_ahmetb-samples-playground_europe-west1-b_dublin`允许您通过运行`kubectx eu`切换到那个集群。另一个巧妙的技巧是 kubectx 会记住你之前的上下文——很像电视遥控器上的“最后”按钮——并允许你通过运行`kubectx -`切换回来。

kubectx 博客是[这里](https://ahmet.im/blog/kubectx/)。感谢 Google Ahmet Alp Balkan—[Ahmet b on GitHub](https://github.com/ahmetb)—提供的工具。回购在这里是。

### 在 Linux 上安装

kubectx 和 kubens(本文后面会提到)都是用 Bash 编写的，所以它们与任何 Linux 发行版都非常兼容。下载它们的说明——以及其他一些优点，比如 tab 补全——在 kubectx 的 Github repo 中有详细介绍。

## 库本斯

这个脚本允许您轻松地在 Kubernetes 名称空间之间切换。一个简单的`kubens foo`将使`foo`成为您的活动名称空间。这个工具的简单性是它如此吸引人的原因。它做一件事，而且做得很好。与 kubectx 一样，破折号(-)选项会将您返回到以前的值。

库本斯再次向艾哈迈德·阿尔普·巴尔坎致敬。

### 在 Linux 上安装

你猜怎么着:安装 kubectx 的时候就包含了。价值如何？

## 开始你的手工游戏

将这四个工具加载到您的开发机器上，您就可以在轻松浏览 Kubernetes 水域的同时构建一些令人惊叹的应用程序了。再一次，开源社区为寻求进步和发展的开发者提供了机会。

*Last updated: August 7, 2019*