# OpenShift 4.3 控制台开发人员体验中的新功能

> 原文：<https://developers.redhat.com/blog/2020/01/15/whats-new-in-the-openshift-4-3-console-developer-experience>

开发者体验在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 4.3 web 控制台中得到显著改善。如果您使用过 OpenShift 4.2 控制台中引入的开发人员视角，您可能会熟悉我们用于部署应用程序的简化用户流程、新的拓扑视图，以及围绕由 Tekton 提供支持的 [OpenShift 管道和由 Knative 提供支持的 OpenShift 无服务器的增强体验。这个版本继续改进了 4.2 中引入的特性，并为开发人员引入了新的流程和特性。](https://developers.redhat.com/blog/2020/01/08/the-new-tekton-pipelines-extension-for-visual-studio-code/)

## 部署应用程序

开发人员视角提供了几种内置的方法来简化部署应用程序、服务和数据库的过程。在 4.3 中，您会发现这些用户流有以下改进:

*   支持从内部注册表部署图像流。

![](img/4c1ff118b777a47a2d371145d7d8189e.png)

*   从 Git 用户流导入中的生成器图像检测。

![](img/ec23d5128f4a7bc280f1ff4e3f041aa1.png)

*   Kubernetes 部署(默认)、OpenShift 部署配置和 Knative 服务(技术预览)之间的部署选项。(该选项为开发人员提供了在部署类型之间切换的简单方法，无需学习 YAML 或其他模板解决方案。

![](img/5503f2564f4c2d86b4f97bb9532e69cd.png)

## 拓扑视图

拓扑视图为 4.3 提供了显著的可用性改进和新的流程:

*   在项目的图形表示和列表表示之间切换。

![](img/c3da1cedc4161ef698c2acf655e6ee79.png)

![](img/214ede6342a4fda3360b0f03fcf6e4b9.png)

*   通过侧面板或相关的详细信息页面，可以轻松地扩大/缩小和增加/减少 pod 数量。

![](img/9a23811997a818e5df73ead97aa0a2b0.png)

*   在拓扑中以及相关的侧面板中查看滚动的实时可视化效果，并在组件上重新创建展开。
*   删除一个应用程序，这是通过删除带有相关标签的所有组件来完成的，如 Kubernetes 推荐的标签所定义的。

![](img/214ede6342a4fda3360b0f03fcf6e4b9.png) ![](img/ae2a5adc45c67cdd59b0f9eea874c8e7.png)

*   可以通过右键单击以及相关侧面板中的操作菜单来访问上下文菜单。

![](img/be2e9e003450aa9d33a860d198d72227.png) ![](img/07482837f24256fa23732e9fcd5012cf.png)

## 多方面的

其他改进使您能够:

*   单击新添加的项目详细信息导航项目，以访问新的项目仪表板并删除您的项目。

![](img/1da0f1990464db0ca6bf02d5509d564e.png)

*   如果您是具有适当访问权限的开发人员，可以通过简化的项目成员视图与其他用户轻松共享您的项目。

![](img/9459b18de96c872ea5396dc270face49.png)

*   通过对项目运行 Prometheus Query Language (PromQL)查询并检查绘图上显示的指标，解决应用程序的问题。该技术预览功能在指标页面上提供。

![](img/ddf8bd86a464e0c1a2291974e1e0b1d7.png)

## 有约束力的

[服务绑定操作符](https://github.com/operator-backing-service-samples/postgresql-operator)使应用程序开发人员能够更轻松地将应用程序与操作符管理的后台服务(如数据库)绑定在一起，而无需手动配置机密、配置映射等。在 4.3 版的拓扑中，您可以快速轻松地在两个组件之间创建绑定。

### OpenShift 管道

OpenShift 管道运营商利用利用 Tekton 项目的开发人员 CI/CD 解决方案增强了 OpenShift 控制台。在 4.3 中，您将看到以下改进:

*   用户可以启用应用程序的 CI/build 管道。
*   当管道与拓扑中的组件相关联时，用户将能够在拓扑视图中查看关联以及预览管道的状态。
*   管道运行的“日志”选项卡提供了实时查看任务日志的能力。用户现在能够下载管道的任务日志。

![](img/fccef006adc68c6ba1556e9e9a93bb2f.png)

## Red Hat OpenShift 无服务器

在 4.3 中，对技术预览版 [Red Hat OpenShift 无服务器](https://www.openshift.com/learn/topics/serverless)功能的改进包括:

*   将 Knative 服务可视化为一个组，这允许用户在拓扑视图中直观地查看流量块中的所有修订。
*   允许用户修改已知服务版本之间的流量分配。

![](img/8e4cedeb445a05262fb68a6410e96ba2.png)

*   显示 Knative Eventing 的元素，即事件源，这为开发人员提供了快速洞察哪些事件源将触发他们的应用程序。

![](img/2c7a13b2105b7a332e2dd76d52ea3fbb.png)

## 了解更多信息

有兴趣了解更多关于 OpenShift 应用程序开发的信息吗？在 OpenShift 上查看这些关于[应用开发的 Red Hat 资源。](http://developers.redhat.com/openshift)

## 提供反馈

加入我们的 [OpenShift 开发者体验 Google 小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，参与讨论，或者参加我们的办公时间反馈会议。或者，给我们发一封[电子邮件](mailto:openshift-ux@redhat.com)，告诉我们您对 OpenShift 控制台用户体验的意见。

*Last updated: June 29, 2020*