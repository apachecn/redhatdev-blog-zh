# 使用 Delve 调试 Red Hat Enterprise Linux 上的 Go 程序

> 原文：<https://developers.redhat.com/blog/2021/03/03/using-delve-to-debug-go-programs-on-red-hat-enterprise-linux>

Delve 现在可以在[红帽企业 Linux](https://developers.redhat.com/topics/linux) (RHEL)上使用。从 RHEL 8.2 和`devtools-2020.2`版本开始， [Go 语言](https://developers.redhat.com/blog/category/go/)调试器 [Delve](https://github.com/go-delve/delve) 将通过`go-toolset`包与 Go 工具链一起安装。

Delve 是专门为 Go 定制的，拥有 Go 运行时的复杂知识，并提供了其他调试器中没有的功能和环境。该工具旨在简化使用，在您发现程序有什么问题时不要碍手碍脚。Delve 还提供了强大的功能，让你尽可能快地调试你的围棋程序。

## 装置

安装只需要一个命令来安装 Go 和 Delve:

```
sudo dnf install -y go-toolset
```

如果命令成功，Go 工具链和 Delve 调试器都将被安装并准备好使用。

## Delve 的优势

正如我前面提到的，这个工具的目标是尽可能地不碍事。这意味着消除许多手动步骤，以提供直观的调试体验。

例如，只要可行，Go 编译器就试图选择合理的缺省值，所以它默认打开优化。这些优化对于在生产中运行你的 Go 程序来说是非常好的，但是它们会使调试变得更加困难。Delve 通过在构建二进制文件时自动关闭优化来解决这个问题。Delve 还可以和其他工具一起工作，比如 Mozilla RR T1，它目前还不能作为一个包在 RHEL 上使用，但是可以从源代码中构建或者从上游版本中安装。

## 使用 Delve

Delve 的目标是像 Go 命令本身一样简单易用。开始调试包的基本用例是:

```
dlv debug
```

发出这个命令会导致 Delve 在自动禁用优化的情况下编译您的程序，并让您进入准备开始调试的命令提示符。

用 Delve 开始调试 Go 程序的其他最常见的方法是连接到一个正在运行的进程或执行一个预先构建的二进制文件:

```
dlv attach $pid
dlv exec ./path/to/binary
```

如果二进制文件是通过优化构建的，那么这些选项的唯一潜在缺点就会出现。在这种情况下，调试器可能无法获得某些信息。因此，如果您想在预编译的二进制文件上调用 Delve，我们建议在禁用优化的情况下构建它，使用以下标志:

```
-go build -gcflags="-N -l"
```

## 钻研技巧和诀窍

让我们来看看一些额外的技巧和窍门，帮助你充分利用 Delve。

### Delve 可以调用被调试进程中的函数

例如，要跳转到 Delve 中名为`main.callme`的函数，请输入:

```
(dlv) call main.callme
```

### Delve 可以像 strace 一样工作

您可以连接并跟踪正在运行的二进制文件，而不是启动调试会话。例如，您可以如下跟踪名为`callme`的函数:

```
dlv trace callme
> goroutine(1): main.callme(12) => (16)

```

调用函数时，Delve 的输出显示参数和返回值——所有这些都不需要启动交互式调试会话！

### Delve 允许不同的后端

例如，您可以切换到 Mozilla RR 后端进行记录和重放调试，如下所示:

```
dlv debug --backend=rr
```

要了解更多关于这种技术的信息，请观看我的演讲，[用 Delve](https://video.fosdem.org/2020/UB2.252A/debuggingwithdelve.webm) 进行确定性调试。

### Delve 允许您编写调试器脚本并添加您自己的命令

为了给用户提供自动化 Delve 或添加新命令的 API，Delve 使用了脚本语言 [Starlark](https://docs.bazel.build/versions/master/skylark/language.html) ，它类似于 Python，由创建 [Bazel 构建工具](https://docs.bazel.build/versions/master/bazel-overview.html)的项目开发。参见 [Delve 的 API 文档](https://github.com/go-delve/delve/blob/master/Documentation/cli/starlark.md)了解更多关于这个工具以及如何使用它的信息。

## 结论

现在 Delve 可以在 RHEL 的`go-toolset`包中获得，调试 Go 程序变得前所未有的简单。尝试一下，并在[上游回购](https://github.com/go-delve/delve/)中给出您的反馈。