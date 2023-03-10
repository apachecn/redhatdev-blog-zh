# 如何在自己的电脑上运行 OpenShift 进行开发？

> 原文：<https://developers.redhat.com/openshift/local-openshift>

要在虚拟机(VM)内部快速启动 OpenShift 集群，可以使用[Red Hat Code ready containers](https://developers.redhat.com/products/codeready-containers)。

CodeReady Containers 是一种在本地机器上尝试或开发 OpenShift 的简单方法。预配置的 OpenShift 集群专为笔记本电脑或台式机开发而定制，可以更轻松地快速使用个人集群。

CodeReady Containers 需要一个 hypervisor 来运行包含 OpenShift 的 VM。根据您的主机操作系统，您可以选择以下虚拟机管理程序:

*   MAC OS:xyve(默认)，VirtualBox
*   GNU/Linux: KVM(默认)，VirtualBox
*   Windows: Hyper-V(默认)，VirtualBox

要下载最新的 CodeReady 容器并查看任何发行说明，请访问 CodeReady 容器页面。在使用 CodeReady 容器之前，请确保查看了[安装说明](https://developers.redhat.com/products/codeready-containers)，以了解您的系统必须满足的任何先决条件。

CodeReady Containers 不是 mini shift——它是一种全新的本地运行 Kubernetes 的方法。

[在 OpenShift 上开发应用](https://developers.redhat.com/openshift)

**红帽 OpenShift 集装箱平台**

【OpenShift 和 Kubernetes 有什么区别？

[有哪些关于 OpenShift 的书籍？](https://developers.redhat.com/openshift/openshift-books/)

在哪里可以试用 OpenShift，看看它是什么样的？

[如何在自己的电脑上运行 OpenShift 进行开发？](https://developers.redhat.com/openshift/local-openshift/)

[有哪些使用 OpenShift 的托管服务？](https://developers.redhat.com/openshift/hosting-openshift/)

*Last updated: November 19, 2020*