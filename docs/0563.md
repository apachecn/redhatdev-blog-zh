# OpenShift 4.3 的新的和改进的拓扑视图

> 原文：<https://developers.redhat.com/blog/2020/01/16/new-and-improved-topology-view-for-openshift-4-3>

在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 控制台的开发者视角中的拓扑视图是一个精心设计的界面，提供了应用程序结构的可视化表示。这个视图有助于开发人员清楚地将一种资源类型与另一种资源类型区分开来，并理解应用程序中的整体通信动态。随着 open shift 4.2 版本的发布，拓扑视图已经在云原生应用程序开发领域赢得了关注。持续的反馈周期和对开发人员社区中正在进行的趋势的定期跟踪有助于在即将到来的版本中形成一种良好的体验。本文主要关注拓扑视图中为 OpenShift 4.3 添加的几个 showstopper 特性。

## **在列表视图和图形视图之间切换**

为了解决用户社区的一个常见问题，拓扑视图现在附带了一个切换按钮，可以在给定项目的列表视图和图表视图之间快速切换，如图 1 所示。虽然图形视图在需要识别应用程序架构中各个组件所扮演的角色的用例中非常方便，但列表视图对于更侧重于数据和调查的任务可能会有所帮助。这种切换的引入将实现视图间的无缝导航，而不管用例中的对比度如何。

![Red Hat OpenShift Topology view toggling between the List and Graph view.](img/843254f4c9152dbf02fbb16bce107941.png)

图 1:在列表和图表视图之间切换。

## **通过上下文动作菜单**选择组件

拓扑视图提供了一个详细的组件列表，作为图形的一部分，如图 2 所示。有各种资源类型、连接器、分组和事件源之类的项目，其中每一种都支持上下文中不同的一组操作。用户可以通过右键单击每个列出的项目来访问这个专用菜单，这将进一步打开一个包含所有可用操作的下拉列表。此外，用户可以单击菜单外的任何地方，使其从视图中消失。

![Red Hat OpenShift Graph view components.](img/13555e9b773833978bbc96b27560dec9.png)

图 2:通过拓扑的图形视图访问许多组件。

## **创建资源之间的绑定**

拓扑视图允许您通过从源节点拖动一个句柄并将其放在目标节点上来创建任何一对资源之间的连接，如图 3 所示。该功能通过提供一个智能评估来减少开发人员的认知负荷，该评估是关于一个[运营商管理的支持服务](https://developers.redhat.com/blog/2019/12/19/introducing-the-service-binding-operator/)是否可用于创建预期的绑定。在没有运营商管理的支持服务的情况下，创建基于注释的连接。

![Creating a connection between a pair of resources in OpenShift by dragging a handle from an origin node to the target node.](img/753e8e33d0712eed1fa24a0b955ae6b1.png)

图 3:创建资源之间的连接。

## **可视化** **吊舱在 r** **eal 时间**的过渡

4.3 中的拓扑视图提供了方便的前端访问，可通过侧面板扩大/缩小和增加/减少您的机架数量。类似地，用户也可以从上下文菜单(通过右键单击或从侧面板上的**动作**按钮访问)中开始展示或重新创建给定节点的窗格。在从侧面板执行相关的交互时，用户可以看到 pod 所经历的转换的实时可视化，如图 4 所示。

![Adjusting pod settings and view pod transitions in real time through OpenShift's Topology view.](img/0bd4958a616632d4ec554aa0fbf645ac.png)

图 4:调整 pod 设置并实时查看 pod 转换。

## **删除应用程序**

拓扑视图现在支持从图形视图中删除应用程序。通过调用给定应用程序分组的上下文菜单——通过执行右键单击或通过侧面板——用户可以访问 **Delete** 动作，如图 5 所示。确认此操作后，应用程序组(由带有相关标签的组件组成，如 Kubernetes 推荐的标签所定义)将被删除。

![Deleting an application in OpenShift's Topology Graph view.](img/5244395c8dabadc69e8000173bf5638f.png)

图 5:通过拓扑图视图删除应用程序。

## **可视化事件源接收器**

拓扑视图显示了 Knative Eventing 中的元素——即事件源，这为开发人员直观地提供了快速洞察哪些事件源将触发他们的应用程序，如图 6 所示。

![Viewing Knative event sources in the OpenShift Topology view.](img/d89a9450b0be00f1f9171d46b54c3d6e.png)

图 6:在拓扑视图中查看 Knative 事件源。

## **查看已知服务和相关修订**

用户现在可以在拓扑视图中查看 Knative 服务及其相关的修订和部署。活动流量块中的服务修订在拓扑视图中显示为一个组，以及它们的流量消耗信息，如图 7 所示。

![View Knative service Revisions and Deployments in the OpenShift Topology view.](img/f459de531e9f5aede1fa446d15138ba8.png)

图 7:在拓扑视图中查看服务修订和部署。

随着 Kubernetes 相关技术的不断发展以及新实践和集成的引入，OpenShift 不断更新以反映这一进展。

## **了解更多信息**

有兴趣了解更多关于 OpenShift 应用程序开发的信息吗？在 OpenShift 中查看我们关于[应用程序开发的资源。](http://developers.redhat.com/openshift)

## **提供反馈**

要提供反馈，要么加入我们的 [OpenShift 开发者体验谷歌小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，在那里您可以参与讨论或参加我们的办公时间反馈会议，要么随时给我们发一封[电子邮件](mailto:openshift-ux@redhat.com)，告诉我们您对 OpenShift 控制台用户体验的意见。

*Last updated: May 13, 2021*