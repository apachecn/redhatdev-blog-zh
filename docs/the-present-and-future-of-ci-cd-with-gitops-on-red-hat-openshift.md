# 在 Red Hat OpenShift 上使用 GitOps 的 CI/CD 的现在和未来

> 原文：<https://developers.redhat.com/blog/2020/09/03/the-present-and-future-of-ci-cd-with-gitops-on-red-hat-openshift>

更快地交付应用程序的需求几乎是普遍的，甚至在传统上被视为规避风险的组织中也是如此。作为 [DevOps](https://developers.redhat.com/topics/devops) 的基础，持续集成(CI)和持续交付(CD)对于大多数组织中的应用交付都是必不可少的。总之， [CI/CD](https://developers.redhat.com/topics/ci-cd) 工具和流程在每次代码或配置更改时自动构建和测试应用程序，然后触发一系列工作流，将应用程序交付到生产环境中。

自动化有助于开发人员更快地交付高质量的应用程序，同时减少人为错误，并带来经证实的结果。例如，美国最大的汽车制造商之一福特通过采用 DevOps 流程和 CI/CD 工作流加快了开发流程，[将应用交付时间从数月缩短至数分钟](https://www.redhat.com/en/resources/ford-motor-company-case-study)。

Kubernetes 和 [containers](https://developers.redhat.com/topics/containers) 通过提供启动基础设施和按需部署应用所需的 API 和工具，在减少自动化应用交付障碍方面发挥了重要作用。这种级别的自动化为许多组织开始 DevOps 转型铺平了道路，不仅采用了工具，还采用了随之而来的思维模式和协作文化。作为面向开发人员的 Kubernetes 平台，[Red Hat open shift Container Platform](https://developers.redhat.com/products/openshift/overview)旨在支持开发人员团队采用 CI/CD 实践和自动化应用交付工作流。

在本文中，我们将了解 OpenShift 如何通过集成云原生工具来满足 CI/CD 的未来，这些云原生工具包括 [Tekton](https://www.openshift.com/learn/topics/pipelines) 、 [GitOps](https://developers.redhat.com/devnation/tech-talks/gitops) 和 [Argo CD](https://argoproj.github.io/argo-cd/) 。我们将从当前 CI/CD 环境的概述开始；然后，我将介绍引领我们走向未来的技术。

## OpenShift 上的 CI/CD

CI/CD 环境正在不断扩展，包括更广泛的工具和应用交付方法。利用这些工具为您提供了改进开发运维工作流的机会。由于对遗留和传统工具的现有投资，这也带来了挑战。在 Red Hat，我们通过确保 OpenShift 与现有工具集相集成来支持开发团队的现有投资和工作流。以下是与 OpenShift 集成的内部和基于云的 CI/CD 服务的几个示例:

*   [OpenShift Jenkins 插件](https://www.jenkins.io/doc/pipeline/steps/openshift-pipeline/)简化了针对来自 Jenkins 管道的 OpenShift 的运行操作。
*   [GitHub 的 OpenShift 操作](https://developers.redhat.com/blog/2020/02/13/openshift-actions-deploy-to-red-hat-openshift-directly-from-your-github-repository/)支持直接从 GitHub 存储库中的 GitHub 操作工作流部署到 OpenShift 中。
*   用于 Azure DevOps 的 OpenShift 扩展提供了集成到 Azure 管道中的任务，并在您的 OpenShift 集群上执行操作。
*   [GitLab Runner for OpenShift](https://www.openshift.com/blog/installing-the-gitlab-runner-the-openshift-way) 将 OpenShift 与 GitLab 集成，在您的 OpenShift 集群上运行 GitLab 作业，并将结果报告给 GitLab。

[![An illustration showing how OpenShift integrates technologies for automated builds, traditional CI/CD, and cloud-native CI/CD.](img/2695845067e0e83b1b7020ebca26f5bc.png "CI/CD with GitOps on OpenShift")](/sites/default/files/blog/2020/08/openshift.png)

Figure 1: The present and future of CI/CD on OpenShift.

如图 1 所示，我们还参与了 Kubernetes [开源](https://developers.redhat.com/topics/open-source) CI/CD 生态系统中的当前创新，以支持专注于[云原生开发和现代应用交付](https://developers.redhat.com/blog/2020/08/14/introduction-to-cloud-native-ci-cd-with-tekton-kubecon-europe-2020/)的团队。让我们从我们之前列出的自动化构建、传统 CI/CD 和云原生 CI/CD 技术如何在今天的 OpenShift 中结合在一起开始。

## 自动化构建

开发团队在他们所拥有的应用程序中交付代码变更，以解决业务需求。为应用程序构建容器映像并部署到 Kubernetes 通常不在开发人员的关注范围内，而且会中断他们的工作流程。自从第一个 OpenShift 发布以来，我们已经用 OpenShift 构建服务解决了这个挑战。它提供了在平台上运行的构建和部署自动化，并将应用程序源代码转换成容器映像，然后部署到 OpenShift。对应用程序源代码、配置或运行时更新的每次更改(例如在 [Java](https://developers.redhat.com/topics/enterprise-java) 或 [Node.js](https://developers.redhat.com/blog/category/node-js/) 中)都会导致新的应用程序映像构建，并在此后部署。使用 Dockerfiles 的开发人员可以精确地规划他们希望如何构建他们的应用程序容器映像。喜欢直接指向源代码的开发人员可以使用源到映像(Source-to-Image，S2I)策略，该策略从应用程序源构建一个容器映像。

**注意**:随着 Kubernetes 生态系统中构建工具数量的增长，OpenShift 正在扩展对其他构建工具的支持，如 Kaniko 和 Buildpacks，以及基于[造船厂](https://github.com/shipwright-io)开源项目的下一代 OpenShift 构建功能。

## 传统 CI/CD

持续集成在许多组织中是一种由来已久的实践。应用程序开发团队通常从 Jenkins 开始，自动构建和测试他们的应用程序。Jenkins 是一个灵活的开源 CI 引擎，具有丰富的插件生态系统。

为了支持这种集成，OpenShift 为 Node.js、Java 和[提供了一个 Jenkins 服务器和 Jenkins 代理。NET Core](https://developers.redhat.com/topics/dotnet) 应用。能够在 OpenShift 上轻松运行现有的 Jenkins 管道让团队能够利用他们在 Jenkins 生态系统中的长期投资。此外，OpenShift 上的 Jenkins 附带了一组插件，可以改善 OpenShift 上的 Jenkins 用户体验，并允许您以声明方式配置 Jenkins 服务器。

**注意**:随着团队采用代码配置来实现基础设施部署和配置的可重复性，我们收到了来自 Jenkins 社区和客户的反馈，认为将这些实践应用于配置 Jenkins 服务器本身是一项挑战。虽然 OpenShift 插件有助于缓解这一挑战，但可用的配置只是可用配置的一个子集。

为了解决这个问题并自动化 Jenkins 的运营管理，我们正在与 Jenkins 社区合作开发一个 [Jenkins Operator](https://github.com/jenkinsci/kubernetes-operator) ，它可以作为开发者预览版在 OpenShift 上获得。Jenkins 操作员将操作员模式应用于在 Kubernetes 上安装和管理 Jenkins 服务器。

## 云原生 CI/CD

DevOps 采用率和云原生开发的增加催生了对 CI/CD 技术的需求，这些技术需要适应这些新架构、流程和应用范例的要求。尽管使用 Jenkins 作为已建立的 CI 引擎，但许多团队正在寻找 Kubernetes 本地的替代品，并构建在与他们的应用程序相同的基础设施和抽象之上。

### 泰克顿

Tekton 是一个开源项目，是持续交付基金会的一部分，致力于创建一个在 Kubernetes 上本地运行的 CI/CD 框架。Tekton 提供了使用 Kubernetes 概念以声明方式描述交付管道并在容器中按需执行它们的能力。OpenShift Pipelines 建立在 Tekton 的基础上，提供了一种稳定且受支持的 CI/CD 体验，可无缝集成到 OpenShift 中。通过 OpenShift Pipelines，我们专注于为 Tekton pipelines 提供最佳的用户体验。

图 2 和图 3 显示了 OpenShift 管道和 OpenShift 控制台中的 PetClinic 应用程序示例。

[![A screenshot of PetClinic in OpenShift Pipelines.](img/c17b2e3b66ea1361f6bd131de0954c30.png "OpenShift Pipelines")](/sites/default/files/blog/2020/08/pipeline-viz.png)Figure 2: The PetClinic app in OpenShift Pipelines.">[![A screenshot of PetClinic in the OpenShift console.](img/7846d394a2c4d76636344d526f6b392d.png "Tekton Extension for Visual Studio Code")](/sites/default/files/blog/2020/08/image5.png)

Figure 3: PetClinic in the OpenShift console.

### GitOps

随着 Kubernetes 采用速度的加快和团队向连续交付的过渡，大多数组织都在努力应对保持集群配置同步和跨混合、多集群环境管理应用交付的挑战。虽然基于 CI 向生产环境交付应用很常见，但这种策略缺乏确保生产集群一致性所需的可见性和洞察力。

GitOps 是一种持续交付的方法，也是 DevOps 的一个子集，它是一组越来越受欢迎的实践，作为管理运行混合多集群 Kubernetes 基础架构的复杂性的一种方式。GitOps 支持开发人员将应用程序部署到多个 Kubernetes 集群，同时保证一致性和可预测性。使用 Git 存储库作为事实的来源，GitOps 允许开发人员将已经用于应用程序开发的 Git 工作流应用于基础设施和应用程序操作人员，从而为投入生产的每个更改提供可见性和可追溯性。

### 阿尔戈光盘

Argo CD 是一个声明性的 GitOps 操作符，它使团队能够在 Kubernetes 上实现 GitOps 模式。它通过将应用程序配置同步到目标集群并确保集群处于所需状态，自动将应用程序部署到多个集群。Argo CD 还监视 Kubernetes 集群上部署的应用程序的状态，并通过不断地将它们与 Git 存储库进行比较来提高安全性和可靠性。Argo CD 能够检测和纠正任何偏离所需状态的情况(例如，手动更改的情况)。 [Red Hat 最近加入了 Argo 社区](https://www.redhat.com/en/about/press-releases/red-hat-and-intuit-join-forces-argo-project-extending-gitops-community-innovation-better-manage-multi-cluster-cloud-native-applications-scale)，旨在合作并为使用 Argo CD 在 OpenShift 上实现 GitOps 的开发人员提供集成体验。

## 结论

概括地说，Tekton 侧重于云原生 CI/CD 和交付管道，Argo CD 侧重于可见性、可追溯性、安全性和集成 GitOps 实践。将 Argo CD 与 Tekton 相结合，为组织开始构建和部署自动化之旅以及在混合多集群环境中交付云原生应用提供了坚实的基础。

我们继续扩展 OpenShift 上的 CI/CD 功能，以解决团队在多集群环境中面临的新的复杂挑战。我们正在创建和集成工具，帮助团队开始在他们的组织中建立 CI/CD 和 GitOps 实践。下一个 OpenShift 版本将包括这些更新的预览版。