# 红帽峰会:构建生产就绪容器

> 原文：<https://developers.redhat.com/blog/2018/05/31/red-hat-summit-building-production-ready-containers>

在展会的最后一天，Scott McCarty 和 Ben Breard 讨论了生产就绪型容器的最佳实践，为今年的红帽峰会画上了圆满的句号。

在容器时代，Scott 指出有四个构建模块需要考虑:

*   容器图像
*   容器主机
*   容器编排
*   注册服务器

这些话题中的每一个都是一个巨大的兔子洞，如果你想真正了解所有关于它们的知识，你可以进入这个洞。正如您从会议标题中所料，Scott 和 Ben 主要关注容器图像。尽管如此，你还是要考虑其他三个。你的图像将如何与他人互动？你如何向他们提供数据？他们之间会如何互动？你会在图片中嵌入密码吗？(剧透警告:*号*)当你进入容器的世界时，你需要考虑所有这些事情。

一个单独的容器和一个单独的乐高积木 [¹](#link1) 一样有用。您需要以有趣的方式将它们结合在一起，以获得容器的全部功能。斯科特引用了红帽公司的瑞安·哈利西的话:

> 使用容器既是商业优势，也是技术优势。在构建和使用容器时，分层至关重要。你需要看着你的应用程序，思考每一个部分以及它们是如何协同工作的——就像你把一个程序分成类和函数一样。

如果你正在建造一个乐高模型，你需要一张说明书和一组积木，然后用它们来创造一些东西。在容器的世界里，你接受指令(YAML)和构建块(图像)并从中创建应用程序。

开放容器倡议(OCI)的目标是定义容器和运行时的标准。(成员包括 Red Hat、CoreOS，以及业内几乎所有其他公司。)如果你是一名架构师，OCI 会保护你的投资，因为你可以创建一次映像，并且知道你可以在可预见的未来使用它们 [²](#link2) ，并且知道工具、分发和后勤机制以及注册服务器将仍然存在并且仍然工作。

就像一个老笑话说的那样，标准的伟大之处在于你有太多的选择余地 [³](#link3) 。Scott 提到了由供应商、社区和标准机构推动的五个相关标准:

*   [OCI 图像规范](https://github.com/opencontainers/image-spec)
*   [OCI 分布规范](https://github.com/opencontainers/distribution-spec)
*   [OCI 运行时规范](https://github.com/opencontainers/runtime-spec)
*   [容器运行时接口](https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/devel/container-runtime-interface.md)(来自 Kubernetes)
*   [容器网络接口](https://www.cncf.io/blog/2017/05/23/cncf-hosts-container-networking-interface-cni/)(来自云本地计算基金会)

当你运行一个容器时，有一个映像，有一个注册表，还有一个主机。有了发行版规范，如果你把图片放在 Amazon 注册中心、Azure 注册中心、Red Hat 提供的注册中心等，你就可以得到保护。您可以移动该图像，并在另一个环境中运行它。

Scott 提到了一些随着标准变得更加根深蒂固而出现的工具。社区为 CRI 标准创建了 crictl T2。[波德曼](https://www.projectatomic.io/blog/2018/02/reintroduction-podman/) ( [现在可以作为技术预览](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/release_notes/technology_previews))提供了类似于 Docker 命令行的体验。runc 是一个命令行工具，根据 OCI 运行时规范运行容器。Project Atomic 创建了 [Buildah](https://www.projectatomic.io/blog/2017/11/getting-started-with-buildah/) 工具来构建符合 OCI 标准的映像。Buildah 的伟大之处在于它将与您的`Dockerfile` s 一起工作。Buildah 允许您在构建映像时将包添加到映像中，这样您的最终容器就不必有包管理器。当然，你可以在工具链中使用这些工具。

Scott 指出，与图像相关的 Docker 命令实际上是与存储库一起工作，而不是与图像一起工作。例如，此命令:

> `docker pull registry.access.redhat.com/rhel7/rhel:latest`

在`registry.access.redhat.com`转到注册表，然后在`rhel7`名称空间中查找，然后查找名为`rhel`的存储库，然后使用标签`latest` [⁴](#link4) 获取该映像的版本。这里的等级制度是`registry server/namespace/repo:tag`。注册服务器由 DNS 解析，但是根据注册服务器的不同，名称空间可以有不同的含义。在`registry.access.redhat.com`，名称空间是产品名称。在 Dockerhub，名称空间是提交图像的用户的名称。在您自己的注册表中创建图像时，考虑名称的不同组成部分的含义是很重要的。您的组织将如何使用名称空间？你将如何命名你的回购？(Scott 的一点建议:总是在你的`Dockerfile`中使用图片的完整 URL，以确保你得到的正是你想要的。)

注意:如果你想更好地处理注册和回购，并对容器有更深的理解，可以看看 Scott 的文章: [*容器术语实用介绍*](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/) 。这篇文章于 2018 年 2 月发表在 developers.redhat.com/blog上，完全更新了 Scott 2016 年的热门文章。2018 年版本包括了 docker 之外的容器世界正在发生的许多事情。

接下来，本开始讨论他构建图像的原则。他强调你应该对所有事情都使用源代码控制，这样你所有的工件都可以从代码中构建出来。他构建生产就绪容器的五个基本原则是:

*   使标准化
*   最小化存储
*   代表
*   过程
*   重复

我们将在接下来的几段中讨论这些。

*标准化:*你的目标应该是拥有一套具有共同血统的标准图像。您的基础映像将是应用程序框架、应用服务器、数据库和其他中间件。这里显而易见的好处是，您的图像更容易缩放，公共层的重用最大化，并且您的各种容器中的环境之间的差异最小化。注册表的大小可能是一个问题，特别是当工具链和构建管道发挥它们的魔力时，成千上万的开发人员不断地制作图像。在尽可能少的映像上实现标准化在注册表中、运行时以及任何需要更新基本映像的时候都有巨大的好处。(Red Hat 鼓励您使用我们的基本图像，尤其是 LTS 图像。)

*最小化存储:*目标是限制给定图像中的内容，特别是基本图像，以便它只包含您正在使用的内容。红帽提供了一个名为 [rhel7-atomic](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhel7-atomic) 的图像(不要与[项目 Atomic](https://www.projectatomic.io) 混淆)。这个图像有`glibc`和刚好足够的`rpm`来添加包。没有`python`，没有`systemd`，或者类似你可能不需要的东西。记住，有了图像，你就在建造一个沙箱。如果你的沙箱有体育场停车场那么大，那它就不再是沙箱了。

代表:图像的所有权应该属于对该图像最有专业知识的人，无论它包含什么。不要逞英雄；不要对你组织中的每一个形象负责。利用你团队的技能。

*关注流程和自动化:*这是最重要的规则。容器入门的门槛真的很低。创建一个`Dockerfile`运行`docker build`也没那么麻烦。这意味着做一件事很简单，一次就不再想它。但是容器并不是“一放了之”从测试到部署再到安全，你需要围绕它们的流程。本提到了像 [OpenSCAP](https://www.open-scap.org) 和 [Clair](https://coreos.com/clair/docs/latest/) 这样的工具，它们可以扫描你的图像寻找漏洞。 [⁵](#link5)

*迭代*是最终目标。不要重复过去的错误。做出改变已经不是什么大事了。如果您将测试和安全扫描作为构建链的一部分，那么您每次都会从“已知良好”到“已知良好”。

开发人员笔记本电脑上的映像应该与开发/测试环境中的映像相同，也应该与云中生产环境中的映像相同。(顺便说一下，这里的术语“开发人员”实际上是指“构建图像的任何人。”这可能包括系统管理员或架构师或其他人。)当您分发映像时，定义持久卷、秘密、缩放策略和其他元数据的 YAML 文件也应该随之一起分发。

最后，Ben 指出，您需要在一个有管道的系统中构建映像。你可以在你的本地机器上为[冒烟测试](https://en.wikipedia.org/wiki/Smoke_testing_(software))构建映像，但是任何重要的改变都应该通过管道运行。重要的是要记住，你不能只是在笔记本电脑上启动 Docker 来测试你的映像。要进行任何有意义的测试，您需要一个可以启动 Kubernetes 集群的环境，将生产中一起运行的所有映像拉下来，然后启动管道。没有人想在他们的笔记本电脑上运行一个完整的编排系统，但是这是你可以可靠地测试一组协同工作的服务的唯一方法。

一段伟大代码的标志不仅仅是它能工作，而是它的架构优雅、直观、灵活。大多数有经验的开发人员都知道如何去做。随着我们向前发展，创建一组优雅组合的容器的能力将是一项基本技能。每个容器中适当数量的功能、简化更改和升级的容器层次结构以及管道的适当使用都是生产就绪的容器化应用程序的一部分。

总而言之，这是一次很棒的会议，两位经验丰富的演讲者提供了许多最佳实践和良好建议。如果你想了解所有细节，斯科特和本的演示视频是 [100+红帽峰会 2018 分组会议](https://developers.redhat.com/blog/2018/05/15/100-red-hat-summit-2018-session-videos-online/)之一，你可以在线免费观看

https://youtu.be/nizud-1IK9c]

* * *

想想看，集装箱船上的集装箱就像乐高积木一样组装在一起。让这个比喻更加有力。

² 大概也就是几年吧。说真的，你今天做了多少 2015 年没做的事？

³ 我没说这个笑话好笑，我只是说它很老套。很抱歉。

⁴ 请记住，标签“最新”只是一个约定。它可能指向也可能不指向图像的最新版本。

⁵ 本推荐阅读 [*凤凰计划*](https://itrevolution.com/wp-content/uploads/files/PhoenixProjectExcerpt.pdf) 引以为戒。