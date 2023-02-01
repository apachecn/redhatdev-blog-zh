# Jakarta EE 8:Java EE 的新时代

> 原文：<https://developers.redhat.com/blog/2019/09/12/jakarta-ee-8-the-new-era-of-java-ee-explained>

Java EE 是一个了不起的项目。然而，它是在 1999 年以 J2EE 的名字创建的，已经有 20 年的历史了，这意味着它在满足企业需求方面也面临着挑战。

现在， [Java EE 有了新家，有了新品牌](https://developers.redhat.com/videos/youtube/f2EwhTUmeOI/)。该项目从 Oracle 迁移到了 [Eclipse Foundation](https://www.eclipse.org/org/) ，它被称为 Jakarta EE，隶属于 Eclipse Enterprise for Java (EE4J)项目。Eclipse 基金会在 9 月 10 日发布了 [Jakarta EE 8](https://jakarta.ee/release/) ，在本文中，我们将看看这对企业 Java 意味着什么。

Java EE 是一个非常强大的项目，被广泛应用于多种企业 Java 应用程序和许多大型框架中，如 [Spring](https://spring.io/) 和 [Struts](https://struts.apache.org/) 。开发人员可能会质疑它的功能和发展过程，但看看它在市场上的高使用率和时间，它的成功是不可否认的。尽管如此，企业世界并没有停止，新的挑战一直在涌现。变化的速度越来越快，云计算等新技术被开发出来以提供更好的解决方案，Java EE 也需要跟上步伐。

## 雅加达 EE 目标

Java 生态系统对云计算有了新的关注，Jakarta EE 是这种方法的关键。Jakarta EE 的目标是加速云计算的业务应用程序开发(云原生应用程序)，使用许多供应商开发的规范。这个项目基于 Java EE 8，它的规范、技术兼容性工具包(tck)和参考实现(RI)都是从 Oracle 迁移到 Eclipse Foundation 的。

然而，要为云计算发展这些规范，我们不能使用 Java EE 上使用的相同流程，因为它们对于当前的企业挑战来说太慢了。因此，Eclipse Foundation 的第一个行动是改变过程以发展 Jakarta EE。

Jakarta EE 8 拥有与 Java EE 8 相同的一组规范，其特性没有任何变化。唯一的变化是发展这些规范的新过程。因此，Jakarta EE 8 是 Java 企业历史上的一个里程碑，因为它将这些规范插入到一个新的流程中，从而将这些规范提升到一种云原生应用程序方法。

## 雅加达 EE 规范流程

[Jakarta EE 规范流程(JESP)](https://jakarta.ee/about/jesp/) 是新的流程，将由 [Jakarta EE 工作组](https://jakarta.ee/about/)用来发展 Jakarta EE。JESP 正在取代以前用于 Java EE 的 [JCP 进程](https://www.jcp.org/en/home/index)。

JESP 基于 Eclipse Foundation 规范过程(EFSP ),有一些变化，这些变化包含在[项目页面](https://jakarta.ee/about/jesp/)中。变化如下:

*   对雅加达电气工程规范流程的任何修改或修订，包括采用新版本的 EFSP，都必须得到规范委员会绝大多数成员的批准，包括雅加达电气工程工作组的绝大多数战略成员，以及 EFSP 中规定的任何其他投票要求。
*   *所有规范委员会批准投票周期将具有如下所述的最短持续时间(尽管 EFSP 规定了例外程序，但这些周期不得缩短)*
    *   *创作评审:7 日历天；*
    *   *计划评审:7 日历天；*
    *   进度回顾:14 个日历天；
    *   *发布审核:14 日历天；*
    *   服务发布审核:14 个日历日；和
    *   JESP 更新:7 个历日。
*   如果规范团队退出相应的审查，投票将被宣布无效并立即结束。
*   *规范项目在积极开发的同时，每年必须至少进行一次进展或发布评审*。

JESP 的目标是尽可能的轻量级，其设计更接近于开源开发，并且牢记代码优先开发。因此，JESP 提倡一种新的文化，这种文化注重实验，以通过实验获得的知识为基础来发展这些规范。

[![](img/c2b9f06305c4b924feb10978643812c5.png)](https://che.openshift.io/f?url=https://raw.githubusercontent.com/redhat-developer/devfile/master/getting-started/java-maven/devfile.yaml/?sc_cid=7013a000002D1quAAC)

## Jakarta EE 9

Jakarta EE 8 专注于更新其流程以进行发展，第一次功能更新将于 [Jakarta EE 9](https://www.eclipse.org/community/eclipse_newsletter/2019/february/Jakarta_EE_9.php) 发布。预计雅加达 EE 9 的主要更新是雅加达 NoSQL 规范的诞生。

[雅加达 NoSQL](https://projects.eclipse.org/proposals/jakarta-nosql) 是一个规范，旨在简化 Java 应用程序和 NoSQL 数据库之间的集成，促进一个标准的解决方案，用一个高层次的抽象将它们连接起来。这个功能太棒了。这也是让 Java 平台更接近云原生方法的一大步，因为 NoSQL 数据库在云环境中被广泛使用，并且有望得到改进。雅加达 NoSQL 基于 [Eclipse JNoSQL](http://www.jnosql.org/) ，这将是它的参考实现。

Jakarta EE 中的另一个更新涉及名称空间。基本上，Oracle 将 Java EE 项目交给了 Eclipse 基金会，但 Oracle 仍然持有商标。这意味着 Eclipse Foundation 不能在 Jakarta EE 的新特性的项目名或名称空间中使用 Java 或`javax`。因此，社区正在讨论过渡到`jakarta.*`名称空间。你可以在这里阅读讨论帖[。](https://www.eclipse.org/lists/jakartaee-platform-dev/msg00029.html)

## 结论

Jakarta EE 8 标志着 [Java 生态系统](https://developers.redhat.com/blog/2019/09/05/why-java-is-so-hot-right-now/)的新时代；它让重要的 Java EE 项目在开源流程下工作，并为必要的改进铺平了道路。虽然这个 Jakarta EE 版本没有特性更新，但是它为将来的新特性打开了大门。因此，我们将在 Jakarta EE 的下一个版本中看到许多基于云开发规范的解决方案。

*Last updated: July 1, 2020*