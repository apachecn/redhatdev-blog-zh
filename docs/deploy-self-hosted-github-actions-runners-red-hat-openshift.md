# 为 Red Hat OpenShift 部署自托管 GitHub 操作运行程序

> 原文：<https://developers.redhat.com/articles/2021/06/16/deploy-self-hosted-github-actions-runners-red-hat-openshift>

**注意**:这篇文章是关于 Red Hat 在 GitHub Actions 生态系统中的工作的更新。如果你还没有读过[12 月份的新闻稿](https://www.redhat.com/en/about/press-releases/red-hat-and-github-collaborate-expand-developer-experience-red-hat-openshift-github-actions)或者[最初的 openshift.com 博客文章](https://www.openshift.com/blog/deploying-to-openshift-using-github-actions)，一定要去看看它们的完整内容。

迄今为止，红帽已经发布了一系列的 [GitHub 动作](https://github.com/redhat-actions/)和一个 OpenShift starter 工作流，以方便将[红帽 OpenShift](/products/openshift/overview) 与 GitHub 广受欢迎的 [CI/CD](/topics/ci-cd/) 平台集成。自最初发布以来，我们一直在努力回应社区反馈，以改进我们现有的行动，并添加一些新的行动。

除了我们的直接工作之外，我们还实现了一种在 Red Hat OpenShift 集群上自托管 GitHub Actions 运行程序的方法，因此您可以在自己的集群上运行工作流，而不是在 GitHub 的集群上。

如果您无法访问 OpenShift 集群，我们强烈建议您查看免费的 Red Hat OpenShift 的[开发者沙箱。还有](/developer-sandbox)[Red Hat code ready Containers](/products/codeready-containers/overview)用于在您的工作站上运行的本地选项。由于 GitHub 运行程序向外连接到 GitHub，您可以在本地或防火墙后运行它们，而不用担心联网问题。

## Red Hat OpenShift 推出自托管跑步者

我们已经开发了一个在 OpenShift 上托管 GitHub Actions runners 的解决方案。这意味着您可以在 OpenShift 上的容器中运行工作流作业，这有很多优点，我们将在本节中看到。

默认情况下， [GitHub 提供了基础设施](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)，用户可以在其上运行他们的动作工作流作业。许多操作用户对托管解决方案感到满意，不考虑其他选项。

不过，GitHub 也为用户提供了运行自己的 Actions runners 的工具。这些被称为[自托管运行器](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)，是 GitHub 运行器的替代产品。自托管运行器可以是几乎任何物理或虚拟机，运行器软件支持许多操作系统和架构。

我们特别希望在容器中构建运行器，然后我们可以在任何容器运行时启动它们——即 Red Hat OpenShift 和一般的 Kubernetes。

### 为什么要使用自托管运行器？

使用 GitHub 托管的 runners 对公共存储库来说很容易，而且是免费的。GitHub runners 是一个核心组件，随着它成为新的、[开源](/topics/open-source/)项目的免费 [CI/CD](/topics/ci-cd/) 平台，它导致了 Actions 的快速采用。然而，举办自己的跑步者有几个好处。

#### 好处#1:自托管跑步者与 GitHub 企业服务器一起工作

GitHub 企业(GHE)服务器[不支持 GitHub 托管的运行者](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)。GHE 服务器在客户公司的基础设施上本地运行。

所以，如果你是一个把所有东西都放在内部的企业，你必须使用自托管的运行器。

#### 好处#2:自主跑步者没有使用限制

GitHub 托管的跑步者并不总是对私有库免费。使用私有存储库的活动项目可能会很快用完几分钟。您不希望浪费开发人员的资源来优化您的工作流，或者为了省钱而完全禁用它们。

除了您的基础设施可以提供的资源之外，自托管运行程序没有任何资源限制。我们的工作流程还可以简化创建转轮、运行作业和拆除转轮的过程，从而降低资源使用量。或者，如果优先考虑工作流执行速度，您可以让运行程序保持在线。

#### 好处#3:自托管跑步者拥有持久磁盘

GitHub 托管的运行程序给每个工作流运行一个干净的环境。虽然在工作流之间清除状态有很多好处，但是您牺牲了在工作流之间保存数据的能力。Actions 有很好的[缓存功能](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)，但是它有时间和空间的限制，不能方便地缓存容器图像。

例如，您的工作流可能需要下载一个运行时或 SDK、您定制的开发工具堆栈、您项目的所有依赖项以及一组容器映像。

自托管运行器可以将您需要的所有工具和依赖项预安装到 Containerfile 文件中。它们可以缓存容器图像，如果您正在构建自己的图像，它们甚至会缓存中间图像层。他们甚至缓存用于运行[容器化动作](https://docs.github.com/en/actions/creating-actions/about-actions#types-of-actions)的图像，这些图像必须为 GitHub runners 上的每个工作流任务重新构建。

总的来说，由一个合适的持久磁盘保存的工作流运行时间是很重要的。

### OpenShift 操作跑步者概述

我们开发了一套工具，可以将 GitHub Actions runners 安装到现有的 Red Hat OpenShift 或 Kubernetes 集群上。它们包括:

*   一组运行 GitHub 操作运行程序并在 Red Hat OpenShift 上工作的容器图像， [OpenShift 操作运行程序](https://github.com/redhat-actions/openshift-actions-runner)。
*   从这些图像中展开吊舱的舵图， [OpenShift Actions Runner 图](https://github.com/redhat-actions/openshift-actions-runner-chart)。
*   一个自动化舵安装的动作，将转轮管理构建到您的工作流程中， [OpenShift Actions 转轮安装程序](https://github.com/redhat-actions/openshift-actions-runner-installer)。

我们将把重点放在安装程序上，这是安装跑步者最简单的方法。

#### OpenShift 操作运行器安装程序

向您的 Red Hat OpenShift 环境添加自托管运行器的最简单方法是使用 [OpenShift Actions Runner 安装程序](https://github.com/redhat-actions/openshift-actions-runner-installer)。此操作所需的配置很少。

1.  创建包含 OpenShift 集群 API 服务器 URL 和凭证的机密，并创建一个工作流步骤来登录集群。
    *   参见 [`oc-login`](https://github.com/redhat-actions/oc-login) 了解如何从动作工作流认证到 Red Hat OpenShift。
    *   在下面的例子中，URL 和访问令牌秘密分别被称为`OPENSHIFT_SERVER`和`OPENSHIFT_TOKEN`。
2.  创建一个包含 GitHub [个人访问令牌(PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) 和`repo`权限的动作秘密。
    *   如果您想在组织级别安装转轮，令牌还必须具有`admin:org`权限。
    *   您需要对想要添加跑步者的任何存储库或组织拥有管理员权限。
    *   在这个例子中，令牌秘密被称为`GITHUB_PAT`。

工作流程将如下所示:

```
name: Build Quarkus project on OpenShift
on: 
  push: 

jobs: 
  install_runner: 
    name: Install Runner 
    runs-on: ubuntu-20.04 

    steps: 
      - name: Log in to OpenShift 
        uses: redhat-actions/oc-login@v1 
        with: 
          openshift_server_url: ${{ secrets.OPENSHIFT_SERVER }} 
          openshift_token: ${{ secrets.OPENSHIFT_TOKEN }} 

      - uses: redhat-actions/openshift-actions-runner-installer@v1 
        with: 
          github_pat: ${{ secrets.GITHUB_PAT }}     # Personal access token with organization permissions
          runner_location: redhat-actions                               # Install organization level runners
          runner_image: quay.io/redhat-github-actions/java-runner-11    # Use our prebuilt Java runner image
          runner_labels: openshift, java                                

  build-quarkus: 
    name: Self Hosted Quarkus Build and Test

    runs-on: [ self-hosted, openshift, java ]             # Use the same labels we gave the runner above    
    needs: install_runner                                 # Wait until the job above completes

    env:
      WORKDIR: getting-started                  

    steps:
      - uses: actions/checkout@v2                         # Checkout a java project to build
        with:
          repository: redhat-actions/quarkus-quickstarts

      # https://github.com/redhat-actions/quarkus-quickstarts/tree/master/getting-started#readme

      - name: Install and build
        run: ./mvnw install -ntp
        working-directory: ${{ env.WORKDIR }}

      - name: Run tests
        run: ./mvnw test
        working-directory: ${{ env.WORKDIR }}

      - name: Upload jar artifacts
        uses: actions/upload-artifact@v2
        with:
          name: app-jar-files.zip
          path: ${{ env.WORKDIR }}/target/*.jar
```

这就是设置跑步者的全部工作。跑步者出现在组织或存储库设置下的**动作**菜单下的**跑步者**子部分中。

有很多方法可以配置`runner-installer`来使用你自己的图像，给你的跑步者添加描述性标签，或者增加复制品的数量。参见动作的[输入列表](https://github.com/redhat-actions/openshift-actions-runner-installer#Inputs)和[舵图`README`](https://github.com/redhat-actions/openshift-actions-runner-chart) 。

如果您希望将运行者管理与项目的工作流分开，那么一旦运行者准备就绪，您可以使用[存储库分派](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#repository_dispatch)事件来触发另一个工作流。工作流可以在同一个存储库中，也可以在不同的存储库中。请务必查看[示例](https://github.com/redhat-actions/openshift-actions-runner-installer#example-workflows)。

默认情况下，不会在每次工作流运行时重新创建流道。如果匹配相同图像、标记和标签的转轮已经存在，安装程序将退出。或者，如果您希望保持一个干净的状态并最大限度地减少资源使用，您可以在每次运行结束时删除该流道。

如果你喜欢自己管理你的跑步者操作，你可以直接使用[舵图](https://github.com/redhat-actions/openshift-actions-runner-chart)。

### OpenShift 操作跑步者图像

预构建的流道映像相对简单，可以控制映像的大小和复杂性。由于容器在没有 root 权限的情况下运行，并且没有对舵图进行修改，因此任何需要 root 权限的设置步骤都必须在映像构建时运行。这意味着必须用特定的工具链来构建图像。

到目前为止，我们已经构建了以下图像。容器文件和文档可以在[runners repository](https://github.com/redhat-actions/openshift-actions-runners)中找到:

*   基本图像，它是一个 Fedora 图像、一些核心实用程序和 actions runner 应用程序。
    *   我们所有的其他图像都是从这张图像构建的，当您构建自己的图像时，应该将这张图像作为基础图像。
*   Buildah 和 Podman 映像，它无根地运行这些工具。这个映像可以运行 OpenShift starter 工作流。
    *   此 pod 必须以比 OpenShift 默认值更高的权限运行。
    *   在 [Buildah/Podman image `README`](https://github.com/redhat-actions/openshift-actions-runners/tree/main/buildah#readme) 中可以找到设置服务帐户的说明，该帐户可以创建具有必要权限的跑步者。
*   一个 Kubernetes (K8s)工具映像，它捆绑了一组公共的 OpenShift 和 Kubernetes 相关的命令行接口(CLI)。
    *   因为 runner 是一个 pod，所以它已经通过了 OpenShift 集群的身份验证，但是默认情况下没有权限。
    *   与 Buildah 映像类似，您可以覆盖用于部署 pod 的服务帐户，以便能够读取和编辑集群中的资源。
*   一张 [Node.js](/blog/category/node-js/) 14 的图片。
    *   此图像对于运行 Node.js 项目的 CI/CD 工作流非常有用。
    *   它是一个向基本运行程序添加运行时和开发工具的例子。
*   一张 [Java](/topics/enterprise-java/) 开发套件 11 的图片。
    *   此图有助于运行 Java 项目的 CI/CD 工作流，并用于上一节示例中的 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 工作流。
    *   它也是一个向基本运行程序添加运行时和开发工具的例子。

您可以使用这些图像，或者将它们作为如何扩展基础图像和创建您自己的图像的示例。说明见[跑垒者`README`](https://github.com/redhat-actions/openshift-actions-runner/tree/main/base#building-your-own-runner-image) 。

我们很乐意帮助您创建一个可以运行您的工作流程的映像。此外，如果您认为其他人可能想要使用您的图像，请提出问题，我们可以探索将其合并到我们的组织中。

#### 结论

尝试一下新的动作和 OpenShift 自托管 runner。我们努力使 Red Hat OpenShift 和 GitHub 操作之间的集成尽可能简单和有用，在您的 OpenShift 集群中托管工作流运行程序是一个很好的下一步。

我们一直在寻求社区反馈。如果您有错误报告、增强请求、问题、行动想法或新的用例，请在我们的任何存储库上提出问题，我们一定会看一看。

*Last updated: August 24, 2022*