# 使用 Red Hat CodeReady 容器和 Red Hat Marketplace 在您的笔记本电脑上安装 Red Hat OpenShift 操作符

> 原文：<https://developers.redhat.com/blog/2020/09/09/install-red-hat-openshift-operators-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace>

[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)(CRC)是开发者在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 4.1 或更新版本上开始使用集群的最快方式。CodeReady 容器设计为在本地计算机上运行。它通过在本地模拟云开发环境简化了设置和测试，并提供了开发基于[容器](https://developers.redhat.com/topics/containers/)的应用程序所需的所有工具。

[Red Hat Marketplace](https://marketplace.redhat.com/) 是一个开放的云市场，让您可以轻松发现和购买构建企业级应用所需的经过认证的容器化工具。它旨在帮助开发人员使用 OpenShift 构建应用程序，并在混合云中部署它们。Red Hat Marketplace 可以在任何运行 CodeReady 容器的开发人员工作站上工作。

本文将指导您完成设置 Red Hat Marketplace 和在本地基于 CodeReady 容器的 OpenShift 集群中安装容器化产品的步骤。

## 步骤 1:安装 CodeReady 容器

CodeReady Containers 作为[Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux)(RHEL)虚拟机交付，支持 Linux、macOS 和 [Windows](https://developers.redhat.com/blog/category/windows/) 10 的本机虚拟机管理程序。对于本文，我们将在 macOS 上使用带有 [OpenShift 4.5.7](https://developers.redhat.com/blog/2020/08/18/openshift-4-5-bringing-developers-joy-with-kubernetes-1-18-and-so-much-more/) 的 CodeReady Containers 版本 1.15。

安装 CodeReady 容器需要您使用 Red Hat ID 登录，并下载 CodeReady 容器档案以及 [pull secret](https://cloud.redhat.com/openshift/install/crc/installer-provisioned) 文件。以下是完整的安装说明:

*   macOS 上 *[本地 OpenShift 开发环境 macOS 上](https://developers.redhat.com/openshift/local-openshift-macos)*
*   Windows 10 企业版 [*如何在 Windows 10 企业版*](https://developers.redhat.com/blog/2020/09/09/how-to-run-red-hat-codeready-containers-on-windows-10-enterprise/) 上运行红帽 CodeReady 容器

一旦安装了 CodeReady Containers，您应该会看到类似这样的输出，表明命令`crc setup`成功完成:

```
INFO Checking if oc binary is cached
INFO Caching oc binary
INFO Checking if podman remote binary is cached
INFO Checking if goodhosts binary is cached
INFO Checking if CRC bundle is cached in '$HOME/.crc'
INFO Unpacking bundle from the CRC binary
INFO Checking minimum RAM requirements
INFO Checking if running as non-root
INFO Checking if HyperKit is installed
INFO Checking if crc-driver-hyperkit is installed
INFO Installing crc-machine-hyperkit
INFO Will use root access: change ownership of /Users/rojan/.crc/bin/crc-driver-hyperkit
Password:
INFO Will use root access: set suid for /Users/rojan/.crc/bin/crc-driver-hyperkit
INFO Checking file permissions for /etc/hosts
INFO Checking file permissions for /etc/resolver/testing
Setup is complete, you can now run 'crc start' to start the OpenShift cluster

Started the OpenShift cluster
WARN The cluster might report a degraded or error state. This is expected since several operators have been disabled to lower the resource usage. For more information, please consult the documentation

```

## 步骤 2:启动代码就绪容器

完成设置后，通过运行命令`crc start`或`crc start -p pull-secret.txt`启动 CodeReady 容器。确保您的虚拟专用网络(VPN)会话已关闭。让 VPN 开着会导致启动不正常。下面是我从`crc start`命令得到的输出:

```
$ crc start -p ./pull-secret.txt

INFO Checking if oc binary is cached
INFO Checking if podman remote binary is cached
INFO Checking if goodhosts binary is cached
INFO Checking minimum RAM requirements
INFO Checking if running as non-root
INFO Checking if HyperKit is installed
INFO Checking if crc-driver-hyperkit is installed
INFO Checking file permissions for /etc/hosts
INFO Checking file permissions for /etc/resolver/testing
INFO Extracting bundle: crc_hyperkit_4.5.7.crcbundle ...
crc.qcow2: 9.68 GiB / 9.68 GiB [-------------------------------------------------------------------------------------------------------------------------------------------------------------------] 100.00%
INFO Checking size of the disk image /Users/rojan/.crc/cache/crc_hyperkit_4.5.7/crc.qcow2 ...
INFO Creating CodeReady Containers VM for OpenShift 4.5.7...
INFO CodeReady Containers VM is running
INFO Generating new SSH Key pair ...
INFO Copying kubeconfig file to instance dir ...
INFO Starting network time synchronization in CodeReady Containers VM
INFO Verifying validity of the cluster certificates ...
INFO Network restart not needed
INFO Check internal and public DNS query ...
INFO Check DNS query from host ...
INFO Starting OpenShift kubelet service
INFO Configuring cluster for first start
INFO Adding user's pull secret ...
INFO Updating cluster ID ...
INFO Starting OpenShift cluster ... [waiting 3m]
INFO Updating kubeconfig
INFO
INFO To access the cluster, first set up your environment by following 'crc oc-env' instructions
INFO Then you can access it by running 'oc login -u developer -p developer https://api.crc.testing:6443'
INFO To login as an admin, run 'oc login -u kubeadmin -p ILWgF-VfgcQ-p6mJ4-Jztez https://api.crc.testing:6443'
INFO
INFO You can now run 'crc console' and use these credentials to access the OpenShift web console
Started the OpenShift cluster
WARN The cluster might report a degraded or error state. This is expected since several operators have been disabled to lower the resource usage. For more information, please consult the documentation

```

在继续之前，保存登录凭证供以后使用，或者您可以运行命令`crc console --credentials`以便以后检索它们。

**注**:输出显示集群以降级状态启动。参见 CodeReady 容器文档，了解 CodeReady 容器和生产 OpenShift 集群之间的[差异。](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.11/html/getting_started_guide/introducing-codeready-containers_gsg#about-codeready-containers_gsg)

现在，运行下面的命令以确保您已经在路径中设置了正确的`oc`:

```
$ eval $ (crc oc-env)

```

通过输入命令`crc console`打开集群控制台，验证集群正在运行。使用来自`crc start`命令的凭证以管理员身份登录。您将看到如图 1 所示的对话框。

[![A screenshot of the admin view of the cluster console.](img/309e43fb9a9432ea940f54273f27d819.png "crc-admin-console")](/sites/default/files/blog/2020/09/crc-admin-console.png)

Figure 1: The Administrator's view of the OpenShift cluster console.

## 步骤 3:安装市场必备组件

[本文](https://developers.redhat.com/articles/use-marketplace-add-cockroachdb-operator-openshift/)描述了建立 Red Hat Marketplace 帐户和 pull secret 的过程。注意，您将需要 OpenShift 命令行界面(CLI)和 [jq 命令行 JSON 处理器](https://stedolan.github.io/jq/download/)插件来运行 Marketplace 安装脚本。OpenShift CLI 随 CodeReady 容器安装一起提供。

## 步骤 4:添加一个 CodeReady 容器集群

使用来自`crc start`命令的凭证从命令窗口以管理员身份登录:

```
$ oc login -u kubeadmin -p 8rynV-SeYLc-h8Ij7-YPYcz https://api.crc.testing:6443
Login successful.

You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".

```

在市场门户中，选择**工作区>集群**，点击**添加集群**，如图 2 所示。

[![The Add cluster screen.](img/990aa96f211399a5f8b3859ebb274c9c.png "img_5f51a64a4e027")](/sites/default/files/blog/2020/09/img_5f51a64a4e027.png)

Figure 2: Select 'Add Cluster'.

生成一个拉取密码(或使用先前生成的密码)。在命令窗口中复制并运行安装脚本，将 Marketplace 操作符安装到 CodeReady 容器中:

```
$ curl -sL https://marketplace.redhat.com/provisioning/v1/scripts/install-operator | bash -s -- -i 5eb98eb8995d3a00148c38b5 -p <pull_secret> -a Automatic

```

注意:

*   如果在步骤 1 中生成了密码，则`pull-secret`会自动设置为安装脚本的参数。如果之前创建了`pull-secret`，则显式设置它。
*   出现提示时，输入集群的名称。默认情况下，会生成一个随机字符串作为群集的名称。
*   市场运营商的源代码可在[https://github . com/red hat-market place/red hat-market place-Operator](https://github.com/redhat-marketplace/redhat-marketplace-operator)获得。

以下输出表明您已经成功安装了 Red Hat Marketplace 操作员:

```
==================================================================================
[INFO] Installing Red Hat Marketplace Operator...
==================================================================================
> Cluster Name: 1CF169E2-CA2D-40A4-94EE-99C65C8FBE0B
Edit cluster name for easy reference in Red Hat Marketplace? [Y/n]
y
Enter Cluster Name:
devadv-mac-crc

Detected the following options:
> Account Id: 5e616369cea4170013e06453
> Cluster Name: rhm-mac-crc-45
> Pull Secret: eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1OTg0NDU5NjQsImp0aSI6ImFhM2Q1YmRkYzNlNzRkYThhMGNjYWY4YzM3OWEzYWQyIn0.9Jgw35E4oWfHO5aa6BqZfjFeehEaeGAeQgg-DtpI_yo
> Marketplace Operator Approval Strategy: Automatic
Continue with installation? [Y/n]:

Continue with installation? [Y/n]:
y

STEP 1/5: Validating Namespace
Installing Red Hat Marketplace Operator

STEP 2/5: Installing the Red Hat Marketplace Operator. This might take several minutes
namespace/openshift-redhat-marketplace created
operatorgroup.operators.coreos.com/redhat-marketplace-operator created
secret/rhm-operator-secret created
subscription.operators.coreos.com/redhat-marketplace-operator created
Checking for Cluster Service Version
Checking for Custom Resource Definition

STEP 3/5: Creating Red Hat Marketplace Operator Config custom resource
marketplaceconfig.marketplace.redhat.com/marketplaceconfig created

STEP 4/5: Checking for Razee resources to be created
pod/rhm-watch-keeper-5ddb67c5d7-26fzp condition met
All Razee resources created successfully.

STEP 5/5: Applying global pull secret
W0908 11:28:44.716669 74930 helpers.go:549] --dry-run=true is deprecated (boolean value) and can be replaced with --dry-run=client.
W0908 11:28:44.924940 74933 helpers.go:549] --dry-run=true is deprecated (boolean value) and can be replaced with --dry-run=client.
secret/pull-secret data updated
Applying global pull secret succeeded
Install complete, all resource created.
==================================================================================

Red Hat Marketplace Operator successfully installed.

It may take a few minutes for your cluster to show up in the Marketplace console so you can install purchased software or trials.

Would you like to go back to the Red Hat Marketplace now? [Y/n]
n

```

验证在 CRC 控制台中成功安装了 Marketplace Operator，并且 pod 正在运行，如下所示。

[![](img/720405740e50e6789cc64eebcb2ca39b.png)](https://developers.redhat.com/blog/wp-content/uploads/2020/09/img_5f57a4e6d5797.png)

```
$ oc get pods -n openshift-redhat-marketplace

NAME                                               READY   STATUS    RESTARTS   AGE
prometheus-operator-8447b8b887-pm4nm               2/2     Running   0          10m
redhat-marketplace-operator-f7ff67bcd-mdjwq        1/1     Running   0          11m
rhm-metric-state-65fff5c5f8-9ttdk                  3/3     Running   0          10m
rhm-remoteresources3-controller-5cc6b4945b-lqb6b   1/1     Running   0          10m
rhm-watch-keeper-5ddb67c5d7-26fzp                  1/1     Running   2          10m

```

## 步骤 5:配置提取密码

要在 CodeReady 容器中使用集群全局 pull secret，还需要几个额外的步骤:

1.  运行`oc get secret pull-secret -n openshift-config --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode`并复制输出。
2.  通过运行`oc get node`获得节点名称。
3.  使用`oc debug node/<nodename>`调试节点。
4.  当调试窗格出现时，运行`chroot /host`。
5.  用之前从`oc get secret ..`命令复制的输出替换`/var/lib/kubelet/config.json`中的内容。
6.  输入`exit`退出调试窗格。
7.  运行`crc stop`，然后运行`crc start`。

以下是这些步骤的输出:

```
$ oc get node
NAME STATUS ROLES AGE VERSION
crc-m27h4-master-0 Ready master,worker 9d v1.17.1
$ oc debug node/crc-m27h4-master-0

Starting pod/crc-m27h4-master-0-debug ...
To use host binaries, run `chroot /host`
Pod IP: 192.168.126.11
If you don't see a command prompt, try pressing enter.
sh-4.2# chroot /host
sh-4.4# vi /var/lib/kubelet/config.json
sh-4.4#
sh-4.4# exit
exit
sh-4.2# exit
exit

Removing debug pod ...

```

这就完成了市场中的集群设置。集群启动后，Marketplace 产品即可安装。

## 步骤 6:安装一个 OpenShift 操作符

Red Hat Marketplace 提供 12 个类别的各种产品。举个例子，假设你想试用 [Cortex Certifai](https://marketplace.redhat.com/en-us/products/cortex-certifai) 。

在 CodeReady 容器集群中创建一个项目，并将其命名为`cortex-certifai-test`。

进入[市场目录](https://marketplace.redhat.com/en-us)并搜索 **Cortex Certifai** 。选择正确的图块并点击`Free trial`开始 30 天的试用，如图 3 所示。

[![A screenshot of the Red Hat Marketplace product page for Cortex Certifai.](img/10d55947bf43940a75c8a8cff8d64b1e.png "img_5ef5549daeed8")](/sites/default/files/blog/2020/06/img_5ef5549daeed8.png)

Figure 3: Click 'Free trial' for a 30-day trial.

进入**工作区>我的软件**，点击**安装操作员**图标，如图 4 所示。

[![A screenshot of Cortex Certifai tile with the 'Install Operator' button.](img/63928adeb2ba0fcb4762ce1cf7fcc668.png "img_5ef554ed06d6d")](/sites/default/files/blog/2020/06/img_5ef554ed06d6d.png)

Figure 4: Install the OpenShift Operator for your free trial.

从目标集群列表中选择 CodeReady Containers 集群，然后选择要安装操作符的名称空间范围，如图 5 所示。

[![A screenshot of the 'Target clusters' page in CodeReady Containers.](img/595faa185421bacb5bba3c81d77721bf.png "img_5ef555749be9e")](/sites/default/files/blog/2020/06/img_5ef555749be9e.png)

Figure 5: Select the namespace scope for your cluster installation.

登录到集群，验证操作者是否安装成功，如图 6 所示。

[![A screenshot showing the Cortex Certifai Operator is installed.](img/9459053c0ae4606ff06bd62432dd9dff.png "img_5ef555947c05a")](/sites/default/files/blog/2020/06/img_5ef555947c05a.png)

Figure 6: Check the cluster's 'Installed Operators' page to ensure your Operator is installed.

现在，进入 **Cortex Certifai 操作符**选项卡，安装 Cortex Certifai 操作符的操作数或实例。您可以直接从集群安装 OpenShift 操作符。安装 Marketplace 操作器后，可以在 OperatorHub 中使用 Red Hat Marketplace 操作器。要直接从 CodeReady Containers 安装产品，请登录集群，转到 **Operators > OperatorHub** ，搜索并安装。图 7 显示了安装 Cortex Certifai 的对话框。

[![A screenshot showing the option to install an instance of Cortex Certifai.](img/844434968c19d264739bd2e40c6a2f00.png "img_5ef5562da088d")](/sites/default/files/blog/2020/06/img_5ef5562da088d.png)

Figure 7: Install a product instance or operand.

请注意， **Marketplace** 应该作为过滤参数出现在 OpertorHub 的搜索选项中的**产品类型**下。

## 步骤 6:卸载操作员

从图 8 所示的下拉列表中选择**卸载操作员**卸载 Cortex Certifai 操作员。该集群的运营商将自动从市场运营商页面中除名。

[![A screenshot of the drop-down list showing options to edit or uninstall an installed Operator.](img/26ee812bd5744f0d2a6e24d68a2f2b11.png "img_5ef556752e12e")](/sites/default/files/blog/2020/06/img_5ef556752e12e.png)

Figure 8: Select the 'Uninstall Operator' option.

## 结论

在 CodeReady 容器上安装 Red Hat Marketplace 可以让您在应用程序开发生命周期的早期，灵活地在 CRC 工作站上尝试和测试 Marketplace 产品。作为开发人员，您可以在向您的团队推荐使用认证软件之前，在您的本地开发环境中快速评估该软件。下载 [CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview) 并创建您的 [Red Marketplace](https://marketplace.redhat.com/) 账户开始吧！

你可以在 *[中找到一个更具体的例子，如何使用 Red Hat CodeReady 容器和 Red Hat Marketplace](https://developers.redhat.com/blog/2020/09/09/how-to-install-the-couchbasedb-operator-for-red-hat-openshift-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace/) 在你的笔记本电脑上安装用于 Red Hat OpenShift 的 CouchbaseDB 操作符。*

*Last updated: April 7, 2022*