# Command-line tools for Kubernetes: kubectl, stern, kubectx, kubens

> 原文：<https://developers.redhat.com/blog/2019/05/27/command-line-tools-for-kubernetes-kubectl-stern-kubectx-kubens>

如果你曾经用手工作过，你就会知道，没有合适的工具，你就不能把工作做好。这句格言也适用于软件开发。不管底层技术是什么，正确的工具都可以决定成败。在 Kubernetes 生态系统中，随着人们找到解决共同问题的方法，越来越多的工具被引入。这篇文章看起来就是其中的四个工具。

## 库贝特尔

Kubernetes 的标准命令行工具，您可以执行 Kubernetes 所需的所有操作。这是任何 Kubernetes 政府的起点。kubectl 是 Kubernetes 的命令行工具，由 T2 云计算基金会管理，该基金会为我们带来了 Kubernetes。

### 在 MacOS 上安装

```
brew install kubectl
```

### 在 Windows 上安装

```
choco install kubernetes-cli
```

说真的，如果你的 Windows 机器没有使用 [Chocolatey](https://chocolatey.org/) ，停止一切，现在就安装。我喜欢玩的一个游戏就是试着安装一些软件，甚至不用去查。我只需在命令行输入，比如说,`choco install notepad++`,然后看看会发生什么。巧克力几乎和真正的巧克力一样好。差不多了。

## 严厉的

与库伯内特航海主题保持一致，船尾是船的尾部...以及显示容器和多个容器的日志尾端的工具。真正的开源社区精神，[斯特恩项目](https://github.com/wercker/stern/tree/master/stern)来自 wer cker(2017 年被甲骨文收购)。

Stern 是一个礼物，因为您可以使用 stern 来观看日志展开，而不是查看整个日志来查看最近发生的事情。

### 在 MacOS 上安装

```
brew install stern
```

### 在 Windows 上安装

安装程序可以从[斯特恩发布页面](https://github.com/wercker/stern/releases)下载。

## 库贝特尔

Kubectx 对于多集群安装很有帮助，在这种情况下，您需要在一个集群和另一个集群之间切换上下文。与其键入一系列冗长的 kubectx 命令， [kubectx](https://github.com/ahmetb/kubectx) 在一个简短的命令中发挥了它的魔力。它还允许您将一个很长的集群名作为别名。例如(直接取自 kubectx 网站)，`kubectx eu=gke_ahmetb-samples-playground_europe-west1-b_dublin`允许您通过运行`kubectx eu`切换到那个集群。另一个巧妙的技巧是 kubectx 会记住你之前的上下文——很像电视遥控器上的“上一页”按钮——并允许你通过运行`kubectx -`切换回来。

kubectx 博客是[这里](https://ahmet.im/blog/kubectx/)。感谢 Google Ahmet Alp Balkan—[Ahmet b on GitHub](https://github.com/ahmetb)—提供的工具。回购在这里是。

### 在 MacOS 上安装

```
brew install kubectx
```

### 在 Windows 上安装

因为是 Bash shell 脚本，所以没有针对 Windows 的 kubectx。**不要绝望**；GitHub 用户[thomasliddledba](https://github.com/thomasliddledba)(Thomas Liddle)已经创建了 [kubectxwin](https://github.com/thomasliddledba/kubectxwin) ，它可以与 Windows 一起工作。下载和构建说明在 [GitHub repo 自述文件页面](https://github.com/thomasliddledba/kubectxwin)上。

**提示:**使用 PowerShell `Set-Alias kubectx kubectxwin`

## 库本斯

这个脚本允许您轻松地在 Kubernetes 名称空间之间切换。一个简单的`kubens foo`将使`foo`成为您的活动名称空间。这个工具的简单性是它如此吸引人的原因。它做一件事，而且做得很好。像 kubectx 一样，破折号(“-”)选项会将您返回到以前的值。

库本斯再次向艾哈迈德·阿尔普·巴尔坎致敬。

### 在 MacOS 上安装

当你运行`brew install kubectx`时，你会得到 kubens。价值如何？

### 在 Windows 上安装

托马斯·利德尔又一次和库本斯温一起度过了难关。下载和构建说明可在 [GitHub repo README 页面](https://github.com/thomasliddledba/kubenswin)上获得。

## 开始你的手工游戏

将这四个工具加载到您的开发机器上，您就可以在轻松浏览 Kubernetes 水域的同时构建一些令人惊叹的应用程序了。再一次，开源社区为寻求进步和发展的开发者提供了机会。

*Last updated: September 3, 2019*