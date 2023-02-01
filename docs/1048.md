# Red Hat 容器开发工具包 3.6 现已推出

> 原文：<https://developers.redhat.com/blog/2018/10/09/red-hat-container-development-kit-3-6-now-available>

我们很高兴地宣布 [Red Hat 容器开发工具包](https://developers.redhat.com/products/cdk/overview/) (CDK) 3.6 的上市。CDK 3.6 基于 [Minishift](https://www.okd.io/minishift/) 1.24.0，这是一个命令行工具，可以在本地机器上快速提供 [OpenShift](https://www.openshift.com/) 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 集群，用于开发基于云和容器的应用。您可以在 Windows、macOS 或 Linux 上运行 CDK/Minishift。

今天，我们还宣布为 Eclipse 2018-09 提供 [Red Hat Developer Studio 12.9 和 JBoss Tools 4.9。您可以使用熟悉的桌面 IDE 开发基于云/容器的应用程序，该桌面 IDE 集成了用于 CDK/Minishift 的工具。](https://developers.redhat.com/blog/2018/10/09/devstudio-12-9-jboss-tools-4-9-eclipse-2018-09/)

以下是 CDK 3.6 新特性的总结:

## CDK 3.6 的新功能

*   CDK 现在可以运行本地代理服务器来处理代理问题。本地代理服务器帮助在代理环境中使用 CDK。更多信息，请参见[本地代理服务器](https://access.redhat.com/documentation/en-us/red_hat_container_development_kit/3.6/html-single/getting_started_guide/#local-proxy-server)。这个特性是一个技术预览。
*   可用性改进:
    *   CDK 现在将给予`minishift start`命令的非默认标志保存到永久配置中。您不需要显式保存这些设置。要禁用该功能，运行以下命令:

        ```
        $ minishift config set save-start-flags false
        ```

    *   您现在可以使用以下命令使用最新的 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/)版本启动 CDK:

        ```
        $ minishift start --ocp-tag latest
        ```

    *   通过执行允许 OpenShift 在远程[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/)(RHEL)7 机器上启动所需的所有步骤，CDK 的通用驱动程序得到了改进。
    *   通过改进的方式来表达附加组件之间的依赖关系，CDK 附加组件更易于管理。

## 什么是 CDK？

CDK 基于 Red Hat Enterprise Linux，提供了一个预构建的容器开发环境，帮助您使用预配置和本地版本的 OpenShift(业界领先的 Kubernetes 发行版)快速开发基于容器的应用程序。您构建的容器可以轻松地部署在任何 Red Hat 容器主机或平台上，包括 RHEL、Red Hat OpenStack 平台和 OpenShift 容器平台。

## 更多信息

*   参见 [CDK 产品页面](https://developers.redhat.com/products/cdk)。
*   [今天下载](https://developers.redhat.com/products/cdk/download/)新版本。

*Last updated: September 3, 2019*