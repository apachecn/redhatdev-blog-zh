# 使用 creative Coderland 教程开始反应式编程

> 原文：<https://developers.redhat.com/blog/2019/07/31/get-started-with-reactive-programming-with-creative-coderland-tutorials>

[Reactica 过山车](https://developers.redhat.com/coderland/reactive/)是 [Coderland](https://developers.redhat.com/coderland/) 的最新成员，这是我们为开发者设计的虚拟游乐园。它展示了[反应式计算](http://reactivemanifesto.org)的威力，这是一种重要的架构，用于处理使用异步数据相互协作的微服务组。

在这个场景中，我们需要构建一个 web 应用程序来显示不断更新的过山车等待时间。

## reactiva 过山车

练习中的不同微服务会产生如下事件:

*   一位客人排队买杯垫。
*   一位游客上了过山车。
*   一名游客从过山车上下来。
*   游乐设备开始运行，搭载一定数量的游客，并将这些游客的状态从“排队”更改为“乘坐中”
*   游乐设备停止，将一些游客的状态更改为“已完成游乐设备”

该场景使用[红帽 AMQ](https://developers.redhat.com/products/amq/) 和[红帽数据网格](https://developers.redhat.com/products/datagrid/)来完成它的工作。有一个组件可以生成新的`User`对象和新的`Ride`对象，新的`User`对象在运行过程中需要一些`User`。关于那些`User`的数据存储在 AMQ 和数据网格中；`Ride`号天体的详细资料储存在 AMQ。从那里，我们有几个组件监视数据网格和 AMQ，以随着时间的推移改变`User`和`Ride`对象的状态。

过山车展示了反应式编程的基本定义，正如 Andre Staltz 的反应式教程所定义的:

> *反应式编程是用异步数据流编程。*

通过阅读 Reactica repo 附带的文章和教程，您可以看到微服务是如何协同工作的。通过停止系统的某些部分，您还可以看到响应性和弹性的反应原则。系统的其余部分继续工作，您可以重新启动部分系统，并查看整个应用程序如何继续工作，在一些微服务关闭时恢复已创建但未在系统中传播的数据。

总的来说，我们认为这是对反应式编程的很好的介绍。这很有趣。我们真的很喜欢把这些内容放在一起，我们希望你也喜欢。一如既往，我们希望听到您的反馈。您可以在文章或 YouTube 视频中发表评论，并通过[coderland@redhat.com](mailto:coderland@redhat.com)联系我们。

祝您参观[科德兰](https://developers.redhat.com/coderland/)愉快！

*Last updated: August 6, 2019*