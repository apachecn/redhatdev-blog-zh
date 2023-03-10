# 改进了 Azure 上 RHEL 的支持信息:更新了 sosreport 插件

> 原文：<https://developers.redhat.com/blog/2018/11/12/improved-support-information-for-rhel-on-azure-sosreport-plugin-updated>

Red Hat Enterprise Linux 有一个名为`sosreport`的工具，它通过收集系统信息、配置和诊断信息的准确当前细节来提高获得支持的能力。当您的系统出现问题时,“sos”命令是开始调查的最佳切入点。此外，`sosreport`生成的文件通常是 Red Hat 支持工程师调查的起点。即使你不是管理员而是开发人员，`sosreport`也很容易使用。只需使用`sudo`或 root 键入“sos”。

随着 Red Hat Enterprise Linux 系统在 Azure 上运行，一些 Azure 特定的组件是需要的，并且具有重要的作用。例如， [WALinuxAgent](https://github.com/Azure/WALinuxAgent) 是运行在 Azure 上的 RHEL 系统所必需的服务。使用之前版本的 Azure 插件`sosreport`,您可以使用以下命令收集 AzureLinuxAgent 的配置和日志。

```
$ sudo sos -e azure
```

*`-e`选项明确启用了 azure 插件。)

## Sosreport 3.6

`sosreport` Azure 插件从`sosreport` 3.6 开始更新，改进了一些功能。可以通过更新包来试试。

```
$ sudo yum update
```

今天，我想介绍一下新功能。

*   安装 WALinuxAgent 时，Azure 插件会自动启用。现在你不必指定"-e azure "选项来启用这个插件。安装 WALinuxAgent 后，`sosreport`自动启用 Azure 插件。WALinuxAgent 安装在 Azure 上的所有 RHEL 服务器上，因为 WALinuxAgent 是在 Azure 上运行 RHEL 的基本软件包。
*   Azure 实例元数据已收集。 [Azure 实例元数据](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service)帮助收集 Azure 虚拟机的基础信息。不用手动收集。
*   当虚拟机是 Azure Marketplace 虚拟机时，将收集 RHUI 信息。Azure Marketplace RHEL 虚拟机订阅了 [Azure RHUI](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/update-infrastructure-redhat) 。新插件测试 Azure RHUI 的连接并收集客户端证书。

此外，像旧版本一样收集 WALinuxAgent 配置文件和日志。以下是收集到的文件截图。

![](img/cfc1c63aeb57835b6e3eb806e4b13af8.png)

![](img/b3f42e66f0aad8100d242aa18981bf26.png)

请更新`sosreport` 3.6 来尝试这些新功能，并给我任何反馈。您只能通过“yum update sos”更新`sos`包。然而，Red Hat 建议更新您系统中的所有软件包。

*Last updated: November 11, 2018*