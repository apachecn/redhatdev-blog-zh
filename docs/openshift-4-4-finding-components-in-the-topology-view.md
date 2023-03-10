# OpenShift 4.4:在拓扑视图中查找组件

> 原文：<https://developers.redhat.com/blog/2020/04/30/openshift-4-4-finding-components-in-the-topology-view>

自从引入以来， [Red Hat OpenShift](https://developers.redhat.com/openshift/) web 控制台的开发人员视角中的拓扑视图已经在创造性地增强开发人员的能力方面取得了很大进展。它帮助他们摆脱了文本提示的限制，开启了一个全新的视觉交互主导的操作世界。但是拓扑视图甚至还没有完成追求卓越开发体验的过程。

让我们来看看新的查找功能。

## 查找功能

[Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview) 4.4 中新的 find 特性增加了开发者应用内部组件的可发现性。您可以使用此功能按名称显示相关组件或组。根据 OpenShift 用户的反馈，这个特性是改进拓扑视图的主要工作，以便更有效地进行修复和增强。这个特性也有助于装备拓扑图视图来处理具有更多资源的更大的画布。

### 使用查找

当按名称查找组件或组时，用户可以在查找组件的文本输入字段中键入字符串。如果找到匹配项，它会在图形视图中突出显示，并且与不匹配项相比具有更高的不透明度。不匹配项仍然以柔和的不透明度显示在图上，以保留组件存在的上下文，而不允许它掩盖匹配项，如图 1 所示。

[![OpenShift Topology view finding components](img/b44b2253537f9b1af082d0200a513a42.png "figure2-find-2")](/sites/default/files/blog/2020/04/figure2-find-2.gif)Figure 1: Finding components in the Topology view.">

### 查看可视区域之外的比赛

匹配有可能在可视区域之外。我们添加了一个触发器来通知他们这种可能性，以及一个快捷操作按钮来适应视图中的整个项目，如图 2 所示。

[![OpenShift showing find matches outside of the visible area](img/d4e8f96daca35c08268cbed49274802b.png "figure1-find-1")](/sites/default/files/blog/2020/04/figure1-find-1.gif)Figure 2: Viewing find matches outside of the visible area.">

find 特性让我们离处理可伸缩性更近了一步。在未来的版本中，我们对拓扑视图有宏伟的计划。

## 准备好开始了吗？

[今天试试 OpenShift】。](http://www.openshift.com/try)

## 提供您的反馈

请留意其他选项，以帮助提高未来的可伸缩性！让我们帮助您，[在此分享您的想法](https://forms.gle/6HArjszuqyE1xr3f8)，我们将在下一轮拓扑设计中将其作为输入。此外，加入我们的 [OpenShift 开发者体验谷歌小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，参与讨论并了解我们的办公时间会议，在那里您可以与我们合作并提供反馈。

## 了解更多信息

如果您有兴趣了解更多关于 OpenShift 的应用程序开发，请从 OpenShift 上的[应用程序开发](https://developers.redhat.com/openshift)[了解更多](https://developers.redhat.com/blog/2020/04/30/whats-new-in-the-openshift-4-4-web-console-developer-experience/)。

*Last updated: June 29, 2020*