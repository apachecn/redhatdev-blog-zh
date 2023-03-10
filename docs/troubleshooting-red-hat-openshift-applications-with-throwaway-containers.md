# 使用一次性容器对 Red Hat OpenShift 应用程序进行故障排除

> 原文：<https://developers.redhat.com/blog/2019/08/22/troubleshooting-red-hat-openshift-applications-with-throwaway-containers>

想象一下这个场景:你的酷酷的微服务在你的本地机器上运行良好，但是在部署到你的 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 集群上时就失败了。您看不到代码中的任何错误，也看不到服务、配置图、机密和其他资源中的任何错误。但是，你知道有些事情不对劲。你如何从与你的容器化应用相同的角度看待事物？如何比较本地应用程序和容器中的运行时环境？

如果你进行了尽职调查，你就编写了单元测试。没有关于运行时环境的硬编码配置或隐藏的假设。原因应该与您的应用程序在 OpenShift 中接收到的配置有关。是时候在一步一步的调试器下运行你的应用程序，还是在你的代码中添加大量的日志语句了？

我们将展示 OpenShift 命令行客户端的两个特性如何提供帮助:命令`oc run`和`oc debug`。

## 在 Red Hat OpenShift 上启动一次性容器

大多数开发人员认为容器和 OpenShift 只是为了运行长期应用程序。您可以创建部署配置、有状态集或 cron 作业，这些作业将永远保持活动状态，并根据需要创建和重新创建 pod。您的应用程序总是开启的，或者至少在固定的时间间隔内开启。

`oc run`命令运行执行单一任务然后死亡的容器。它创建了非托管容器，OpenShift 不会在这些容器死亡时替换它们。

我曾经有一个应用程序与我的 OpenShift 集群之外的遗留数据库进行对话。应用程序能够从我的本地机器访问数据库，但不能从 OpenShift 访问。我需要测试从 OpenShift 内部访问数据库的能力。这样我就可以知道我是否得到了正确的环境变量。我还会查看容器是否能够解析数据库的主机名。也许有防火墙阻止了对数据库的访问？

对于`oc run`命令来说，这是一个完美的场景。只需启动一个运行数据库容器映像的 pod。在该窗格中，您可以使用数据库客户端和操作系统命令来排除配置网络连接故障。几个快速测试之后，你就不再需要豆荚了。

```
$ oc run -it test --rm --restart Never \
--image registry.access.redhat.com/rhscl/mysql-57-rhel7 bash

```

前面的命令在一个名为 test 的 pod 上给出了一个交互式(`-it` ) Bash 提示符。OpenShift 从不重启这个 pod ( `--restart Never`)并在终止(`--rm`)时移除它。

Red Hat 的 mysql 数据库映像(rhscl/mysql-57-rhel7)提供了 MySQL 客户端和一些其他有用的命令，如 dig 和 host。这样，我可以检查我是否能够解析服务器主机名，连接到数据库，并验证我的访问凭证。

## 为管理客户机启动一次性容器

我可以直接从`oc run`命令启动 MySQL 客户端，或者该容器映像中可用的任何其他命令。例如:

```
$ oc run -it test --rm --restart Never \
--image registry.access.redhat.com/rhscl/mysql-57-rhel7 \
-- mysql -umydbuser -pmydbpassword -hmyserver.domain.example.com mydb
```

注意使用双破折号(`--`)来防止`oc run`命令解释 MySQL 客户端的命令选项。在前面的命令中，没有`--mysql`选项；`--`和`mysql`之间有一个空格。

作为另一个例子，我可以从第一个例子中启动同一个一次性容器，然后使用另一个终端通过`oc cp`命令将 SQL 脚本复制到容器中。然后，我可以使用一次性容器外壳运行 SQL 脚本。

因为 MySQL 客户端可以从标准输入中获取 SQL 脚本，所以我只需在第二个示例中添加输入重定向就可以了。我刚刚填充了一个测试数据库。在我不编写为应用程序部署和初始化数据库的奇特操作符的情况下，从 shell 脚本或 Ansible playbook 来做这件事怎么样？

多亏了`oc run`命令，我可以使用嵌入到许多容器映像中的管理客户端，例如，JBoss EAP、AMQ 和单点登录的 CLI 管理工具。我不需要在我的本地机器上安装它们。很酷，不是吗？

## 将部署克隆到调试容器

我可以向`oc run`命令添加更多的命令行选项，并复制现有部署的所有设置:环境变量、资源限制等等。如果我的意图是复制我的应用程序的运行时环境，这将是太多的工作，并且容易出错。

然而，这将是`oc debug`命令的一个场景。它创建了一个新的 pod，它是现有 pod 的副本。如果您的 pod 由于某种原因没有启动，您可以从其部署配置中创建副本。

假设我使用`oc new-app`创建了我的应用程序，并将其命名为`myapp`。为了从它的部署配置中创建一个 debug pod，我将使用下面的命令:

```
$ oc debug -t dc/myapp
```

我得到了一个在与应用程序相同的约束下运行的 Bash shell、SElinux 上下文、环境变量和相同的容器映像。

如果我怀疑这些约束可能导致故障，我可以使用`oc debug`命令中的选项有选择地忽略它们。例如，将`--as-root`选项添加到前面的示例中，会在 pod 中给我一个根提示，但前提是我的 OpenShift 用户有权访问允许我这样做的安全上下文约束。

调试窗格在禁用健康探测器的情况下运行。我可以手动启动我的应用程序来检查健康探测是否不正确，并强制我的 pod 终止。我可以在`oc debug`命令中添加选项来启用健康探测器、禁用 init 容器、禁用 sidecar 容器、更改影响 pod 调度的标签，从而发现哪些部署设置(如果有的话)对我的应用程序不正确。

## 使用 RHEL 工具容器图像启动一次性容器

与`oc run`命令一样，使用`oc debug`命令的操作受到应用程序容器映像所包含内容的限制。幸运的是，您可以在调试容器中覆盖容器映像。Red Hat 的 rhel7/tools 和 rhel8/support-tools 容器映像是不错的选择。

```
$ oc debug -t dc/myapp \
--image registry.access.redhat.com/rhel7/rhel-tools
```

这些映像提供对大多数应用程序映像中不包含的标准 RHEL 故障排除命令的访问，例如`ping`和`dig`命令。

您需要从基于 Red Hat terms 的注册表(redhat.registry.io)下载 rhel8/support-tools 容器映像。访问基于术语的注册表需要 pull secret。如果需要，请遵循来自 [Red Hat Enterprise Linux 支持工具](https://access.redhat.com/containers/?tab=images&get-method=registry-tokens#/registry.access.redhat.com/rhel8/support-tools)的说明。

## 结论

您不需要本地容器引擎来运行执行故障排除和一次性任务的一次性容器。您可以使用`oc run`和`oc debug`命令在 Red Hat OpenShift 上快速轻松地运行这些容器。此外，如果您的 OpenShift 集群不是一个 Minishift 实例，那么它下载容器映像的速度可能会更快，并且可能比您的本地工作站有更多的存储空间和更好的带宽。

*Last updated: September 3, 2019*