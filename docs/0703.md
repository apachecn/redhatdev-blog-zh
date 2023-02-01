# 使用 Nodeshift 将 Node.js 应用程序轻松部署到 Red Hat OpenShift

> 原文：<https://developers.redhat.com/blog/2019/08/30/easily-deploy-node-js-applications-to-red-hat-openshift-using-nodeshift>

我最近写了一些文章，关于如何在 OpenShift 上部署一个 Express.js 应用程序，以及如何使用 Chrome 开发工具在 OpenShift 上调试 Node.js 应用程序，还有一个简短的系列文章，关于如何在 OpenShift 上部署现代 web 应用程序，分别是关于如何在 open shift 上部署 express . js 应用程序的。所有这些文章都使用了一个名为 [Nodeshift](https://www.npmjs.com/package/nodeshift) 的节点模块，但是我在谈论它的时候做了一个绝地反击。下一系列文章将深入探讨什么是 Nodeshift，以及在开发过程中如何使用它来简化 Node.js 应用程序到 OpenShift 的部署。

## Red Hat OpenShift 上的基本应用部署

尽管有不同的方法来部署一个应用到 Red Hat OpenShift 上，我们将会看到我喜欢使用的工作流程。这个特定的工作流程使用源到图像(S2I)图像和位于我的本地机器上的源代码。不过，在我们了解 Nodeshift 之前，让我们先快速了解一下这个工作流使用的一些部分。这个流程在逻辑上可以分为两个部分:T2 构建阶段和 T4 部署阶段。

### 第 1 部分:构建阶段

这个工作流的第一阶段是构建一个映像，最终在部署阶段运行。对于我们的 Node.js 应用程序，这是我们安装依赖项和运行任何构建脚本的阶段。如果您熟悉 S2I 的各个阶段，这个阶段就是汇编脚本运行的地方。

使用 BuildConfig，我们可以指定我们的代码来自哪里，以及在构建代码时使用什么类型的策略。在我们的例子中，我们使用 DockerImage 策略，因为我们使用的是 Node.js S2I 映像。BuildConfig 还告诉 OpenShift 在完成后将我们构建的代码放在哪里:在我们的例子中，是一个 ImageStream。

最初，我们创建一个空的 ImageStream，然后用成功构建的结果填充它。事实上，如果您查看 OpenShift 的内部映像注册表，您会在那里看到该映像，类似于您在本地机器上运行类似于`docker images`的东西时看到的容器映像。

### 第 2 部分:部署阶段

这个工作流程的第二阶段是运行我们的应用程序，并设置它以供访问。对于我们的 Node.js 应用程序，这是我们可能运行类似于`npm run start`的东西来启动我们的应用程序的阶段。同样，如果您熟悉 S2I 的各个阶段，那么这个阶段就是运行脚本的地方。默认情况下，Node.js S2I 图像与我们在这里使用的命令相同:`npm run start`。

使用 DeploymentConfig，我们可以触发 S2I 运行阶段。DeploymentConfigs 也用于描述我们的应用程序(使用什么图像流、任何环境变量、设置健康检查等等)。一旦部署成功，就会创建一个正在运行的 Pod。

接下来，我们需要一个用于新 Pod 内部负载平衡的服务，如果我们想在 OpenShift 上下文之外访问我们的应用程序，还需要一个路由。

虽然这个工作流并不太复杂，但是有许多不同的部分可以协同工作。这些文件也是 YAML 的档案，有时很难阅读和解释。

## 节点移位基础

现在我们已经有了一些关于将应用程序部署到 OpenShift 的背景知识，让我们来谈谈 Nodeshift 以及它是什么。根据 Nodeshift 模块的自述文件:

> Nodeshift 是一个固执己见的命令行应用程序和可编程 API，您可以使用它将 Node.js 项目部署到 OpenShift。

Nodeshift 的观点是我刚刚描述的工作流，它允许用户开发他们的应用程序并将其部署到 OpenShift，而不必考虑所有这些不同的 YAML 文件。

Nodeshift 也是用 Node.js 编写的，所以它可以适合节点开发人员的当前工作流，或者使用`npm install`添加到现有项目中。唯一真正的先决条件是您使用`oc login`登录到您的 OpenShift 集群，但这并不是真正的要求。您还可以指定一个外部配置文件，我们将在后面关于更高级用法的文章中看到。

### 运行节点转移

在命令行上使用 Nodeshift 很容易。您可以全局安装它:

```
$ npm install -g nodeshift

$ nodeshift --help

```

或者使用`[npx](https://www.npmjs.com/package/npx)`，这是首选方式:

```
$ npx nodeshift --help

```

与其他命令行工具一样，使用那个`--help`标志运行 Nodeshift 向我们展示了可以使用的命令和标志:

```
Commands:
  nodeshift deploy                default command - deploy             [default]
  nodeshift build                 build command
  nodeshift resource              resource command
  nodeshift apply-resource        apply resource command
  nodeshift undeploy [removeAll]  undeploy resources

Options:
  --help                   Show help                                   [boolean]
  --version                Show version number                         [boolean]
  --projectLocation        change the default location of the project   [string]
  --configLocation         change the default location of the config    [string]
  --dockerImage            the s2i image to use, defaults to
                           nodeshift/centos7-s2i-nodejs                 [string]
  --imageTag               The tag of the docker image to use for the deployed
                           application.             [string] [default: "latest"]
  --outputImageStream      The name of the ImageStream to output to.  Defaults
                           to project name from package.json            [string]
  --outputImageStreamTag   The tag of the ImageStream to output to.     [string]
  --quiet                  supress INFO and TRACE lines from output logs
                                                                       [boolean]
  --expose                 flag to create a default Route and expose the default
                           service
                               [boolean] [choices: true, false] [default: false]
  --namespace.displayName  flag to specify the project namespace display name to
                           build/deploy into.  Overwrites any namespace settings
                           in your OpenShift or Kubernetes configuration files
                                                                        [string]
  --namespace.create       flag to create the namespace if it does not exist.
                           Only applicable for the build and deploy command.
                           Must be used with namespace.name            [boolean]
  --namespace.remove       flag to remove the user created namespace.  Only
                           applicable for the undeploy command.  Must be used
                           with namespace.name                         [boolean]
  --namespace.name         flag to specify the project namespace name to
                           build/deploy into.  Overwrites any namespace settings
                           in your OpenShift or Kubernetes configuration files
                                                                        [string]
  --deploy.port            flag to update the default ports on the resource
                           files. Defaults to 8080               [default: 8080]
  --build.recreate         flag to recreate a buildConfig or Imagestream
           [choices: "buildConfig", "imageStream", false, true] [default: false]
  --build.forcePull        flag to make your BuildConfig always pull a new image
                           from dockerhub or not
                               [boolean] [choices: true, false] [default: false]
  --build.incremental      flag to perform incremental builds, which means it
                           reuses artifacts from previously-built images
                               [boolean] [choices: true, false] [default: false]
  --metadata.out           determines what should be done with the response
                           metadata from OpenShift
        [string] [choices: "stdout", "ignore", ""] [default: "ignore"]
  --cmd                                                      [default: "deploy"]

```

我们来看看最常见的用法。

### 部署节点转移

假设我们在本地开发了一个简单的 express.js 应用程序，我们已经将它绑定到端口 8080，我们希望将这个应用程序部署到 OpenShift。我们只要跑:

```
  $ npx nodeshift

```

一旦该命令运行，Nodeshift 就开始工作。下面是该命令使用默认部署命令所经历的步骤:

1.  Nodeshift 将您的源代码打包成一个`tar`文件，上传到 OpenShift 集群。
2.  Nodeshift 查看应用程序的`package.json`的`files`属性(默认情况下，它忽略任何`node_modules`、`tmp`或`.git`文件夹):
    *   如果存在一个`files`属性，Nodeshift 使用`tar`来归档这些文件。
    *   如果没有`files`属性，Nodeshift 归档当前目录。
3.  创建归档文件后，将在远程集群上创建新的 BuildConfig 和 ImageStream。
4.  存档文件已上传。
5.  OpenShift 构建开始在 OpenShift 上运行。
6.  Nodeshift 监视构建过程，并将远程日志输出到控制台。
7.  一旦构建完成，Nodeshift 就会创建一个 DeploymentConfig，这将触发一个实际的部署和一个 Kubernetes 服务。(默认情况下不会创建路线，但是如果需要，您可以使用`--expose`标志。)

如果您修改代码并再次运行`nodeshift`命令，这个过程会再次发生，但是这次它使用第一次运行时创建的现有配置文件。

## 直到下次

在本文中，我们通过一个简单的例子分析了 Red Hat OpenShift 部署的结构，以及 Nodeshift 如何帮助抽象复杂性。请继续关注未来的文章，在这些文章中，我们将研究 Nodeshift 提供的其他命令。在那些文章中，我们将探索几个常用的选项，并展示如何在我们的代码中使用 Nodeshift，而不仅仅是在命令行中使用它。

*Last updated: August 29, 2019*