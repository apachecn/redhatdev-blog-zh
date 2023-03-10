# 使用 Red Hat CloudForms 和 Openstack 构建您的软件定义的数据中心-第 2 部分

> 原文：<https://developers.redhat.com/blog/2017/11/02/build-software-defined-data-center-red-hat-cloudforms-openstack>

欢迎回来，在这里我们将继续我帖子的第二部分，我们将使用 **Red Hat Cloudforms** 。如果你还记得，在我们的第一篇文章中，我们谈到了**红帽 OpenStack 平台 11** (RHOSP)。除了这篇博客文章之外，在本文的结尾还有一个我制作的演示视频，向我们的客户/合作伙伴展示他们如何构建一个全自动化的软件数据中心。

**动手部分 2**

现在，我们需要一种能够为我们管理环境提供单一平台的东西。

它将管理我们的虚拟机管理程序、私有/公共云、Sdn、Paas，提供**自助服务**功能、**合规性**和**治理**、**预测瓶颈**，提供**预测准确性、按存储容量使用计费/按存储容量使用显示、**和**深度分析，并保护我们的环境**。

这里的产品名称是 **Red Hat CloudForms。**

![](img/b5f1095e894619a7d8002fe9707f3617.png)

*图片 1*

我邀请您阅读我们的博客[http://CloudFormsblog.redhat.com/](http://cloudformsblog.redhat.com/)和我们的官方文档[https://access . red hat . com/documentation/en/red-hat-CloudForms/](https://access.redhat.com/documentation/en/red-hat-cloudforms/)来充分了解 cloud forms 有多神奇！

现在让我们从一些极客的东西开始。

我们希望为我们的最终用户提供相同的热堆栈，但以自助方式提供。

CloudForms 通过**自助服务用户界面**能够向我们的用户展示不同类型的服务项目(不同提供商上的虚拟机供应、热堆栈执行、可行的行动手册即服务等。)，将它们组合成一个**服务目录**或一个服务目录包。

在某些情况下，您可能希望呈现一个**自助服务对话框**，它由一个简单的文本框、一个复选框或一个下拉列表组成，或多或少是您希望授予您的用户的一个简单 UI，只需点击几下鼠标就可以订购他们的服务！

让我在实践中向你展示我的意思。

您需要从 Red Hat 客户门户下载 Red Hat CloudForms 设备(qcow2 格式),然后将其导入 KVM。

记得使用 appliance_console 文本菜单设置 CF，并为 VMDB (postgres)添加一个专用磁盘，如此处指出的。[ [1](https://access.redhat.com/documentation/en-us/red_hat_CloudForms/4.5/html-single/installing_red_hat_CloudForms_on_red_hat_virtualization/#Configuring-CloudForms)

请注意，在 RH-V、OpenStack、Vmware Vsphere、Azure 和 Amazon EC2 上完全支持 CloudForms...但不是在 rhel + kvm 上，所以**不要**在生产环境中使用这种配置。

此处提供了能够托管 CloudForms 设备的平台的完整列表。[ [2](https://access.redhat.com/documentation/en-us/red_hat_CloudForms/4.5/html-single/support_matrix/#cfme_hosts)

让我们从管理界面开始导入 CloudForms 中的 heat stack。

从**服务**->-**编排模板**->-**配置**->-**创建新的编排模板**您将能够创建您的堆栈。

![](img/9d58bbdce2e261d1ac218a566144c84e.png)

*图片 2*

名称、描述和我们的堆栈内容是必需的；然后可以点击**添加。**

![](img/9af0c300e9dd9e050bf30cbdf21b12bb.png)

*图片 3*

现在我们必须创建服务对话框来管理堆栈的输入参数。

从**配置**->-**从流程编排模板创建服务对话框，**命名您的对话框，例如 stack-3tier-heat-service 对话框。

![](img/c2a6b26f1ff6ed27062705bc0cdd1f4e.png)

*图片 4*

现在，让我们进入**自动化** - > **自动化** - > **定制**来验证服务对话框是否创建正确。

![](img/01a0e84cd57de6cc76ab76abbf884ae8.png)

*图片 5*

点击**配置**->-**编辑。**

我们希望隐藏一些输入参数，因为您的客户/最终用户通常不知道体系结构的详细信息(例如堆栈名称或租户 id 或管理/Web 提供商网络 id)。

因此，让我们通过取消选择复选框**可见**和**必需**并设置一个**默认值来编辑至少这些值。**

下面是一个名为“demo-3tier-stack”的堆栈名称示例，不会向最终用户显示。

![](img/b432db2dc5f94a206fbbfbc0140b0976.png)

*图片 6*

至少对堆栈前缀、管理网络和 Web 提供商网络重复相同的配置。

请注意，管理网络和 Web 提供商网络将连接到我们的 OpenStack 外部网络，因此我们需要输入正确的网络 ID。

在我们的例子中，从我们的 all-in-one rhosp 中，我们可以使用以下命令获得该值:

[root @ OSP ~(keystone _ admin)]# open stack 网络列表-f 值-c ID -c 名称| grep FloatingIpWeb

**a18e 0 aa 1-88ab-44d 3-b751-EC 3d fa 703060**floating ipweb

![](img/7bb11175b9d71d6249e80fdd89a357f5.png)

*第七幅图*

完成修改后，我们将看到服务对话框的预览。

![](img/b243ad85280972194d9b6ede70bc1dda.png)

*图片 8*

酷！现在我们已经创建了我们的编排模板和服务对话框，让我们创建我们的服务目录，转到**服务**->-**目录**->-**目录**->-**所有目录。**

现在点击**配置**->-**添加一个新的目录。**

![](img/228ff1ea2039d8636e001eadb670c861.png)

*图片 9*

![](img/4011297919cd12155c1782fd604382a9.png)

*图片 10*

我们必须添加最后一项，即要在我们的“演示”服务目录下创建的服务目录项目。

进入**服务** - > **目录** - > **目录** **商品**，选择我们的“演示”目录。

![](img/5b871c413d13e451dded9d40c91c2262.png)

*图片 11*

选择编排作为**目录项目类型**并填写必填字段(在目录中显示非常**重要**)。

![](img/f1b127c1628fbd4c51c066b047475d3d.png)

*画面 12*

如果您想要限制服务目录项目的可见性，您可以从**策略**->-**编辑标签中选择要分配的标签。**

在这种情况下，我之前已经创建了一个组(Demo-developers-grp)的用户(开发人员)成员，并拥有一个名为“Demo-developers”的自定义角色。

![](img/f9feed0a152a3ae03a6e0858edc75af2.png)

*图片 13*

![](img/3bd8a85a2eabd15b9c2194c2bff12c24.png)

*图片 14*

我已授予自定义角色“演示开发人员”仅访问服务功能的权限，因此我们的开发人员用户将能够订购、查看和管理自助服务目录中的服务项目。

此外，我扩展了我们组的 rbac 功能，为用户组(图 13)和服务项目(*图 8)分配了一个名为“ **Access** 的定制标记。*

该地图允许演示开发者组的用户成员请求和管理标记有相同值的服务项目(注意前面图像上的 **My Company tags** 值)。

现在，我们可以订购我们的服务目录项目，让我们切换到自助服务用户界面(SSUI)，指向 URL[https://[IP address]/](https://192.168.122.11/ui/service/login)Self _ Service，并作为开发人员登录，我们将看到我们的 3 层堆栈服务项目。

![](img/da8c50a15c1c846941cd0d1349c38973.png)

*图片 15*

单击该服务，然后选择 CloudForms 租户(在此设置中启用了租户映射，因此 cloudform 租户可以准确映射我们的 OpenStack 租户)。如果您想更改输入参数或保留默认值。

![](img/f976db2638e93147ad9675d42955f0ad.png)

*图片 16*

让我们继续订购我们的服务项目，点击**添加到购物车**，然后点击**订单。**

![](img/affe928a717658fbaf53ac826b8b19c8.png)

*图片 17*

我已经编辑了标准的服务提供方法，以便在请求来自开发人员组的情况下请求批准，因此作为管理员并从 web 管理界面，批准来自**服务- >请求的请求。**

![](img/d806c859421cb7fe3f13a9c519009957.png)

*画面 18*

获得批准后，我们可以切换回自助服务用户界面，在这里，我们将在**我的服务**下找到我们订购的内容，几分钟后，OpenStack 中创建的所有酷东西都会显示出来。

![](img/000bcb9a76e3102ebc8ba2c487196535.png)

*图片 19*

![](img/606ac3182a8f008c95319f6f3a968e9b.png) *第二十幅图*

![](img/7b9b3075d9dd4b7892146eec17afa5b7.png)

*图片 21*

**演示视频**

[video width = " 1920 " height = " 1080 " MP4 = " https://developers . red hat . com/blog/WP-content/uploads/2017/11/buildsddc _ blog . MP4 "][/video]

**结论**

这只是一个示例，说明如何借助 Red Hat 产品和服务创建一个功能强大、完全自动化、可扩展的软件定义的数据中心

我们非常乐意帮助您和您的组织实现您的业务和技术目标。

这篇博客重点介绍了我们为一家大型金融客户的重要转型项目所做的部分工作。

非常感谢我的同事弗朗切斯科·沃勒罗、马特奥·皮齐尼尼、克里斯蒂安·容、尼克·卡特林、亚历桑德罗·阿里基耶洛和卢卡·米奇尼，感谢他们在比赛期间的支持。

* * *

[1][https://access . red hat . com/documentation/en-us/red _ hat _ cloud forms/4.5/html-single/installing _ red _ hat _ cloud forms _ on _ red _ hat _ virtual ization/# Configuring-cloud forms](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html-single/installing_red_hat_cloudforms_on_red_hat_virtualization/#Configuring-cloudforms)

[2]][https://access . red hat . com/documentation/en-us/red _ hat _ cloud forms/4.5/html-single/support _ matrix/# cfme _ hosts](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html-single/support_matrix/#cfme_hosts)

[3][https://access . red hat . com/documentation/en-us/red _ hat _ cloud forms/4.5/html-single/managing _ providers/# adding _ an _ open stack _ infra structure _ provider](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html-single/managing_providers/#adding_an_openstack_infrastructure_provider)

* * *

**无论你是 Linux 新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/linux-cheatsheet/) **都可以在遇到你最近没做过的任务时辅助你。**