# 如何将 Red Hat OpenShift 虚拟化从硬件虚拟化切换到软件仿真

> 原文：<https://developers.redhat.com/blog/2020/09/17/how-to-switch-red-hat-openshift-virtualization-from-hardware-virtualization-to-software-emulation>

**免责声明** : 以下设置不受 Red Hat 支持，即使是开发/测试/沙盒环境。这只是为了展示技术上的可能性。请参见[为 OpenShift 虚拟化配置您的集群](https://docs.openshift.com/container-platform/4.5/virt/install/preparing-cluster-for-virt.html)了解更多信息。此外，[微型代码生成器(TCG)](https://wiki.qemu.org/Documentation/TCG) 不受 Red Hat 支持或测试。

[OpenShift 虚拟化](https://www.openshift.com/blog/blog-openshift-virtualization-whats-new-with-virtualization-from-red-hat)是[Red Hat open shift Container Platform](https://developers.redhat.com/products/openshift/getting-started)(OCP)和[open shift Kubernetes Engine](https://www.openshift.com/products/kubernetes-engine)的一项功能，允许您在运行和管理容器工作负载的同时运行虚拟机工作负载。基于开源项目 [KubeVirt](https://kubevirt.io/) ，OpenShift 虚拟化的目标是帮助企业从基于 VM 的基础设施转移到基于 Kubernetes 和容器的堆栈，一次一个应用。

[在我之前的文章](https://developers.redhat.com/blog/2020/08/28/enable-openshift-virtualization-on-red-hat-openshift/)中，我向您展示了如何设置和启用在[亚马逊 Web 服务弹性计算云](https://aws.amazon.com/ec2/) (AWS EC2)上运行的 OpenShift 虚拟化。在那篇文章中，我注意到 OpenShift 虚拟化默认寻找硬件虚拟化，这需要一个[裸机服务器](https://en.wikipedia.org/wiki/Bare-metal_server)实例。如果您像我一样在 AWS EC2 上运行 OpenShift，那么您必须在默认的硬件虚拟化上启用软件模拟。否则，您需要来自公共云提供商的裸机实例或纯裸机解决方案。

在本文中，我将向您展示如何将 OpenShift 虚拟化从默认的硬件虚拟化切换到基于 QEMU 的软件仿真。然后，您将能够通过 OpenShift 虚拟化启动和操作虚拟机，即使是在 AWS EC2 等非裸机实例中。

## 硬件虚拟化与软件虚拟化

默认情况下，OpenShift 虚拟化使用 [Linux 内核虚拟机](https://www.linux-kvm.org/page/Main_Page) (KVM)，这是一个完全虚拟化的解决方案，适用于 x86 硬件上的 Linux。KVM 使用虚拟化扩展 Intel VT 或 AMD-V 来实现设备仿真。

出于性能原因，硬件虚拟化(虚拟机直接使用硬件)几乎总是比现实实施中的软件虚拟化更可取。现代 CPU、内存和硬件组件的设计都考虑到了硬件虚拟化。然而，硬件虚拟化并不总是可用的，特别是因为它需要裸机实例(物理服务器)来工作，而不是使用云计算或共享硬件的虚拟服务器所采取的方法。

例如，默认的 AWS EC2 使用 KVM 来处理基于 [AWS Nitro 系统](https://aws.amazon.com/ec2/nitro/#:~:text=The%20AWS%20Nitro%20System%20is,re%2Dimagined%20our%20virtualization%20infrastructure.)的实例类型，但是不公开[嵌套虚拟化](https://www.datto.com/library/what-is-hyper-v-nested-virtualization)，其中一个虚拟化环境在另一个虚拟化环境中运行。除非它是一个纯裸机服务器，否则嵌套虚拟化是一个解决办法，尽管 OpenShift 虚拟化可以使用流行的开源机器仿真器和虚拟器， [QEMU](https://wiki.qemu.org/) 及其[微型代码生成器(TCG)](https://wiki.qemu.org/Documentation/TCG) 在 AWS EC2 中设置这种配置。

如果您尝试在 AWS EC2 之上的 OpenShift 上启动一个虚拟机，并使用 OpenShift 虚拟化进行配置，则在非裸机实例中使用默认硬件仿真将会失败。在这种情况下，当您试图启动新的虚拟机时，您会看到一个 pod 错误，如图 1 所示。

[![VM start fails if the default hardware emulation is used in non-baremetal instance](img/4abd71548b80be411ce7f1c0e2d0572c.png "VM start fails if the default hardware emulation is used in non-baremetal instance")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.14.16-AM.png)VM start fails if the default hardware emulation is used in non-baremetal instance

Figure 1: A virtual machine started with OpenShift Virtualization fails if the default hardware emulation is used in a non-bare-metal instance.

图 1 中的错误消息是，“0/6 节点可用:1 devices.kubevirt.io/vhost-net."不足在 Red Hat OpenShift 中收到这样的错误通常表明您需要调整您的 OpenShift 资源限制，如配额、限制范围或实例节点容量。对于 OpenShift 虚拟化，您必须问的第一个问题是您的 OpenShift 实例*是否运行在裸机服务器上。*

除非您能够在裸机实例上提供 Red Hat OpenShift，否则这种方法只适用于沙盒和测试环境。通过 OpenShift 虚拟化的软件仿真方法运行虚拟机可能会导致性能下降，这可以通过稍后切换回硬件虚拟化来解决。*软件仿真*是通过将代码转换为 CPU 所理解的汇编语言，人工执行“外来”架构和硬件配置文件的过程。幸运的是，KVM 基于流行的开源机器仿真器和虚拟器 QEMU，提供了一种切换到软件仿真的更简单的方法。

**注**:参见[虚拟化和仿真的区别](http://jpc.sourceforge.net/oldsite/Emulation.html)以更好地理解硬件虚拟化和软件仿真的区别。

## 先决条件

通过本文中的例子，您将了解如何将 OpenShift 容器本地虚拟化配置从默认的硬件虚拟化切换到软件仿真。所需要做的就是改变一个属性字段——真的很简单。要遵循这些说明，您必须在开发环境中满足以下先决条件:

*   将 Red Hat OpenShift 4.4 安装在非裸机实例上，如 AWS EC2。
*   在默认名称空间`openshift-cnv`中启用并提供 OpenShift 虚拟化。

**注意**:参见我的获得启用 OpenShift 虚拟化的指南。

## 步骤 1:在 openshift-cnv 中找到 kubevirt-config

我们的第一步是定位名为`kubevirt-config`的`ConfigMap`，我们将使用它将默认的硬件虚拟化机制切换到软件仿真。`kubevirt-config`文件位于默认的 OpenShift 虚拟化名称空间`openshift-cnv`中。

如果您使用的是 OpenShift CLI，使用命令`oc project openshift-cnv`打开 OpenShift 虚拟化项目。然后用`oc get cm`定位`kubevirt-config`，如图 2 所示。

[![ConfigMap "kubevirt-config" can make us to switch to software emulation](img/085cad3952370ce64c11c1d9505c6ac8.png "ConfigMap "kubevirt-config" can make us to switch to software emulation")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.26.44-AM-1.png)ConfigMap "kubevirt-config" can make us to switch to software emulation

另一个选择是使用 OpenShift 的 web 控制台来定位`ConfigMap`。确保您在`openshift-cnv`名称空间中，然后单击**工作负载- >配置图**，如图 3 所示。

[![You can also find kubevirt-config in Openshift web console](img/6826941034b9529046b33a0431cda7fc.png "You can also find kubevirt-config in Openshift web console")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.27.04-AM.png)You can also find kubevirt-config in Openshift web console

点击 **ConfigMap** 查看`kubevirt-config`的 YAML 文件，如图 4 所示。

[![YAML configuration file for CM kubevirt-config in web console](img/afa4a5224801a725a933b1591a1ff4c0.png "YAML configuration file for CM kubevirt-config in web console")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.27.14-AM.png)YAML configuration file for CM kubevirt-config in web console

## 步骤 2:为软件仿真更新 kubevirt-config 文件

一旦您找到了`kubevirt-config`文件，您所要做的就是向该文件添加一个新的属性，指示 OpenShift 虚拟化使用软件仿真而不是硬件虚拟化。如果您正在使用 OpenShift CLI，您可以执行`oc edit cm/kubevirt-config`来编辑配置图，如图 5 所示。

[!["oc edit cm kubevirt-config" to modify the Config Map in CLI](img/fea2ea7b6717e20e39ff24423336f765.png ""oc edit cm kubevirt-config" to modify the Config Map in CLI")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.28.07-AM.png)"oc edit cm kubevirt-config" to modify the Config Map in CLI

一旦进入 ConfigMap 的 YAML 文件，在`data`下添加一个名为`debug.useEmulation`的新属性，然后将值设置为`true`，如图 6 所示。

[![Add "debug.useEmulation" under "data" and set to "true"](img/4e3e524cecddbe92cc6677d11eff500d.png "Add "debug.useEmulation" under "data" and set to "true"")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.28.57-AM.png)Add "debug.useEmulation" under "data" and set to "true"

完成这些更改后，保存文件。仅此而已！CNV 操作器自动获取`ConfigMap`并应用更改。

## 步骤 3:使用软件仿真启动虚拟机

在我们结束之前，让我们测试一下对软件仿真的更改。返回到默认的 CNV 项目名称空间，并创建一个新的虚拟机。您应该不会看到前面的错误；相反，您应该看到 VM 正在启动，如图 7 所示。

[![VM is successfully getting started](img/a7639eca489bcdf6df93accb786a6dcc.png "VM is successfully getting started")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.31.47-AM.png)VM is successfully getting started

Figure 7: Confirmation that the virtual machine is starting.

如图 8 所示，虚拟机的状态将变为`running`。

[![VM is finally up and running](img/78c6053b649b5bd6ad2c09a5606c33c4.png "VM is finally up and running")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.35.37-AM.png)VM is finally up and running

Figure 8: Confirmation that the VM is running.

您还可以通过 OpenShift web 控制台屏幕看到服务器已经成功启动，如图 9 所示。或者，您可以使用 KubeVirt 的 [virtctl](https://docs.openshift.com/container-platform/4.2/cnv/cnv_install/cnv-installing-virtctl.html) 工具(我在上一篇文章中介绍过)连接到 VM。

[![You can monitor the new VM getting started in the web console](img/1de3a67a9964be2ac8b58457558dd773.png "You can monitor the new VM getting started in the web console")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-28-at-11.36.11-AM.png)You can monitor the new VM getting started in the web console

Figure 9: Monitor the new virtual machine in OpenShift's web console.

## 结论

就是这样！我希望这个讨论和简短的指南能够帮助您理解如何从 OpenShift 虚拟化的默认硬件虚拟化转换到基于 QEMU 的软件仿真。通过遵循本文中的说明，您可以轻松地使用 OpenShift 虚拟化操作虚拟机，即使是在 AWS EC2 等非裸机实例中。

*Last updated: October 28, 2020*