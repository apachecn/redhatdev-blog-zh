# 将应用程序迁移到 OpenShift，第 1 部分:概述

> 原文：<https://developers.redhat.com/blog/2020/04/07/migrating-applications-to-openshift-part-1-overview>

我帮助团队将他们的应用程序迁移到 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 上，所以我不禁注意到迁移过程中出现的模式和考虑事项。这种操作有许多特定于领域的因素，但是关于在 OpenShift 上启动和运行应用程序，似乎有几个团队用来成功迁移的通用模式。

我发现以下是有用的工作断点:

1.  概念证明。
2.  为单一环境设置持续集成和持续交付(CI/CD)。
3.  为多种环境设置 CD。

在这个系列中，我将分解每个阶段所涉及的工作，并使用来自 [Istio](https://istio.io/) 项目的 [`bookinfo`](https://istio.io/docs/examples/bookinfo/) 应用程序来演示。我选择了`bookinfo`,因为它是一个现有的示例微服务应用程序，我可以从头开始将其部署到 OpenShift 上。此外，`bookinfo`是多语言的(每个微服务都用不同的语言编写)，这允许我通过在一系列编程语言中移植不同类型的服务来展示我的思维过程。

我使用 GitHub 描述的过程从 Istio 分叉出`bookinfo`子目录:[将一个子文件夹拆分成一个新的存储库](https://help.github.com/en/github/using-git/splitting-a-subfolder-out-into-a-new-repository)。你可以在`rh-tstockwell/bookinfo`找到我的叉子。master 分支是从 [`istio/istio`](https://github.com/istio/istio/tree/master/samples/bookinfo) 的直接分叉，而 development 分支则包含了这个系列所做的变更。

## 先决条件

对于本系列，我假设您至少具备初级 OpenShift、Kubernetes 和容器知识。此外，您将需要开发人员访问 OpenShift 集群。如果您没有可用的，您可以免费试用(参见[开始使用 OpenShift](https://www.openshift.com/learn/get-started/) )。

您还需要[安装`oc`客户端](https://docs.openshift.com/container-platform/3.11/cli_reference/get_started_cli.html#installing-the-cli)，并使用标准开发人员权限成功登录到您的集群。如果您遵循了入门页面中的某个指南，您的指南应该会为您提供说明。

在本系列中，我主要使用`oc`客户端(我通常更喜欢命令行)，但是您也可以使用 GUI 进行所有这些更改。

## 概念证明

首先，目标是让您的应用程序快速启动并运行，作为概念验证。这样做的目的是尽可能早地了解你的事业的复杂性，从而尽可能提前降低风险。

我在这个阶段的经验法则是，如果可能的话:

*   尽可能少花时间让你的应用程序工作。
*   使用标准 s2i 图像或模板([在此了解原因](https://github.com/openshift/source-to-image/blob/master/README.md#goals))。
*   避免代码更改。

**注意:**根据经验，在平台迁移期间，Hard Work 很难支持漂移的代码库，所以我们尽量避免这个问题。

在本系列的第 2 部分中，我将展示使用 [`oc new-app`](https://docs.openshift.com/container-platform/3.11/dev_guide/application_lifecycle/new_app.html#using-the-cli) 在 OpenShift 集群上启动并运行`bookinfo`是多么容易。

## 为单一环境设置 CI/CD

第二个目标是能够通过简单的 CI/CD 管道对您的应用程序进行更改。这个管道允许您更快地发展您的应用程序，并且具有更高的可靠性和保证。

在这个阶段，我们将 Kubernetes (k8s)资源导出到[版本控制](https://www.atlassian.com/git/tutorials/what-is-version-control#benefits-of-version-control)，它可以在应用程序旁边，也可以在一个单独的存储库中。然后，我们创建一个简单的 [CI/CD 管道](https://www.redhat.com/en/topics/devops/what-is-ci-cd)，它:

*   运行您定义的测试。
*   按需部署新的应用程序代码。
*   部署应用程序所需的 Kubernetes 资源的更新。

理想情况下，这个过程应该从任何可能的状态开始工作，并到达任何可能的状态。

一旦你有了一个完整的管道，我建议清理掉让你的应用程序工作所产生的任何技术债务。您还应该实现您在概念验证中遗漏的任何东西，您需要这些东西来实现一个工作的、生产就绪的应用程序(从应用程序的角度来看，不一定从运营的角度来看)。我故意推迟处理这些问题，直到我们有一个工作管道，这样我们就可以在实现这些更困难的特性时利用管道的好处。

在本系列的第 3 部分中，我将使用 [OpenShift 和](https://docs.openshift.com/container-platform/3.11/dev_guide/openshift_pipeline.html) [Jenkins](https://docs.openshift.com/container-platform/3.11/dev_guide/openshift_pipeline.html) [管道](https://docs.openshift.com/container-platform/3.11/dev_guide/openshift_pipeline.html)展示一个简单的 CI/CD 管道系列。

## 为多种环境设置连续交付

最后，我们需要重新配置我们在上一步中创建的 CD 管道，以增加部署到多个环境的能力。我喜欢将这一步与前一步分开，因为它通常涉及额外的步骤、配置和 k8s 资源的模板化。您也可以用多种方式实现这一步。

在本系列的第 4 部分中，我将展示如何在没有外部软件的情况下在 Jenkins 中实现一个基本的多环境 CD 管道。

## 进一步的考虑

现在，我还没有尝试涵盖在 OpenShift 上运行应用程序的每一个方面，但是这里有一些我没有涵盖的内容，一旦您已经到达您的旅程的这个阶段，您可能想要研究一下:

*   用 [GitOps](https://blog.openshift.com/introduction-to-gitops-with-openshift/) 管理您的 k8s 资源(您应该已经完成了一部分，因为管道已经从版本控制中获得了 k8s 资源)。
*   使用类似于[哈希公司金库](https://www.vaultproject.io/)的东西来改善你的秘密管理。
*   建立持久的存储/备份/冗余生命周期(取决于您的特定环境和需求)。

*Last updated: January 18, 2022*