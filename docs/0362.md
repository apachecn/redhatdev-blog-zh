# Red Hat CodeReady Workspaces 2.3 中改进的配置和更多功能

> 原文：<https://developers.redhat.com/blog/2020/08/21/improved-configuration-and-more-in-red-hat-codeready-workspaces-2-3>

基于 [Eclipse Che](https://www.eclipse.org/che/getting-started/cloud/?sc_cid=701f2000000RtqCAAS) ，[红帽 CodeReady Workspaces](https://developers.redhat.com/products/codeready-workspaces/overview) (CRW)是一个[红帽 open shift](https://developers.redhat.com/openshift/)-原生开发者环境，支持云原生开发。CodeReady Workspaces 2.3 现已推出。对于这个版本，我们专注于改进 CRW 的配置选项，更新到 IDE 插件的最新版本，并添加新的开发文件。

CodeReady Workspaces 2.3 可从以下网站获得:

*   [OpenShift 3.11](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.0/html/installation_guide/installing-codeready-workspaces-on-openshift-3-using-the-operator_crw) 和 [OpenShift 4.3](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.3/html/installation_guide/installing_codeready_workspaces_on_openshift_container_platform) 及以上，包括 [OpenShift 4.5](https://developers.redhat.com/products/openshift/getting-started) 。
*   [OpenShift 专用](https://www.openshift.com/products/dedicated/) 4.3，通过附加功能。

## 将秘密注入工作空间的两种方法

从 CodeReady Workspaces 2.3 开始，您可以自动将加密的 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 秘密挂载到 workspace [容器](https://developers.redhat.com/topics/containers/)中，这些秘密包含诸如用户名、密码和认证令牌等敏感信息。要挂载这些秘密，您必须在 OpenShift 名称空间中创建它们，在这里创建您的工作空间容器。让我们考虑将机密装载到他们的工作空间的两种机制:将机密装载为环境变量，以及将机密装载到文件中。

### 作为环境变量的越来越多的秘密

下面的 YAML 文件包含两个秘密，它们与环境变量`FIRST_ENV_VAR`和`SECOND_ENV_VAR`相关联。这些环境变量分别设置有`myvalue1`和`myvalue2`的值。一旦设置好，这些变量将驻留在所有工作区中一个名为`maven`的容器内，这些工作区是在与这个 YAML 文件相同的 OpenShift 名称空间中创建的:

```
apiVersion: v1
kind: Secret
metadata:
  name: mvn-settings-secret
  annotations:
    che.eclipse.org/target-container: maven
    che.eclipse.org/mount-as: env
    che.eclipse.org/firstkey_env-name: FIRST_ENV_VAR
    che.eclipse.org/secondkey_env-name: SECOND_ENV_VAR
  labels:
   …
data:
  firstkey: myvalue1
  secondkey: myvalue2

```

### 在文件中增加秘密

下一个样本文件展示了一个秘密，它被安装在一个名为`settings.xml`的文件中。该文件可用于所有工作区中名为`maven`的容器，这些工作区是在与该 YAML 文件相同的 OpenShift 名称空间中创建的:

```
apiVersion: v1
kind: Secret
metadata:
  name: mvn-settings-secret
  labels:
    app.kubernetes.io/part-of: che.eclipse.org
    app.kubernetes.io/component: workspace-secret
  annotations:
    che.eclipse.org/target-container: maven
    che.eclipse.org/mount-path: /home/user/.m2/
    che.eclipse.org/mount-as: file
    che.eclipse.org/automount-workspace-secret: true

data:
  settings.xml: __<base64 encoded data content here>__

```

## OpenShift 集群范围的代理支持

当您使用集群范围的出口代理启用 OpenShift 时，CodeReady Workspaces 现在会自动支持该代理在其组件和外部服务之间进行通信。

## 实验工作空间存储设置

现在，您可以使用实验值`async`设置 CheCluster 自定义资源配置属性`CHE_WORKSPACE_STORAGE_AVAILABLE_TYPES`。该值混合了短暂存储和持久存储。它允许更快的 I/O，并保留工作区更改。

## 提高终端连接的可靠性

我们改进了 CodeReady 工作区，以防止由于终端连接的处理而导致任务执行失败。

## 设计文件更新

我们更新了随 CodeReady 工作区提供的 devfiles 集，用于运行时映像和堆栈的最新版本。

寻找一个新的 devfile，带有[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/overview)(JBoss EAP)XP 1.0，OpenJDK 11，Maven 3.5。只要有可能，我们就将基于 Java 的开发文件迁移到 JDK 11。(除非需要，否则 JBoss EAP 不包括在内。)我们还用数字更新了 devfiles 中的命令，这些数字反映了建议的命令执行顺序。

## IDE 插件更新

我们更新了 CodeReady Workspaces 提供的各种 IDE 插件的新版本。用于 [Java](https://developers.redhat.com/topics/enterprise-java) 语言支持、[应用依赖分析](https://marketplace.visualstudio.com/items?itemName=redhat.fabric8-analytics)、项目初始化器、 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 语言支持和 Sonarlint 的插件都已更新。

## 获取 CodeReady 工作区 2.3

CodeReady Workspaces 2.3 现已在 OpenShift 3.11 和 OpenShift 4.x 上推出:

*   如果你使用的是 OpenShift 3.11，你可以[在这里](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.0/html/installation_guide/installing-codeready-workspaces-on-openshift-3-using-the-operator_crw)找到安装说明。
*   如果你使用的是 OpenShift 4.x，你可以直接从 OpenShift OperatorHub 安装[遵循这里的文档](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.3/html/installation_guide/installing_codeready_workspaces_on_openshift_container_platform)。
*   [下载](https://developers.redhat.com/products/codeready-workspaces/download)code ready work spaces 命令行界面。
*   查看 CodeReady Workspaces 产品页面。

*Last updated: August 20, 2020*