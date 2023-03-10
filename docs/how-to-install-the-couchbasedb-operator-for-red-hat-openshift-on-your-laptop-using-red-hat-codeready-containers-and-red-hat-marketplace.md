# 如何使用 Red Hat CodeReady 容器和 Red Hat Marketplace 在您的笔记本电脑上安装用于 Red Hat OpenShift 的 CouchbaseDB 操作符

> 原文：<https://developers.redhat.com/blog/2020/09/09/how-to-install-the-couchbasedb-operator-for-red-hat-openshift-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace>

[Red Hat Marketplace](https://marketplace.redhat.com/en-us) 是一个网上商店，在这里你可以选择你想要在你的 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/getting-started) 集群上安装和运行的软件。打个比方，这是一个手机应用程序商店，你在那里选择一个应用程序，它就会*自动*安装到你的手机上。有了 Marketplace，您只需注册您的集群，选择您想要的软件，它就会为您安装。这再容易不过了。

在本文中，我将向您展示如何在 OpenShift 集群上安装 couch base Server Enterprise Edition。在我的例子中，集群使用 Red Hat CodeReady Containers (CRC)在 Fedora 32 上运行。Couchbase Server 企业版目前可以免费试用，CRC 也可以零成本获得。这种设置提供了一种无风险的方式来尝试[容器](https://developers.redhat.com/topics/containers)、 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 、OpenShift，以及在本例中的 Couchbase。这绝对是“开发人员摆弄软件”级别的东西。

## 先决条件

我们首先假设您的机器上安装了 CodeReady 容器。CRC 是免费的，它将最新版本的 OpenShift 带到了你身边的 PC 上，所以[去抢吧](https://www.openshift.com/try)。你可以在这里找到[Windows 10 Enterprise](https://developers.redhat.com/blog/2020/09/09/how-to-run-red-hat-codeready-containers-on-windows-10-enterprise/)和[MAC OS](https://developers.redhat.com/openshift/local-openshift-macos)的安装说明。

**注意**:你可以在 try.openshift.com[，](https://www.openshift.com/try)获得免费的 CodeReady Containers，在那里你还可以找到其他免费的 OpenShift 学习材料。

一旦您的集群启动并运行，请访问 [Red Hat Marketplace 网站](https://marketplace.redhat.com/en-us)，在这里您将被要求登录或创建一个帐户。如果您没有帐户，您需要创建一个。

为了完全公开，你需要注册一张信用卡来安装任何来自 Red Hat Marketplace 的软件，包括 Couchbase 免费试用版。然而，免费试用不会 *—* 重复，*也不会—* 自动转为付费订阅。你永远不会有你的信用卡被自动扣款的风险。不要让注册拖了你的后腿；这里有太多对你有利的东西。

## 第一步:注册红帽市场

假设您有一个帐户，第一步是进入 [Red Hat Marketplace 主页](https://marketplace.redhat.com)，如图 1 所示，并登录。

[![red hat marketplace main web site with option to log in or create an account](img/9662605ca1aed5adf0f91ef899c7f68c.png "rhm-website")](/sites/default/files/blog/2020/08/rhm-website.png)

Figure 1: Sign in to Red Hat Marketplace.

登录后，单击右上角的用户名，从图 2 所示的下拉列表中选择 **My account** 选项。

[![my account option on red hat marketplace web page](img/6d3ca3c0b1b0a7895e21f27246005a80.png "my-account-menu")](/sites/default/files/blog/2020/09/my-account-menu.png)Figure 2: Select 'My account'.">

## 步骤 2:注册您的集群

登录到您的帐户后，您可以访问您的工作空间并注册您的 OpenShift 集群。你只需要注册一个集群，非常简单。图 3 显示了我的个人资料页面和帐户信息；你的应该看起来差不多。

[![A screenshot of the Red Hat Marketplace 'My profile' page.](img/100fcb4aa09ad97f1302c36e04535213.png "my-profile-page")](/sites/default/files/blog/2020/09/my-profile-page.png)Figure 3: An example profile page for Red Hat Marketplace.">

当您点击个人资料页面顶部的**工作区**时，您将看到您已经安装的软件列表。在新群集中安装任何软件之前，您需要注册它。注册集群会安装 Red Hat Marketplace 操作器，它会与 Red Hat Marketplace 网站对话并安装您需要的软件。

首先，点击**添加集群**，如图 4 所示。

[![add cluster page](img/402dd8c4a9d16264f2eb0d73c8367965.png "add-cluster-page")](/sites/default/files/blog/2020/09/add-cluster-page.png)Figure 4: Click 'Add cluster'.">

**添加集群**页面将引导您完成接下来的步骤。

### 生成您的秘密

首先，您将生成一个拉秘密，如图 5 所示。

[![A screenshot of the 'Add cluster' setup page.](img/50f3edd5d3464cbf9f694ea45900725d.png "add-cluster-prompt")](/sites/default/files/blog/2020/09/add-cluster-prompt.png)Figure 5: Generate your pull secret.">

### 安装红帽市场操作员

当您点击 **Generate Secret** 时，**安装 Red Hat Marketplace Operator** 的命令(图 5 中的步骤 2)会更新以包含您的秘密。点击图 6 所示的**复制**图标，将命令复制到您系统的剪贴板上。

[![A screenshot of the 'Add cluster' page with the command-line command to install the Operator.](img/5740de9495009e17185208fc300c8386.png "generated-command-line")](/sites/default/files/blog/2020/09/generated-command-line.png)Figure 6: Get the command and secret to install the Red Hat Marketplace Operator.">

### 注册您的集群

现在，将该命令粘贴到命令行中并运行它。几分钟后，您将能够返回到 web 界面来查看您新注册的集群，如图 7 所示。

[![The web site display of the new cluster.](img/2f20ce91f099935619694db809716fc8.png "new-cluster-is-shown")](/sites/default/files/blog/2020/09/new-cluster-is-shown.png)Figure 7: Check the cluster registration.">

当您的集群注册后，您还会收到一封电子邮件，因为没有人会收到足够的电子邮件。

### CodeReady 容器的附加注册

如果您正在使用 CRC(并且仅当您正在使用 CRC 时)，您将需要完成一些额外的步骤，以确保您的群集具有从 Red Hat Marketplace 安装软件所需的凭证。对于 CRC 集群，您只需要这样做一次，大约需要两分钟。

转到市场文档页面，按照 *[中的说明在 Red Hat code ready Containers](https://marketplace.redhat.com/en-us/documentation/clusters#register-openshift-cluster-on-red-hat-codeready-containers)*上注册 OpenShift 集群。这是九个简单的步骤，也是 CRC 成功的必要条件。如果您不使用 CRC，可以跳过这一部分。

## 第三步:选择你的软件

您的群集已在您的 PC 上启动并运行，并且已在 Red Hat Marketplace 中注册。现在有趣的部分开始了:通过简单的鼠标点击找到并安装一个免费软件试用版。如前所述，对于这个例子，我选择了 Couchbase Server 企业版。

### 搜索市场

您可以搜索或过滤以查看 Red Hat Marketplace 提供了哪些软件。图 8 显示了有各种产品类别可供选择的搜索页面。

[![A web page with options for filtering by type of software.](img/3163c3c6f31265496c8354cddf4f03f3.png "filter-for-database")](/sites/default/files/blog/2020/08/filter-for-database.png)Figure 8: The Red Hat Marketplace search page.">

### 按类别过滤

我们选择**数据库**类别，它返回如图 9 所示的选项(目前总共有 13 个数据库选项)。

[![All of the Database offerings on Red Hat Marketplace.](img/b4555fedadf7968e47c8f89ab30c396b.png "Screen Shot 2020-09-04 at 10.16.42 AM")](/sites/default/files/blog/2020/09/Screen-Shot-2020-09-04-at-10.16.42-AM.png)Figure 9: Database products available from Red Hat Marketplace.">

### 选择您的免费试用

点击 Couchbase Server Enterprise Edition 面板会将您带到图 10 所示的 couch base 安装页面。

[![The Couchbase Server Enterprise Edition page with purchase and free trial options](img/9fe99d4011fc78abb6a291bad5fc4c7e.png "Screen Shot 2020-09-04 at 10.42.06 AM")](/sites/default/files/blog/2020/09/Screen-Shot-2020-09-04-at-10.42.06-AM.png)Figure 10: The Couchbase Server Enterprise Edition page with the free-trial option.">

点击**免费试用**按钮继续，您将看到图 11 中的屏幕。

[![The purchase summary shows no cost for the free trial.](img/4a59baffb3353d313bc6d41262f1e50e.png "couchbase-start-trial")](/sites/default/files/blog/2020/09/couchbase-start-trial.png)Figure 11: Start the free trial.">

## 步骤 4:设置并运行安装

现在，您将选择您的集群，选择您希望安装 Couchbase 的名称空间(OpenShift 项目),并启动安装。填写完这些信息后，如图 12 所示，只需点击一下，就可以在集群中安装免费试用版。

[![The web page with Couchbase installation options.](img/6a84f8f46c5b176dfe0961b434c08642.png "couchbase-install-options")](/sites/default/files/blog/2020/09/couchbase-install-options.png)Figure 12: Set up the installation.">

图 13 显示了正在进行的安装。

[![A web page showing Couchbase being installed from Red Hat Marketplace.](img/22eafeba940bd66d7fc3c3d5d34aaaf6.png "couchbase-is-installing")](/sites/default/files/blog/2020/09/couchbase-is-installing.png)Figure 13: The installation in progress on Red Hat Marketplace.">

### 确认成功！

现在，真正酷的事情来了:你可以切换到你的 OpenShift 仪表盘，选择**已安装的操作符**，然后观察安装过程。

[![OpenShift dashboard showing Couchbase being installed.](img/87f8cf2ec553cdadf58d5be47f3e4cd2.png "couchbase-installing-dashboard")](/sites/default/files/blog/2020/09/couchbase-installing-dashboard.png)Figure 14: The installation in progress.">

在我的电脑上，从**安装**(图 14)到【T2 成功】(图 15)的状态切换用了大约两分钟。

[![OpenShift dashboard showing Couchbase Operator installed.](img/ea1850560738487c49bf9f177a5747e2.png "installed-operator")](/sites/default/files/blog/2020/09/installed-operator.png)Figure 15: The installation succeeded.">

## 结论

就是这样:通过简单的注册和几次点击，您已经在集群中安装了 CouchbaseDB 操作符。如果你想体验 Couchbase，请访问这项技术的优秀的[入门网页](https://blog.couchbase.com/fully-configured-couchbase-on-red-hat-openshift-under-five-minutes/)。此外，如果您想知道，您可以在任何时候使用 Red Hat Marketplace 卸载 CouchbaseDB 操作符。

想再举个例子吗？你可以在 *[中找到一个更通用的使用红帽 CodeReady 容器和红帽市场](https://developers.redhat.com/blog/2020/09/09/install-red-hat-openshift-operators-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace/)* 在你的笔记本电脑上安装红帽 OpenShift 操作符。