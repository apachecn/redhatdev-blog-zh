# 今天就在 Red Hat OpenShift 上使用 Node.js 12

> 原文：<https://developers.redhat.com/blog/2019/04/29/use-node-js-12-on-red-hat-openshift-today>

4 月 23 日，Node.js 以 [Node.js 12](https://nodejs.org/en/blog/release/v12.0.0/) 发布了最新的主版本。因为这是一个偶数版本，所以它将成为十月份的长期支持(LTS)版本，代号为铒。

这个版本带来了大量的改进和特性，这篇博文不打算介绍。相反，我将集中讨论如何在 Red Hat OpenShift 上开始使用这个新版本。如果您对各种改进和新特性感兴趣，请查看本文末尾列出的文章。

Nodeshift 团队为 Node.js 创建并维护[源到映像(S2I)容器映像，我很高兴地向大家报告，我们已经发布了 Node.js 12。](https://cloud.docker.com/u/nodeshift/repository/docker/nodeshift/centos7-s2i-nodejs/tags)

### 部署

对于那些熟悉使用 S2I 图像的过程的人，你继续做你所做的。但是，对于那些对这个过程不太熟悉的人来说，这里有几个关于如何使用 Node.js 12 映像部署应用程序的快速示例。

首先，您可以将`oc new-app command`用于 Git repo:

```
oc new-app nodeshift/centos7-s2i-nodejs:12.x~https://github.com/nodeshift-starters/nodejs-rest-http

oc expose svc/nodejs-rest-http

```

注意，我们指定了 12.x 标签。

或者，您可以使用 [Nodeshift 模块](https://www.npmjs.com/package/nodeshift)部署一个本地目录:

```
npx nodeshift --imageTag=12.x --expose

```

同样，我们指定了 12.x 标签。

### 包裹

如您所见，今天在 Red Hat OpenShift 上使用 Node.js 12 非常简单。

作为额外的奖励，对于那些在 Red Hat OpenShift 上开发 web 应用程序的人，我们还发布了 Web 应用程序构建器映像的 [Node.js 12 版本。](https://cloud.docker.com/u/nodeshift/repository/docker/nodeshift/centos7-s2i-web-app/tags)

要了解如何使用该图像的更多信息，请查看“OpenShift 上的现代 web 应用程序”系列文章:

*   OpenShift 上的现代网络应用:第一部分——两个命令中的网络应用
*   OpenShift 上的现代 web 应用:第 2 部分——使用链式构建
*   OpenShift 上的现代 web 应用:第 3 部分——作为开发环境的 open shift

要了解 Node.js 12 中的改进和特性，您也可以查看官方的 [Node.js 博文](https://medium.com/@nodejs/introducing-node-js-12-76c41a1b3f3f)。

*Last updated: May 1, 2019*