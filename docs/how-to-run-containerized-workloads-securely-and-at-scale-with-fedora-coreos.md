# 如何使用 Fedora CoreOS 安全、大规模地运行容器化工作负载

> 原文：<https://developers.redhat.com/blog/2020/03/10/how-to-run-containerized-workloads-securely-and-at-scale-with-fedora-coreos>

容器优化操作系统的历史很短，但是有各种各样的提议，取得了不同程度的成功。除了 core OS Container Linux(T1)，Red Hat 还赞助了 T2 项目原子(T3)社区，该社区目前拥有许多项目，从 Fedora/CentOS/Red Hat Enterprise Linux 原子主机到容器工具(Buildah、skopeo 和其他)和 T4 Fedora silver blue(T5)，这是一个不可变的桌面操作系统(在接下来的章节中会详细介绍“不可变”一词)。

2018 年 1 月，当红帽收购总部位于旧金山的公司 CoreOS 时，新的视角出现了。red Hat Enterprise Linux CoreOS(RHCOS)是此次合并的首批产品之一，成为 OpenShift 4 中的基础操作系统。由于 Red Hat 专注于开源软件，总是努力创建和支持上游社区，Fedora 生态系统是 RHCOS 相关上游的自然环境， [Fedora CoreOS](https://getfedora.org/en/coreos/) 。Fedora CoreOS 基于 CoreOS Container Linux 和 Atomic Host 的最佳部分，融合了两者的特性和工具。

在第一篇文章中，我介绍了 Fedora CoreOS，并解释了它对开发人员和 DevOps 专业人员如此重要的原因。在本系列的其余部分，我将深入探讨设置、使用和管理 Fedora CoreOS 的细节。

## Fedora CoreOS

Fedora CoreOS 是一个最小的操作系统，旨在安全和大规模地运行容器化的工作负载(Red Hat CoreOS 也是如此)，这就是为什么 Fedora CoreOS 操作系统层保持尽可能小，并且文件系统作为不可变的映像被原子地管理。这些特性为运行容器提供了可靠的背景。

在 Fedora CoreOS 中，我们可以将应用程序作为容器运行，也可以(可选地)使用 [`rpm-ostree`](https://github.com/projectatomic/rpm-ostree) 工具安装额外的包，这种工具自动地在基础映像上分层更改，类似于我们如何使用 Git commit 来完成我们编写或更新的代码。和 Git 一样，这种行为帮助我们跟踪文件系统的变化。

**注意:**[`rpm-ostree`](https://github.com/projectatomic/rpm-ostree)工具基于 [`libostree`](https://github.com/ostreedev/ostree) 和 [`libdnf`](https://github.com/rpm-software-management/libdnf) 库。它结合了映像系统和包装系统方法的最佳特性来管理和升级机器。

### 什么是“不可变的”，为什么它很重要？

在日常工作中，我们通常在标准的 Linux 系统上运行容器，那么为我们的容器化应用程序使用不可变系统有什么好处呢？我坚信系统管理的未来(不仅仅是在云环境中)指向用声明式方法管理的*不可变基础设施*。但是，等等，到底什么是不可变的基础设施？

不可变的基础结构依赖于其组件在创建后不会被更改。如果必须应用更改(如升级)，则整个组件将被其修改后的新版本替换。考虑一个运行 web 服务器的实例。如果我们需要改变它的配置或者添加/升级/删除模块，并且遵循不可变的方法，我们不会修改正在运行的实例。相反，我们部署一个具有所需更改的新实例，并销毁旧版本。

手动管理系统(或者自动化写得不好)会导致*配置漂移*的风险。我们不需要使用这种方法，我们需要自动管理变更的系统*。在计算机科学中，*原子提交*是将一组不同的改变作为单个操作来应用的操作。不可变的系统是这个场景的自然延伸，给我们一个原子管理的系统，应用所有的改变(升级，新的包，等等)。)在位于基本文件系统之上的单个原子操作中。这种实践产生了更加可预测和可靠的系统。*

 *有些人觉得“不可变”这个术语很奇怪，认为它会削弱系统控制和所有权。这里的不变性与应用机器配置的方式有很大关系，原子方法定义了如何管理文件系统的变化，这赋予了类似 Git 的分层方法特权。这个原子行为的一个示例实现是 [`libostree`](https://github.com/ostreedev/ostree) 库，它是我们将要描述的系统的基础。

Libostree 是一个库和一组工具，它们一起为提交和下载可启动文件系统树提供了一个类似 Git 的模型。Libostree(或简称为`ostree`)在不可变系统之上创建了用于管理`/etc`、用户文件和引导加载程序配置的层，该系统作为一个完整的树被自动管理。因此，我们可以在这些基本的、最小的映像之上运行定制工作负载，方法是在它们之上分层进行定制。

这个过程听起来不像容器吗？

除了容器映像和系统映像之间的相似性之外，在不可变系统上运行容器的主要优势是拥有一个更稳定、标准化和无配置漂移的系统，当在协调的环境中在数十或数百个节点上运行工作负载时，该系统提供了可预测的行为，并减少了手动干预的需要和潜在的相关错误。

### Fedora CoreOS、Red Hat CoreOS 和 OpenShift 4

原子管理系统的可预测性和可靠性是遵循不变方法的系统自动化的完美场景。像 [Terraform](https://www.terraform.io/) 这样的基础设施即代码项目利用了这种管理工作流。我们还可以使用 [Red Hat Ansible](https://www.ansible.com/) 从最小的基础映像开始构建和部署不可变的系统。

OpenShift 4 通过 [OpenShift 操作符](https://www.openshift.com/learn/topics/operators)将智能自动化带到了一个新的高度。运营商按照 *NoOps* 方法承担管理、升级和配置系统的负担，让开发运维专家专注于应用交付。在 OpenShift 4 中，[机器配置操作员](https://github.com/openshift/machine-config-operator) (MCO)有一个基本的角色:它管理集群中的机器配置和更新。MCO 将每个 RHCOS 节点上的机器配置守护程序(MCD)作为守护程序集启动。MCD 检索更新的配置(MachineConfig 资源),并使配置的当前状态与所需状态一致。

由于 Fedora CoreOS 是 OpenShift 4 的上游社区 [OKD](https://www.okd.io/) 4 的默认操作系统，学习 Fedora CoreOS 如何工作对理解 OpenShift 集群内部如何管理节点有很大帮助。

## Fedora CoreOS 入门

本系列的其余部分将使用最新的 Fedora CoreOS [QEMU](https://www.qemu.org/) 映像作为安装、配置和管理的示例。你可以从**裸机&虚拟化**标签的这里[下载这张图片。这些是可以在第一次启动时使用 *Ignition configs* 配置的基本映像，这是一种继承自 CoreOS Linux 的启动配置格式。](https://getfedora.org/en/coreos/download/)

如你所见，QEMU 不是你唯一的选择。您可以在[下载 Fedora CoreOS](https://getfedora.org/en/coreos/download/) 上找到其他图片，分类如下:

*   **裸机:** ISO，PXE 内核和 initramfs，以及 raw
*   **云启动:**全球不同地区 AWS 的 ami
*   **针对云运营商:**阿里云和 OpenStack qcow2、AWS vmdk、Azure vhd、GCP
*   **虚拟化:** OpenStack 和 QEMU qcow2 以及 VMware ova

**注意:**如果您不熟悉 QEMU，请查看 *[用 virt-manager](https://developers.redhat.com/blog/2020/03/06/configure-and-run-a-qemu-based-vm-outside-of-libvirt/) 在 libvirt 之外配置并运行一个基于 QEMU 的 VM。*

现在，让我们浏览一下安装和初始配置 Fedora CoreOS、运行测试容器、更新其配置以及测试新实例的过程。

### 用`fcct`创建点火配置

点火配置的底层技术基于[点火](https://github.com/coreos/ignition/)项目，这是一个低级系统配置实用程序，在机器的 initramfs 中启动时执行。在这个早期引导阶段，Ignition 在旋转持久根文件系统之前应用 Ignition 配置文件中定义的所有配置。在本文中，我们将介绍如何准备一个基本的点火配置文件，然后使用`libvirt`和 QEMU/KVM 在 Linux 系统中引导一个 FCOS 实例。稍加修改，这些例子也适用于云实例。

点火配置是标准的 JSON 文件，没有以漂亮的格式编码。它们可能很长，很难阅读或修改。FCOS 提供了一种被称为 Fedora CoreOS Configuration (FCC)的兼容格式，这是一种 YAML 格式的配置文件，更易于读写。要从 FCC 生成点火文件，我们可以使用 Fedora CoreOS 配置传输器( [FCCT](https://github.com/coreos/fcct) )工具，`fcct`。

这个工具很容易使用，但是我们首先需要创建一个 FCC 文件。对于本文来说，这里有一个简单的例子(`example-fcc.yaml`)，它为用户`core`(FCOS 的默认云用户)设置了一个公共 SSH 密钥:

```
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc...
```

在这个例子中，SSH 公钥是故意不完整的。写完 FCC 文件后，我们需要把它翻译成点火文件。[下载最新发布的`fcct`](https://github.com/coreos/fcct/releases) 并本地安装(`/usr/local/bin`是编译或用户提供的二进制文件的最佳选择)。

现在，运行命令并将 FCC 文件转换为点火配置文件:

```
$ fcct -input example-fcc.yaml -output example-ignition.ign
```

### 引导 Fedora CoreOS

如果您还没有下载 QEMU 映像，现在就下载吧(参见本文前面的说明)。一旦有了图像，用`virt-install`命令启动它:

```
$ sudo virt-install --connect qemu:///system \
  -n fcos -r 2048 --os-variant=fedora31 --import \
  --graphics=none \
  --disk size=10,backing_store=/path/to/fedora-coreos-31.20200118.3.0-qemu.x86_64.qcow2 \
  --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/path/to/example-ignition.ign"
```

请确保用您自己的正确文件位置替换虚拟路径。

在上面的命令中，我向 QEMU 传递了一个命令行参数，以定义启动时使用的带有`--qemu-commandline`选项的点火文件。

**注意:**`virt-install`命令是从命令行启动本地虚拟机的一个很好的工具。如果我们需要一个图形化的替代方法来监控和管理虚拟机，我们可以使用 Gnome 中的`virt-manager`工具并手动配置虚拟机。

如果我们在笔记本电脑上运行 Fedora/CentOS/Red Hat Enterprise Linux 系统，并且在我们的机器上启用了 se Linux(理应如此), SELinux 将阻止实例的创建，因为`qemu-kvm`进程试图在没有`virt_image_t`上下文的情况下访问目录中的文件。为了解决这个问题，我们有两个选择:将 SELinux 置于许可模式，或者重新标记包含点火文件的目录。

要启用许可模式:

```
$ sudo setenforce 0
$ sed -i '/^SELINUX=/ s/enforcing/permissive/g' /etc/selinux/config
```

或者，要更改文件上下文:

```
$ sudo semanage fcontext -a -t virt_image_t '/path/to/ignitions(/.*)?'
$ sudo restorecon -Rvv /path/to/ignitions
```

为了我们的实验室，这两个选项可以互换。现在，让我们引导实例。在快速启动过程结束时，我们应该有如下输出:

```
Fedora CoreOS 31.20200118.3.0
Kernel 5.4.10-200.fc31.x86_64 on an x86_64 (ttyS0)

SSH host key: SHA256:0VrCMwoOmSiU9UNBT/HFzJAPRJFcaR9WE/wpCd3lt2I (ECDSA)
SSH host key: SHA256:YAvgZLN6Wiuo+upzRmcDQ2gIOrJHVSHbiITWhrTRhZo (ED25519)
SSH host key: SHA256:oxT9DOFu+QuOE4jyIJecTdElBvqREllfnCGFYNpIzu4 (RSA)
eth0: 192.168.122.209 fe80::1300:f07a:26f4:2fb2
localhost login:
```

### 第一次登录

除了内核、操作系统版本和 SSH 主机密钥，我们还可以看到分配给以太网接口的 IPv4 地址和链路本地 IPv6 地址。现在，让我们使用 IPv4 地址 SSH 到实例:

```
$ ssh -i /path/to/private_key core@192.168.122.209
Fedora CoreOS 31.20200118.3.0
Tracker: https://github.com/coreos/fedora-coreos-tracker

Last login: Thu Feb  6 21:50:26 2020 from 192.168.122.1
[core@localhost ~]$
```

成功！我们已经登录了我们的 Fedora CoreOS 机器。使用 SSH 密钥登录成功，因为启动时传递的点火文件被正确地应用到了`core`用户的 SSH `authorized_keys`文件。检查修改后的文件，查看我们注入的公钥:

```
[core@localhost ~]$ cat /home/core/.ssh/authorized_keys.d/ignition
```

### 运行测试容器

Fedora CoreOS 附带了已经安装的最常用的容器管理工具。 [Podman](https://podman.io/) 是默认的容器运行时。除了波德曼，[斯科佩奥](https://github.com/containers/skopeo)和 Docker 也被安装，默认情况下 Docker 守护进程被禁用。我个人更喜欢使用 Podman，因为它具有无后台的特性，并且只将 Docker 放在那些必须与其 Unix 套接字进行通信的场景中。

让我们用 Podman 运行一个简单的容器:

```
[core@localhost ~]$ podman run -d -p 8080:80 docker.io/library/nginx
```

在这个例子中，容器端口 80/tcp 被映射到主机端口 8080/tcp，以让 nginx 服务来自外部的请求。

我们可以用`podman ps`命令检查状态:

```
[core@localhost ~]$ podman ps
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS                      NAMES
0abc1a48f176  docker.io/library/nginx:latest  nginx -g daemon o...  11 seconds ago  Up 11 seconds ago  0.0.0.0:8080->80/tcp       dazzling_jackson
```

NGINX 服务器现在已经启动并运行，最重要的是，它作为 Fedora CoreOS 中的*无根*容器运行。从安全性的角度来看，这个结果有很大的影响，因为这意味着容器使用 Linux 中用户名称空间提供的映射。

无根容器是 Podman 在项目早期实现的一个非常重要的特性:为了深入分析，从[无根容器宣言](https://rootlesscontaine.rs/)开始。一定要看看丹·沃什在 Opensource.com 写的这篇[文章](https://opensource.com/article/19/2/how-does-rootless-podman-work)。

## 结论

现在，您已经有了一个在 Fedora CoreOS 中运行的测试容器。[在本系列的下一篇文章](https://developers.redhat.com/blog/2020/03/12/how-to-customize-fedora-coreos-for-dedicated-workloads-with-ostree/)中，我们将跳过安装和设置，重点关注定制和管理。让我们知道你是否受到 Fedora CoreOS 或 Red Hat CoreOS 的启发，以及效果如何！

*Last updated: January 7, 2022**