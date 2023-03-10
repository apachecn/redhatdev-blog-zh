# 在您的云中快速尝试 Red Hat Process Automation Manager

> 原文：<https://developers.redhat.com/blog/2018/12/04/quickly-try-red-hat-process-automation-manager-in-your-cloud>

自从我上次与您谈论将 [JBoss BPM Suite(现在称为 Red Hat Process Automation Manager)放入您的云](http://www.schabell.org/2016/03/rocking-appdev-in-cloud-jboss-bpmsuite-install-demo.html)已经有一段时间了，随着新版本的发布，是时候再次谈论云中的 AppDev 了。

是时候更新这个故事了，看看如何将[Red Hat Process Automation Manager](https://developers.redhat.com/products/rhpam/overview/)放入您的云中，这样您就可以使用标准配置开始您的第一个业务规则项目。

使用下面描述的简单安装演示项目，您可以通过在任何 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/) 上容器化运行的 business central web 控制台来利用流程自动化工具。

让我们仔细看看这是如何工作的。

### Red Hat 过程自动化管理器易于安装

[![Log in for Red Hat Process Automation Manager on OpenShift Container Platform](img/e5b9e2434fe4b684ed834c23ee4f9325.png "rhcs-rhpam-ocp-console")](/sites/default/files/blog/2018/12/rhcs-rhpam-ocp-console.png)

Log in for Red Hat Process Automation Manager on OpenShift Container Platform

下面是简单安装项目的概要，这是一个让你以最快的方式开始的演示。

这里的目标是让您使用标准配置启动并运行您的第一个业务规则项目。

本节将向您介绍安装示例项目的简单步骤，该示例项目为您提供了 Red Hat Process Automation Manager 的完全可操作的全新安装。

不仅如此，这将是一个容器化的安装，是在您的 OpenShift 安装上创建的！

1.  首先确保您已经安装了基于 OpenShift 容器的安装，例如以下安装之一:
    *   [RHOCP 安装演示](https://gitlab.com/redhatdemocentral/ocp-install-demo)
    *   或者你自己的 OpenShift 安装比如 [Red Hat 容器开发工具包](https://developers.redhat.com/products/cdk/overview/) /minishift。
2.  [下载并解压演示文件。](https://gitlab.com/redhatdemocentral/rhcs-rhpam-install-demo/-/archive/master/rhcs-rhdm-install-demo-master.zip)
3.  将产品添加到`installs`目录。
    [![Watch the container building live on OpenShift Container Platform](img/d10b7f824e9028ed3a6578d7f9e6f0df.png "rhcs-rhpam-build-ocp")](/sites/default/files/blog/2018/12/rhcs-rhpam-build-ocp.png)

    在 OpenShift 容器平台上实时观看正在构建的容器

4.  运行`init.sh`或`init.bat`文件。请注意，`init.bat`文件必须以管理权限运行。
5.  现在，登录 Red Hat Process Automation Manager，开始开发容器化的规则项目(地址将由`init`脚本生成)。OCP 举例:
    `http://rhcs-rhpam-install-demo-appdev-in-cloud.192.168.99.100.nip.io/business-central`(用户名:erics，密码:redhatpam1！)

确保给容器时间不仅启动，而且启动[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/)with Red Hat Process Automation Manager。您可以通过在 OpenShift 控制台中找到已部署的 pod 并查看日志选项卡来检查这一点。

[![The container instance of Red Hat Process Automation Manager on OpenShift Container Platform](img/fe62f5d00dcfa5726e0c9a6f4110a43e.png "rhcs-rhpam-pod-ocp")](/sites/default/files/blog/2018/12/rhcs-rhpam-pod-ocp.png)

The container instance of Red Hat Process Automation Manager on OpenShift Container Platform

就是这样。现在，您可以在闲暇时开始开发流程自动化项目。

请继续关注这里的更新或关注 [Red Hat Demo Central](https://gitlab.com/redhatdemocentral) 的项目。

作为一个额外的，你可以跟随[在线研讨会](http://bpmworkshop.gitlab.io/)和[红帽自动化过程管理器初学者工具包更新](http://www.schabell.org/p/red-hat-process-automation-manager.html)。这里有一个单独的项目，在 T4 本地安装了 Red Hat Process Automation Manager，以防您想要避免容器和云部署。

*Last updated: September 3, 2019*