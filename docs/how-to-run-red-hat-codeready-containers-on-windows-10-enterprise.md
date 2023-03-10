# 如何在 Windows 10 企业版上运行 Red Hat CodeReady 容器

> 原文：<https://developers.redhat.com/blog/2020/09/09/how-to-run-red-hat-codeready-containers-on-windows-10-enterprise>

[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)允许你在你的本地 PC 上启动一个小型 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 集群，而不需要服务器、云或者一组操作人员。对于想要立即开始云原生开发、[容器](https://developers.redhat.com/topics/containers/)和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) (以及 OpenShift)的开发人员来说，这是一个简单而灵活的工具。它可以在 macOS、 [Linux](https://developers.redhat.com/topics/linux/) 和所有版本的 [Windows](https://developers.redhat.com/blog/category/windows/) 10 上运行。

Windows 10 企业版除外。

我痛苦地认识到。

因为我懒的没注意文档。

好吧，所以我是唯一一个掩饰文档的开发者。幸运的是，我努力设法让 CRC 在我的 Windows 10 Enterprise 笔记本电脑上运行，这篇文章解释了让它工作的相关内容。所以，某种意义上，不客气的说我懒。

## 它就在你面前

原来， [CRC 安装网页](https://developers.redhat.com/download-manager/link/3868678)明确告诉我们支持哪些版本的 Windows 10，而 Windows 10 Enterprise(以及 Windows Server)并没有明确提到是其中之一(见图 1)。

[![screenshot of the supported Windows versions](img/b0cb36ee6e7da282b36fea11b95a00c4.png "crc-windows-versions-supported")](/sites/default/files/blog/2020/05/crc-windows-versions-supported.png)

Figure 1: Do you see Windows 10 Enterprise or Server? I can't either.

我的想法是，如果它支持家庭和专业版，那么显然会支持像 Windows 10 Enterprise 这样的更“企业”的版本。我的意思是，它必须有所有的小版本，再加上更多，对不对？

事实证明，这才是根本问题。

## 一些历史

Windows 98 为 Windows 引入了 Internet 连接共享(ICS ),例如，使网络用户更容易使用共享的拨号调制解调器(还记得那些吗？)那项技术被取代了，取而代之的是在 Windows 10 中作为服务运行的东西——除了企业版。

企业版*也*使用了完全重写的网络连接堆栈，这是为服务器(而不是工作站)构建的，但有一个例外:在 Windows 10 企业版的 Hyper-V 管理器中，截至本文撰写之时，*默认交换机*仍使用旧的 ICS 技术。

## 代码就绪容器行为

CRC 在您的 PC 的 Windows 10 机器上寻找并使用默认开关，这意味着它可以很好地与 Windows 10 家庭和专业版配合使用。但当 CRC 试图在 Windows 10 企业版中使用默认开关时，它使用的是不兼容的旧(ICS)技术。

那你是做什么的？

## 使用源，卢克

或者莎拉，或者随便你叫什么。重点是:因为 CRC 是开源的，所以您可以检查源代码，看看程序如何使用开关进行 DNS 查找。瞧，就在 [Go](https://developers.redhat.com/blog/category/go/) 源代码中，我们可以看到一个替代方案得到了支持，如图 2 所示。

[![screenshot of the Go code showing an alternate method](img/6629ff370f5cd9bb722a508a067889a4.png "crc-windows-network-source-code")](/sites/default/files/blog/2020/05/crc-windows-network-source-code.png)

图 2:解决方案就在那里。">

没错。如果我们创建一个名为`crc`的开关，它将被用来代替默认开关。

## 对此极度兴奋

从这里开始就很简单了。我们打开 Hyper-V 管理器并创建一个名为`crc`的新外部交换机，如图 3 所示。

[![screenshot of the virutal switch manager dialog](img/3842baeffc4dc553680ee7c59153116e.png "crc-crc-network-switch-creation")](/sites/default/files/blog/2020/05/crc-crc-network-switch-creation.png)

Figure 3: Creating the External switch.

## 点燃它

剩下的唯一一步是启动 CRC，如图 4 所示:

```
$ crc start -p ~/Downloads/pull-secret.txt
```

[![animation of the start process](img/49af1acdba95b028d0a2930199ec2a64.png "crc_start_windows_10_enterprise")](/sites/default/files/blog/2020/05/crc_start_windows_10_enterprise.gif)

Figure 4: Look at that, it works!

一旦你启动并运行了 CRC，访问 [Red Hat Marketplace](https://marketplace.redhat.com/) 并下载一些很棒的软件来运行。他们大多提供免费试用；你有机会在 OpenShift 中安装一个事件引擎，看看它有多容易使用。

所以你有它。一个小小的改变，你的 Windows 10 企业版电脑就可以运行 CodeReady 容器了。虽然这个设置没有得到官方支持，但我没有遇到任何问题。请在评论中告诉我你是否也是这样。

从这里，您可能想要检查:

*   *[使用 Red Hat CodeReady 容器和 Red Hat Marketplace](https://developers.redhat.com/blog/2020/09/09/install-red-hat-openshift-operators-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace/)* 在您的笔记本电脑上安装 Red Hat OpenShift 操作器
*   *[如何使用 Red Hat CodeReady 容器和 Red Hat Marketplace](https://developers.redhat.com/blog/2020/09/09/how-to-install-the-couchbasedb-operator-for-red-hat-openshift-on-your-laptop-using-red-hat-codeready-containers-and-red-hat-marketplace/)* 在您的笔记本电脑上安装用于 Red Hat OpenShift 的 CouchbaseDB 操作符

*Last updated: April 7, 2022*