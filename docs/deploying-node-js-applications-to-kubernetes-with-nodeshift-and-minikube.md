# 使用 Nodeshift 和 Minikube 将 Node.js 应用程序部署到 Kubernetes

> 原文：<https://developers.redhat.com/blog/2021/03/09/deploying-node-js-applications-to-kubernetes-with-nodeshift-and-minikube>

在[之前的一篇文章](/blog/2019/08/30/easily-deploy-node-js-applications-to-red-hat-openshift-using-nodeshift/)中，我展示了在开发期间使用 Nodeshift 命令行界面(CLI)将 [Node.js](/topics/nodejs) 应用程序部署到 [Red Hat OpenShift](/products/openshift/overview) 是多么容易。在本文中，我们将了解如何使用 Nodeshift 将 Node.js 应用程序部署到 vanilla[Kubernetes](/topics/kubernetes/)—具体来说，就是 Minikube。

## 入门指南

如果你想跟随这个教程，你需要运行 Minikube。我不会介绍设置过程，但是 Minikube 的文档可以指导你完成它。对于本教程，我还假设您已经安装了 [Node.js 和节点包管理器(npm)](https://nodejs.org/en/download) 。

我们将使用的代码样本可以在 [GitHub](https://github.com/nodeshift-starters/basic-node-app-dockerized) 上获得。我们的示例是一个非常基本的 Node.js 应用程序，带有 Dockerfile。其实是摘自 Nodejs.org 上的[*dockering a node . js web app*](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)指南。

## Nodeshift CLI

正如 Nodeshift 模块自述文件所述，Nodeshift 是一个固执己见的命令行应用程序和可编程 API，您可以使用它将 Node.js 应用程序部署到 [Red Hat OpenShift](/products/openshift/overview) 。您可以使用`npx`命令轻松运行它，它将创建适当的 YAML 文件来部署您的应用程序。

如果您使用 OpenShift 集群进行开发，Nodeshift 是一个很好的工具，它使用了源到映像(S2I)工作流。简而言之，Nodeshift 创建一个 OpenShift `BuildConfig`，它调用 Node.js S2I 映像来构建您的节点应用程序。在大多数情况下，您可以通过运行`npm install`来实现这一点。构建结果被放入 OpenShift `ImageStream`中，它驻留在内部 OpenShift 容器注册表中。然后，这个映像用于部署您的应用程序。

但是，如果部署到一个对 BuildConfigs、ImageStreams 或 S2I 一无所知的普通 Kubernetes 集群呢？那么，从 [Nodeshift 的 7.3 版本](https://github.com/nodeshift/nodeshift/releases/tag/v7.3.0)开始，您现在可以将 Node.js 应用程序部署到 Minikube。

## 将 Node.js 部署到 Minikube

在我们研究 Nodeshift 如何将 Node.js 应用程序部署到 Minikube 之前，让我们花一点时间从较高的层面概述一下部署到 Kubernetes。

首先，您将创建一个应用程序容器映像，这可以用 Docker 来完成。一旦有了容器映像，就需要将该映像推送到集群可以访问的容器注册中心，类似于 [Docker Hub](https://hub.docker.com/) 。一旦映像可用，就必须在部署 YAML 中指定该映像，并创建一个服务来公开应用程序。

当您开始迭代代码时，这个流程开始变得更加麻烦。如果您每次都需要运行 Docker 构建并将新的映像推送到 Docker Hub，这并不真正有利于开发。更不用说您还需要用新版本的映像更新您的部署，以确保它可以重新部署。

Nodeshift 的目标是让开发人员在部署到 OpenShift 和 Kubernetes 时更轻松。让我们看看 Nodeshift 如何帮助完成这些笨拙的步骤。

## Minikube 的内部 Docker 服务器

OpenShift 和 Kubernetes 之间的一个主要区别是，在普通的 Kubernetes 上运行 S2I 版本并不容易。我们也不希望每次更改代码时都运行 Docker 构建并推送到 Docker Hub。幸运的是，Minikube 给了我们一个选择。

Minikube 有自己的内部 Docker 服务器，我们可以使用 [Docker 引擎 API](https://docs.docker.com/engine/api/v1.41/#) 连接到该服务器。我们可以使用这个服务器在环境中运行我们的 Docker 构建，这意味着我们不必将映像推送到 Docker Hub 这样的外部资源。然后，我们可以在部署中使用这个映像。

为了访问内部 Docker 服务器，Minikube 有一个命令可以导出一些环境变量来添加到您的终端 shell 中。这个命令是`minikube docker-env`，它可能会输出如下内容:

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.39.12:2376"
export DOCKER_CERT_PATH="/home/lucasholmquist/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)

```

## 使用 Nodeshift 更容易

Nodeshift 抽象出我们并不真正关心的细节，这样我们就可以专注于我们的应用程序。在这种情况下，我们不想考虑如何连接到 Minikube 的内部服务器或如何手动运行 Docker 命令，也不想考虑每次构建新映像来重新部署时都要更新我们的部署 YAML。

使用带有`--kube`标志的 Nodeshift CLI 简化了这些任务。让我们使用我们的[示例应用程序](https://github.com/nodeshift-starters/basic-node-app-dockerized)来看看它是如何工作的。

我们将使用`npx`将 Node.js 应用程序部署到 Minikube，因此我们不需要全局安装任何东西。在示例目录中像这样运行它:

```
$ npx nodeshift --kube

```

如果没有提供任何服务和部署，Nodeshift 会默认创建一个服务和部署。另外，请注意它创建的服务类型是一个`LoadBalancer`，它允许我们在不使用 ingress 的情况下公开我们的应用程序。

Nodeshift CLI 为 Kubernetes 部署运行与 OpenShift 部署相同的`goals`。关键的区别出现在`build`阶段。Nodeshift 没有创建 OpenShift `BuildConfig`并在集群上运行 S2I 进程，而是使用 [dockerode](https://www.npmjs.com/package/dockerode) 模块连接到 Minikube 的内部 Docker 服务器，并使用提供的 Docker 文件运行构建。构建映像现在位于内部注册表中，准备好由 Nodeshift CLI 创建的部署 YAML 进行部署。Nodeshift 还向部署的元数据中添加一个随机生成的数字，然后在每次重新部署时应用该数字。这将触发 Minikube 使用新的映像重新部署应用程序。

以下是日志输出示例:

```
~/develop/nodeshift-starters/basic-node-app-dockerized» npx nodeshift --kube                                        

2021-02-09T20:03:18.405Z INFO loading configuration
2021-02-09T20:03:18.452Z INFO Using the kubernetes flag.
2021-02-09T20:03:18.762Z INFO using namespace default at https://192.168.39.12:8443
2021-02-09T20:03:18.763Z WARNING a file property was not found in your package.json, archiving the current directory.
2021-02-09T20:03:18.773Z INFO creating archive of .dockerignore, .gitignore, Dockerfile, README.md, package-lock.json, package.json, server.js
2021-02-09T20:03:18.774Z INFO Building Docker Image
2021-02-09T20:03:18.848Z TRACE {"stream":"Step 1/7 : FROM node:14"}
2021-02-09T20:03:18.848Z TRACE {"stream":"\n"}
2021-02-09T20:03:18.849Z TRACE {"stream":" ---\u003e cb544c4472e9\n"}
2021-02-09T20:03:18.849Z TRACE {"stream":"Step 2/7 : WORKDIR /usr/src/app"}
2021-02-09T20:03:18.849Z TRACE {"stream":"\n"}
2021-02-09T20:03:18.849Z TRACE {"stream":" ---\u003e Using cache\n"}
2021-02-09T20:03:18.849Z TRACE {"stream":" ---\u003e 57c9e3a4e918\n"}
2021-02-09T20:03:18.849Z TRACE {"stream":"Step 3/7 : COPY package*.json ./"}
2021-02-09T20:03:18.850Z TRACE {"stream":"\n"}
2021-02-09T20:03:19.050Z TRACE {"stream":" ---\u003e 742050ca3266\n"}
2021-02-09T20:03:19.050Z TRACE {"stream":"Step 4/7 : RUN npm install"}
2021-02-09T20:03:19.050Z TRACE {"stream":"\n"}
2021-02-09T20:03:19.109Z TRACE {"stream":" ---\u003e Running in f3477d5f2b00\n"}
2021-02-09T20:03:21.739Z TRACE {"stream":"\u001b[91mnpm WARN basic-node-app-dockerized@1.0.0 No description\n\u001b[0m"}
2021-02-09T20:03:21.744Z TRACE {"stream":"\u001b[91mnpm WARN basic-node-app-dockerized@1.0.0 No repository field.\n\u001b[0m"}
2021-02-09T20:03:21.745Z TRACE {"stream":"\u001b[91m\n\u001b[0m"}
2021-02-09T20:03:21.746Z TRACE {"stream":"added 50 packages from 37 contributors and audited 50 packages in 1.387s\n"}
2021-02-09T20:03:21.780Z TRACE {"stream":"found 0 vulnerabilities\n\n"}
2021-02-09T20:03:22.303Z TRACE {"stream":"Removing intermediate container f3477d5f2b00\n"}
2021-02-09T20:03:22.303Z TRACE {"stream":" ---\u003e afb97a82c035\n"}
2021-02-09T20:03:22.303Z TRACE {"stream":"Step 5/7 : COPY . ."}
2021-02-09T20:03:22.303Z TRACE {"stream":"\n"}
2021-02-09T20:03:22.481Z TRACE {"stream":" ---\u003e 1a451003c472\n"}
2021-02-09T20:03:22.481Z TRACE {"stream":"Step 6/7 : EXPOSE 8080"}
2021-02-09T20:03:22.482Z TRACE {"stream":"\n"}
2021-02-09T20:03:22.545Z TRACE {"stream":" ---\u003e Running in a76389d44b59\n"}
2021-02-09T20:03:22.697Z TRACE {"stream":"Removing intermediate container a76389d44b59\n"}
2021-02-09T20:03:22.697Z TRACE {"stream":" ---\u003e 8ee240b7f9ab\n"}
2021-02-09T20:03:22.697Z TRACE {"stream":"Step 7/7 : CMD [ \"node\", \"server.js\" ]"}
2021-02-09T20:03:22.698Z TRACE {"stream":"\n"}
2021-02-09T20:03:22.759Z TRACE {"stream":" ---\u003e Running in 1f7325ab3c64\n"}
2021-02-09T20:03:22.911Z TRACE {"stream":"Removing intermediate container 1f7325ab3c64\n"}
2021-02-09T20:03:22.912Z TRACE {"stream":" ---\u003e d7f5d1e95592\n"}
2021-02-09T20:03:22.912Z TRACE {"aux":{"ID":"sha256:d7f5d1e9559242f767b54b168c36df5c7cbce6ebc7eb1145d7f6292f20e8cda2"}}
2021-02-09T20:03:22.913Z TRACE {"stream":"Successfully built d7f5d1e95592\n"}
2021-02-09T20:03:22.929Z TRACE {"stream":"Successfully tagged basic-node-app-dockerized:latest\n"}
2021-02-09T20:03:22.933Z WARNING No .nodeshift directory
2021-02-09T20:03:22.954Z INFO openshift.yaml and openshift.json written to /home/lucasholmquist/develop/nodeshift-starters/basic-node-app-dockerized/tmp/nodeshift/resource/
2021-02-09T20:03:22.975Z INFO creating new service basic-node-app-dockerized
2021-02-09T20:03:22.979Z TRACE Deployment Applied
2021-02-09T20:03:23.036Z INFO Application running at: http://192.168.39.12:30076
2021-02-09T20:03:23.036Z INFO complete

```

部署之后，Nodeshift CLI 还会在控制台输出中提供应用程序运行的 URL。输出可能如下所示:

```
...
INFO Application running at http://192.168.39.12:30769
...

```

导航到提供的 URL 会返回“Hello World”

## 结论

本文简要概述了 Nodeshift CLI 对部署到 Minikube 的支持。将来，我们计划添加更多的 Kubernetes 平台和其他开发人员友好的特性，比如可能让 Nodeshift CLI 创建一个默认的 Dockerfile(如果没有的话)。

如果你喜欢你所看到的，并想了解更多，请查看 [Nodeshift 项目](https://nodeshift.dev/)。和往常一样，如果有更多你想看的功能，[会在 GitHub 上创建一个问题](https://github.com/nodeshift/nodeshift)。要了解更多关于 Red Hat 在 Node.js 方面的进展，请查看我们的 [Node.js 登陆页面。](https://developers.redhat.com/topics/nodejs)

*Last updated: March 8, 2021*