# 为开发人员部署 debuginfod 服务器

> 原文：<https://developers.redhat.com/blog/2019/12/17/deploying-debuginfod-servers-for-your-developers>

在早先的一篇文章中，Aaron Merey 介绍了新的 elfutils `debuginfo-server`守护进程。现在这个软件已经集成并发布到 elfutils 0.178 中，并且即将发布到您身边的发行版中，是时候考虑为什么以及如何为您自己和您的团队建立这样一个服务了。

回想一下,`debuginfod`的存在是为了发布 ELF 或 DWARF 调试信息，以及相关的源代码，用于收集二进制文件。如果您需要运行像`gdb`这样的调试器，像`perf`或`systemtap`这样的跟踪或探测工具，像`binutils`或`pahole`这样的二进制分析工具，或者像`dyninst`这样的二进制重写库，您最终将需要与您的二进制文件相匹配的`debuginfo`。这些工具中的`debuginfod`客户端支持实现了一种快速、透明的方式来动态获取这些数据，而不必停下来，切换到 root，运行所有正确的`yum debuginfo-install`命令，然后重试。Debuginfo 允许您随时随地进行调试。

我们希望这个开场白能解答“为什么”现在，让我们来看看“怎么做”

## 基本服务器操作

为了让客户端能够下载内容，您需要一个或多个`debuginfod`服务器，每个服务器都可以访问所有可能需要的`debuginfo`。理想情况下，您应该在尽可能靠近保存那些构建工件副本的机器的地方运行`debuginfod`服务器。

如果您构建自己的软件，那么它的构建和源代码树在同一个位置。要在您的构建机器上运行`debuginfod`的副本:

```
$ debuginfod -F /path/to/build/tree1 /path/to/build/tree2

```

然后，`debuginfod`将定期重新扫描所有这些树，并在那里提供所有可执行文件和调试数据，以及从那里引用的源文件。如果您重新构建代码，索引将很快跟上(参见`-t`参数)。

如果您一直将自己的软件构建到 RPM 中，那么运行一个包含 RPM 文件的父目录的`debuginfod`副本:

```
$ debuginfod -R /path/to/rpm/tree1 /path/to/rpm/tree2

```

然后，`debuginfod`将定期重新扫描所有这些树，并使 RPMs 中的所有可执行文件和调试文件可用。该工具自动匹配`-debuginfo`和`-debugsource`文件。

自然地，你可以用一个`debuginfod`过程来完成这两个任务:只需要把这些参数加在一起。

如果您需要调试作为 Linux 发行版一部分的软件，您会有一点困惑。在发行版建立公共服务器之前，我们必须自己保护自己。幸运的是，做到这一点并不太难。毕竟，您只需要一台已经安装了发行版相关包的机器——甚至只是下载了:

```
$ mkdir distro-rpms ; cd distro-rpms
$ debuginfod -R .

```

并根据需要重复:

```
$ yumdownloader PACKAGE-N-V-R
$ yumdownloader --debuginfo PACKAGE-N-V-R  

```

使用您的磁盘允许的所有通配符和保留。

如果你在内部运行一个[红帽卫星](https://www.redhat.com/en/technologies/management/satellite)服务器，或者一个非正式管理的发行版包镜像，你可以对那些系统的包档案运行`debuginfod`。没有必要安装(`rpm -i`)、过滤或者以任何人为的方式重新组织它们。只是让一个`debuginfod`的副本扫描目录。

## 客户端配置

好了，你现在有一个或多个服务器在运行，它们可以扫描相同或不同的`debuginfo`材料树。我们如何让客户和他们交谈？简单明了的解决方案是列举所有已知的服务器:

```
$ export DEBUGINFOD_URLS="http://host1:8002/ http://host2:8002/ ...."                                   
$ gdb ... etc.                         

```

对于每一次查找，客户机将立即向所有服务器发送一个查询，第一个返回所请求信息的服务器将“获胜”

虽然这种策略有效，但也有一些缺点。首先，必须将这个 URL 列表传播给每个客户端。第二，没有机会集中缓存内容，因此每个客户端必须从源服务器单独下载内容(用 HTTP 术语来说)。有一个简单的解决办法:联盟。

每个`debuginfod`服务器也可以充当客户端。如果服务器不能回答来自其本地索引的查询，并且已经配置了一个上游`$DEBUGINFO_URLS`列表，那么它将把请求转发给上游服务器。然后，它将缓存一个肯定响应的结果，并将其转发回去。相反，对同一对象的下一个请求将由缓存提供服务(受缓存保留限制)。

这种行为允许您配置一个由`debuginfod`服务器组成的联合层次结构。这样做允许集中配置文件和本地化缓存。然后，您的每一个构建系统`debuginfods`都可以配置一个更高级别的对等列表。你甚至可以拥有`debuginfod`服务器，它根本不扫描任何本地目录，而是纯粹作为上游中继。确保联盟是一棵树或有向图。周期是不好的。

## 服务器管理

现在，您有一个或多个服务器正在运行，客户端依赖于它们。保持他们运行良好呢？有几个实际问题需要担心。

一个是索引期间和之后的资源使用。初始`debuginfod`索引在 CPU 和存储上非常密集。它必须立即对 RPMs 进行流解压缩，并解析每个 ELF 或 DWARF 文件。索引数据库是一个紧密格式化的 SQLite 文件，但是它可以增长到普通压缩 rpm 大小的 1%左右。如果这方面不是问题，那么这下一段就不用担心了。

如果为非常大的一组归档文件建立索引的时间和空间过多，那么使用文件过滤器运行`debuginfod`会很有帮助。它的`-I`和`-X`选项允许您为文件名指定正则表达式，它应该包括或排除。比方说，您的归档有多个不同的混合架构或不同的文件主发行版版本，而您只想跟踪其中的一个子集。您可以使用这些选项来强制`debuginfod`跳过名称与模式不匹配的文件:

```
$ debuginfod -I '\.el[78]\.x86_64' -X 'python' -R /path 

```

如果您的服务器有很多内核，可以考虑将扫描路径分成许多子路径，因为`debuginfod`会为命令行上给出的每个路径启动一个或两个线程。实际的并发被小心地管理，所以当给出大的路径列表时你可以无忧无虑。因此，使用:

```
$ debuginfod -R /path/*
```

代替

```
$ debuginfod -R /path
```

如果您的 ELF、DWARF 或 RPM 归档非常大，您可以考虑在存储服务器附近运行的多个`debuginfod,`副本之间分割扫描任务。您可以使用通配符加上包含和排除路径来给每个`debuginfod`进程一个数据子集。我们在上面讨论了`debuginfod`服务器如何联合。使用该工具创建一个不扫描任何东西的前端`debuginfod`，但是将查询委托给整个稳定的碎片。

在 shell 中手工运行网络服务器是一种不错的老式方法。然而，对于严肃的部署，您会希望您的`debuginfod`服务器由一个监控系统来管理。因为`debuginfod`在一个简单的 shell 中运行得如此之好，它就像一个`systemd`服务或在一个容器中运行得一样好。elfutils 附带了一个示例`systemd`配置，我们计划发布 docker 文件或容器映像，您可以使用它们在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 或另一个编排服务中运行 debuginfod。

一旦服务器开始运行，最好对其进行监控以保持其运行。文本日志进入标准输出和错误流，像`systemd` journal 或 OpenShift 这样的工具可以收集文本。添加更多的`-v`详细选项来生成更详细的跟踪。除了这些文本数据，`debuginfod`还提供了一个`/metrics` web API URL，这是一个 Prometheus export 格式的定量数据源。这个 URL 提供了关于服务器线程的内部统计信息。连接警报系统或其他程序来检测各种类型的异常并不困难。

一旦跨越信任边界提供服务，例如在互联网上向公众提供服务，安全性就成为一个问题。手册页对这种服务对用户和服务操作员来说是安全的所需措施提供了过多的警告。这不是火箭科学，但 TLS 加密和负载控制等普通 HTTP 前端保护是必须的，例如使用 HAProxy 安装。将`debuginfod`索引限制在可信的(非恶意的)二进制文件也很重要。

### 展望未来

未来会怎样？我们希望很快支持 Debian 格式的软件包，这样我们在那个生态系统中的朋友也可以充分利用。我们很乐意帮助 Linux 发行版为他们的用户运行公共服务，并且已经在 [Fedora koji](https://koji.fedoraproject.org/koji/) 中建立了这个服务的原型。我们还设想了更多的可管理性特性，以及与源代码版本控制系统的集成。我们也欢迎我们的早期采用者——您——提出建议！

我们希望这篇文章有助于激励你，帮助你建立自己的`debuginfod`服务。请通过[elfutils-devel@sourceware.org](mailto:elfutils-devel@sourceware.org)邮件列表联系我们。

*Last updated: January 25, 2021*