# Eclipse MicroProfile 和 Red Hat 更新:Thorntail 和 SmallRye

> 原文：<https://developers.redhat.com/blog/2018/08/23/eclipse-microprofile-and-red-hat-update-thorntail-and-smallrye>

在过去的三个月中，Red Hat 的 Eclipse MicroProfile 发生了一些变化。如果你没有关注细节，这篇文章概括了变化并介绍了 Thorntail 和 SmallRye。

## 再见了，野生蜂群！你好索恩泰尔！

你可能错过了这条重要新闻。我们的 MicroProfile 实现在两个月前更改了名称。

在收到社区的大量反馈后，我们决定将“WildFly Swarm”更名为 Thorntail。虽然以前的名字很好听，但我们发现“Swarm”这个术语在 it 行业中有点超载，可能会令人困惑。“野花”的部分也是一样；与我们的 Java EE 应用服务器共享这个名称对一些用户来说是混淆的来源，使他们认为这是 WildFly 的子项目。

有了这个名字，我们也改变了版本控制，回到了更具语义的版本编号。因此，WildFly Swarm 的最后一个发布版本是 [2018.5.0](http://wildfly-swarm.io/posts/announcing-wildfly-swarm-2018-5-0/) ，Thorntail 的第一个版本(相同代码，不同名称)是 [2.0.0.Final](http://wildfly-swarm.io/posts/announcing-thorntail-2-0-0-final/) 。

更改版本编号使我们更容易交流新特性，并更好地链接到下游项目版本。

你会在 Bob McWhirter 接受 InfoQ 的采访中找到更多关于项目重命名和版本变更的信息。

由于还是同一个项目，重命名对现有项目没有技术上的影响，但是你需要更改 Maven 工件`groupid`和`artifactid`以及一些插件名称，以坚持新版本。所有的迁移细节都在 [Thorntail 2.0.0 .最终发行说明](http://wildfly-swarm.io/posts/announcing-thorntail-2-0-0-final/)中列出。

你可以在 [thorntail.io](http://thorntail.io) 找到关于使用或贡献 Thorntail 的一切信息。

## SmallRye 来了，这是一个共享的微文件实现

几个月前，肯·芬尼根发起了一个关于微文件邮件列表的讨论，以启动一个围绕微文件实现的计划。

MicroProfile 是一个快速变化的目标，自从两年前宣布以来已经发展了很多。跟踪规范和匹配实现的快速发展需要所有供应商投入大量精力，因此 Ken 建议将这些实现工作的公共部分放入一个独立于供应商的 MicroProfile 实现中，其项目名称为 SmallRye。

在 [smallrye.io](https://www.smallrye.io/) 你可以看到这个社区驱动的项目做得很好:所有的 MicroProfile 规范现在都有了自己的实现。

源代码可以在 [SmallRye GitHub](https://github.com/smallrye) 库查看。

Thorntail 的 2.1.0.Final 版本是第一个使用 SmallRye 实现的 MicroProfile 容器。

所以现在，除了为所有的 MicroProfile 规范做出贡献之外，由于 SmallRye 的存在，您还可以为一个惠及整个社区的实现做出贡献。

## 下一步是什么？

Thorntail 团队正在为 MicroProfile 社区和 Red Hat 客户的未来开发许多好东西。

例如，即将发布的 Thorntail 版本将开始使用 Java EE 8 规范，如 CDI 2.0 或 JAX-RS 2.1，SmallRye 将在下一版本中支持 MicroProfile 1.4。

如果您想保持最新状态，您可以:

*   在 [Twitter](https://twitter.com/thorntail_io) 上关注我们。
*   查看我们的[谷歌群](https://groups.google.com/forum/#!forum/thorntail)。
*   与 IRC 上的其他社区成员聊天:freenode 上的#Thorntail 频道。

回头见！

*Last updated: November 15, 2018*