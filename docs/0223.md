# 为 odo 2.0 开发您自己的定制开发文件

> 原文：<https://developers.redhat.com/blog/2021/02/12/developing-your-own-custom-devfiles-for-odo-2-0>

[Odo 2.0](https://developers.redhat.com/products/odo/overview) 引入了一个名为`devfile.yaml`的配置文件。Odo 使用这个配置文件来设置云原生项目，并确定构建、运行和调试项目等事件所需的操作。如果你是一个 [Eclipse Che](https://www.eclipse.org/che/) 用户，`devfile.yaml`应该听起来很熟悉:Eclipse Che 使用 devfiles 来表达开发人员工作空间，并且它们已经被证明可以灵活地适应各种需求。

Odo 2.0 为各种项目类型提供了一个内置的 devfile 目录，因此您不一定需要编写或修改 dev file 来启动一个新项目。您还可以创建定制的 devfile，并将它们贡献给 odo 的 dev file 目录。本文探索了如何创建一个 devfile 来采用一个现有的开发流程，以便在一个 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 集群上运行。我们的示例项目基于 [Gatsby](https://www.gatsbyjs.com/) ，一个生成网站的框架。Gatsby 自带开发工具和推荐的开发流程，因此它为 Kubernetes 采用现有流程提供了一个很好的例子。

**注意**:请参见 odo 2.0 中的 *[Kubernetes 集成和更多内容，以了解关于这个最新版本中的 devfiles 和其他新功能的更多信息。](https://developers.redhat.com/blog/2020/10/06/kubernetes-integration-and-more-in-odo-2-0/)*

## 设计文件的剖析

在我们开始使用这个例子之前，让我们快速地看一下 devfile 的结构。

除了`schemaVersion`，devfile 上没有其他强制属性。这种灵活性让开发人员可以将 devfiles 用于多种目的。一个开发文件可以是一个技术库的通用文件，也可以是一个项目的专用文件，项目可以继承和覆盖其他开发文件的部分内容。

组件、命令和事件是最常用的 devfile 属性:

*   组件描述了开发环境中需要创建的部分。例子包括运行时容器和 Kubernetes 资源。
*   **命令**描述了使用所提供的工具来实现特定开发目标的预定义命令。
*   **事件**将命令绑定到开发人员环境的生命周期。目前有四项赛事:`postStart`、`postStop`、`preStart`和`preStop`。

## 实现一个开发文件

您已经快速了解了 devfiles 及其三个最常用的属性。现在，让我们应用你所学的。对于这个例子，我们将把来自 [Gatsby 文档](https://www.gatsbyjs.com/docs/)的指令投射到一个 devfile 中，我们将用它来开发 Kubernetes 上的一个网站。

### 选择基础图像

因为应用程序作为容器运行，所以我们将从选择一个基础映像并将其定义为一个组件开始:

```
schemaVersion: 2.0.0

components:

  - name: gatsby

container:

   image: quay.io/eclipse/che-nodejs10-ubi:nightly

   mountSources: true

   memoryLimit: 700Mi

   endpoints:

     - name: web

       targetPort: 8000

```

值得一提的是容器的`mountSources`属性。Odo 使用这个值作为将本地文件同步到 Kubernetes 集群上运行的容器的提示。

### 定义命令

接下来，让我们定义用于构建和运行应用程序的命令。我们需要定义的两个命令将在应用程序的`gatsby`组件上运行。`gatsby-develop`命令在开发模式下启动应用程序。`setup-gatsby-cli`命令在`gatsby`组件上设置 Gatsby 的开发工具:

```
commands:

  - id: gatsby-develop

exec:

   commandLine: "gatsby develop -H 0.0.0.0"

   component: gatsby

   group:

     kind: run

   attributes:

     restart: "false"

  - id: setup-gatsby-cli

exec:

   commandLine: "npm install -g gatsby-cli && npm install"

   component: gatsby

```

### 定义事件

最后，我们定义一个`postStart`事件来优化组件启动后的设置:

```
events:

  postStart:

- setup-gatsby-cli

```

## 创建并推送示例应用程序

假设您已经在本地安装了 odo 和 Gatsby CLI，您可以让新获得的 devfile 工作了。下面是使用 odo 创建和推送一个简单的 Gatsby 站点的命令:

```
gatsby new hello-world https://github.com/gatsbyjs/gatsby-starter-hello-world

cd hello-world

## create or copy devfile.yaml

## from https://gist.github.com/gorkem/78fd17864218a125b2bd9146728a1af8

odo push

```

## 结论

虽然 odo 附带了一个内置的 devfiles 目录，但是您也可以开发自己的目录。创建定制的 devfiles 可以让您将所使用的技术集成到 Kubernetes 环境中。一旦你创建了一个 devfile，你可以把它贡献到 devfile 目录中，以获得更广泛的社区覆盖。

*Last updated: February 9, 2021*