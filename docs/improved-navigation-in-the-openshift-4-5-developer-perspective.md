# OpenShift 4.5 开发人员视角中改进的导航

> 原文：<https://developers.redhat.com/blog/2020/07/16/improved-navigation-in-the-openshift-4-5-developer-perspective>

新的 [Red Hat OpenShift 4.5 版本](https://developers.redhat.com/blog/2020/07/16/whats-new-in-the-openshift-4-5-console-developer-experience/)从 web 控制台的开发人员角度来看，包含了更加简化和可定制的导航体验。在本文中，我们将快速分享基于用户反馈添加的新导航功能的亮点。

## 一种新的可发现性导航方案

为了有助于发现，我们已经转移到一个平面导航方案，在主导航中有三个部分，如图 1 所示。

[![A screenshot of the new, three-part navigation scheme.](img/31e9dbcfcca382feafdcdf082affeb34.png "45-Nav-F01")](/sites/default/files/blog/2020/07/45-Nav-F01.png)

Figure 1: An overview of the main navigation in the OpenShift 4.5 web console's Developer perspective.

导航的第一部分是基于任务的。在这个部分中，您可以使用 **+Add** 按钮向您的项目添加新的应用程序或组件。本部分进一步分为三个区域:

*   **拓扑**:查看项目中的应用程序并与之交互。
*   **监控**:访问项目细节，包括仪表板、指标和事件。
*   **搜索**:根据资源类型、标签或名称查找项目资源。

第二部分是基于对象的，包括以下选项:

*   **构建**:访问构建、构建配置和管道。
*   **舵**:接近舵释放。
*   **项目**:访问项目信息。

第三部分是一个可定制的区域，您可以在其中添加最常用的资源以便快速访问。我们添加这一部分是基于反馈，即许多用户会从管理员的角度访问机密和配置映射等资源。默认情况下，您现在可以在此部分访问机密和配置映射。

## 可定制导航

现在，您可以使用搜索页面来查找条目并将其添加到导航中，如图 2 所示。

[![A screenshot of the Search page with the option to add an item to the navigation pane.](img/3b1f1a9176a2f7bf871accb2c1583414.png "45-Nav-F02")](/sites/default/files/blog/2020/07/45-Nav-F02.png)

Figure 2: Use the Search page to find components to add to the navigation.

您还可以通过将鼠标悬停在菜单项上并单击带有减号(-)的蓝色圆圈来从导航中删除项目，如图 3 所示。

[![A screenshot of the main navigation with the option to remove an item.](img/6fea93c9a4f3e8399cfe99fafbac054b.png "45-Nav-F03")](/sites/default/files/blog/2020/07/45-Nav-F03.png)

Figure 3: Easily remove items from the main navigation.

在 [OpenShift](https://developers.redhat.com/products/openshift/getting-started) 4.5 web 控制台中对开发者视角的导航改进基于用户反馈。我们希望这些更新能让你更容易、更快地与你的项目互动。

## 请给我们您的反馈！

OpenShift 开发人员体验流程的很大一部分是接收反馈并与我们的社区和客户合作。我们希望收到您的来信。我们希望您能在 [OpenShift 4.5 开发者体验反馈页面](https://forms.gle/zDd4tuWvjndCRVMD8)上分享您的想法。您还可以加入我们的 [OpenShift 开发者体验谷歌小组](https://groups.google.com/forum/#!forum/openshift-dev-users)，参与讨论并了解我们的办公时间会议，在那里您可以与我们合作，并提供关于您使用 OpenShift web 控制台的体验的反馈。

## 开始使用 OpenShift 4.5

你准备好开始使用新的 OpenShift 4.5 web 控制台了吗？[今天试试 open shift 4.5](http://www.openshift.com/try)。

*Last updated: January 25, 2021*