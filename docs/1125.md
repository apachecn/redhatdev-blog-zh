# 如何在 Fedora 28 上安装红帽 CDK 3.4

> 原文：<https://developers.redhat.com/blog/2018/05/30/how-to-install-red-hat-cdk-3-4-on-fedora-28>

Red Hat Container Development Kit(CDK)提供了一个单节点 Red Hat OpenShift 集群，旨在协助容器化应用程序的开发。这个环境类似于生产 OpenShift 环境，但是它被设计为在单个用户的计算机上工作。为此，CDK 在虚拟机中运行红帽企业 Linux 和红帽 OpenShift 容器平台。

按照以下步骤在 Fedora 28 上安装 CDK 3.4:

1.  设置虚拟化环境。
2.  安装和配置 CDK。
3.  开始 CDK。

以下是执行这些步骤的详细信息。

## 设置虚拟化环境

CDK 需要基于内核的虚拟机(KVM)/ `libvirt`虚拟化技术和 KVM Docker 机器驱动程序插件。执行以下步骤来下载和配置所需的组件。

1.下载 KVM 驱动程序插件并使其可执行:

```
$ sudo curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.7.0/docker-machine-driver-kvm -o /usr/local/bin/docker-machine-driver-kvm
$ sudo chmod +x /usr/local/bin/docker-machine-driver-kvm

```

2.安装 KVM/ `libvirt`:

```
$ sudo dnf install libvirt qemu-kvm

```

3.将您自己添加到`libvirt`群组:

```
$ sudo usermod -a -G libvirt ${USER}

```

4.更新您的用户会话以应用组更改:

```
$ newgrp libvirt

```

5.启动`libvirtd`并将其配置为在引导时启动:

```
$ sudo systemctl start libvirtd
$ sudo systemctl enable libvirtd

```

## 安装和配置 CDK

1.[下载 Linux 版 CDK](https://developers.redhat.com/products/cdk/download/) 。

**注意:**以下步骤假设 CDK 被放置在`~/Downloads`目录中。该文件应该命名为`~/Downloads/cdk-3.4.0-2-minishift-linux-amd64`。

2.创建`~/bin`目录并将 CDK 复制到其中:

```
$ mkdir -p ~/bin
$ cp ~/Downloads/cdk-3.4.0-2-minishift-linux-amd64 ~/bin/minishift
$ chmod +x ~/bin/minishift

```

**注意:**`~/bin`目录应该已经在你的`$PATH`里了。您可以使用您选择的另一个目录，但是我们建议将`minishift`放在您的`$PATH`中。如果这是不可能的，您可以从包含`minishift`的目录中运行它作为`./minishift`。

3.设置 CDK:

```
$ minishift setup-cdk

```

**注意:**这将创建目录`~/.minishift`。该目录包括虚拟机映像和相关的配置文件。

## 开始 CDK

1.您必须使用`minishift`二进制文件启动 CDK。

注册运行 Red Hat Enterprise Linux 的虚拟机:

**注意:**用你用来安装其他 Red Hat Enterprise Linux 系统的凭证替换`$RED_HAT_USERNAME`和`$RED_HAT_PASSWORD`。

```
$ export MINISHIFT_USERNAME="$RED_HAT_USERNAME"
$ export MINISHIFT_PASSWORD="$RED_HAT_PASSWORD"
$ echo "export MINISHIFT_USERNAME=\"$MINISHIFT_USERNAME\"" >> ~/.bashrc
$ echo "export MINISHIFT_PASSWORD=\"$MINISHIFT_PASSWORD\"" >> ~/.bashrc

```

2.开始 CDK:

```
$ minishift start

```

3.验证 CDK 正在运行:

```
$ minishift status

```

恭喜你，CDK 现在运行在你的 Fedora 28 系统上了！

有关使用 CDK 的更多信息，请参见 [CDK 入门指南](https://access.redhat.com/documentation/en-us/red_hat_container_development_kit/3.4/html-single/getting_started_guide/index)。

*Last updated: August 21, 2018*