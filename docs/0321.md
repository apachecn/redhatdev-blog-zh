# odo 2.0 中的 Kubernetes 集成和更多功能

> 原文：<https://developers.redhat.com/blog/2020/10/06/kubernetes-integration-and-more-in-odo-2-0>

Odo 是 OpenShift 和 Kubernetes 的一个面向开发人员的命令行界面 (CLI)。这篇文章介绍了 odo 2.0 版本的亮点，它现在集成了 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 。其他亮点包括 odo 2.0 中新的默认部署方法，它使用 devfiles 进行快速迭代开发。我们还将[操作员部署](https://developers.redhat.com/topics/kubernetes/operators)移出了实验模式，因此您可以从`odo`命令行轻松部署操作员支持的服务。

## 奥多 2.0 现在可以和核心库本内特一起使用了！

Odo 2.0 允许您完全在 Kubernetes 上编写、构建和部署应用程序。您可以使用任何兼容的 Kubernetes 集群，无论是托管的云提供商、自我管理的集群，还是使用 Minikube 等工具在本地托管的集群。

Odo 与 Kubernetes 的集成提供了一致的开发体验。您可以从头开始编写应用程序，迭代[开发内部循环](https://developers.redhat.com/devnation/tech-talks/odo-iterative-container-based-development)，并将代码提交给 Git，所有这些都在同一个环境中。

要启动 Kubernetes 安装，请安装 Kubernetes [操作员生命周期管理器](https://github.com/operator-framework/operator-lifecycle-manager)和`etcd`。参见 Kubernetes 操作员中心的 [etcd 安装指南](https://operatorhub.io/operator/etcd)。

## 在 odo 2.0 中使用 devfiles 进行部署

这个主要版本将 devfiles 作为 odo 的默认部署方法。对于喜欢从命令行使用`--s2i`标志的开发人员，Odo 仍然支持源到映像(S2I)部署。

一个 *devfile* 是一个 YAML 文件，用于定义 [Eclipse Che](https://developers.redhat.com/videos/youtube/S3auoOqwDS8) 中的开发人员工作区。Devfiles 有一个开放的格式，所以我们也可以在`odo`中使用它们。Odo 对 devfiles 的支持让开发人员可以轻松地在工具之间切换，无需额外的配置。使用 devfiles 还简化了向`odo`和 Eclipse Che 添加新语言支持的过程。现在，您只需要从模板创建一个 devfile 并更新。

参见 [odo 教程](https://odo.dev/docs/deploying-a-devfile-using-odo/)获取在`odo`中部署你的第一个 devfile 的指南。

### 开发人员工具的通用定义

随着 odo 2.0 的发布，我们已经将 devfiles 作为开发人员工作空间和应用程序生命周期的通用定义格式，贯穿于 Red Hat 的开发人员工具组合。[Red Hat code ready work spaces](https://developers.redhat.com/products/codeready-workspaces/overview)(Eclispe Che 的产品化版本)目前使用 devfiles，所有的 OpenShift IDE 扩展都利用了`odo`，直接为开发者带来迭代开发和部署流程。你可以直接试用`odo`，或者使用 [VS 代码](https://developers.redhat.com/products/vscode-extensions/overview)、 [Eclipse Che](https://www.eclipse.org/che/) 和 [Eclipse 桌面 IDE](https://www.eclipse.org/ide/) 的 IDE 扩展来访问它。

### 改进的语言支持

添加 devfiles 作为缺省部署方法改进了 odo 2.0 中的语言支持。要查看当前支持的 devfile 组件列表，请打开您的`odo` CLI 并运行:

```
$ odo catalog list components

```

表 1 显示了当前可用的`odo`组件，包括 devfile 组件。

Table 1: Odo devfile components

| **名称** | **描述** | **注册表** |
| 爪哇胃 | 上游 Maven 和 OpenJDK 11 | DefaultDevfileRegistry |
| java-openliberty | 在 Java 中打开 Liberty 微服务 | DefaultDevfileRegistry |
| 爪哇-夸尔库斯 | 使用 Java+GraalVM 的上游 Quarkus | DefaultDevfileRegistry |
| 爪哇跳羚 | 使用 Java 的 Spring Boot | DefaultDevfileRegistry |
| nodejs | 带节点的堆栈 12 | DefaultDevfileRegistry |

Odo 的新部署模型可用于使用 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 、 [Node.js](https://developers.redhat.com/blog/category/node-js/) 的 [Java](https://developers.redhat.com/topics/enterprise-java) ，以及用于 [Python](https://developers.redhat.com/blog/category/python/) 的早期访问。

### 新项目的样本启动器

使用 devfiles 的另一个优点是，您现在可以利用样例初学者来搭建新项目。只需使用`odo create`命令提供 devfile 组件的名称。Odo 将从相关的 Git 存储库中提取一个启动器的克隆本地副本。这里有一个例子:

```
$ odo create nodejs --starter

Validation

 ✓  Checking devfile existence [22411ns]

 ✓  Checking devfile compatibility [22492ns]

 ✓  Creating a devfile component from registry: DefaultDevfileRegistry [24341ns]

 ✓  Validating devfile component [74471ns]

Starter Project

 ✓  Downloading starter project nodejs-starter from https://github.com/odo-devfiles/nodejs-ex.git [479ms]

Please use `odo push` command to create the component with source deployed

```

## 使用 odo 调试

在这个版本中，`odo debug`命令已经脱离了技术预览版。关于用 odo CLI 或 VS 代码调试应用程序组件的更多信息，请参见 odo 教程。

## 使用操作符进行安装

开发者现在可以用`odo`部署运营商支持的服务。运算符提供自定义资源定义(CRD)，您可以使用它来创建服务实例，也称为自定义资源(CRs)或操作数。然后，您可以在项目中使用这些实例，并将它们链接到您的组件。

下面是一个使用`etcd`操作符部署 Etcd 集群的例子:

```
$ odo catalog list services

  Operators available in the cluster

  NAME                          CRDs

  etcdoperator.v0.9.4           EtcdCluster, EtcdBackup, EtcdRestore

$ odo service create etcdoperator.v0.9.4/EtcdCluster

```

关于使用 odo 部署运营商支持的服务的更多信息，请参见 odo 教程。

*Last updated: October 7, 2020*