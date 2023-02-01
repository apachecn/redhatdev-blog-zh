# 在 Red Hat OpenShift 上使用 Node.js 14

> 原文：<https://developers.redhat.com/blog/2020/05/21/use-node-js-14-on-red-hat-openshift>

4 月 21 日， [Node.js](https://developers.redhat.com/blog/category/node-js/) 用 [Node.js 14](https://nodejs.org/en/blog/release/v14.0.0/) 发布了它的最新主版本。因为这是一个偶数版本，所以它将在 2020 年 10 月成为长期支持(LTS)版本。这个版本带来了许多改进和特性，比如改进的诊断、V8 升级、实验性异步本地存储 API、强化的流 API 等等。

虽然 Red Hat 将在未来几个月为 Node.js 14 发布一个通用基础映像(UBI)T1，用于 [Red Hat OpenShift](https://developers.redhat.com/openshift) 和 [Red Hat Enterprise Linux](https://developers.redhat.com/topics/linux/) ，但这篇文章可以帮助您立即开始。如果您对 Node.js 14 的改进和新特性感兴趣，请查看最后列出的文章。

让我们使用一个基于官方 [*的示例应用程序来说明如何对 Node.js 应用程序【Nodejs.org 文档*](https://nodejs.org/fr/docs/guides/nodejs-docker-webapp/#dockerizing-a-node-js-web-app)进行 Dockerize。这是一个简单的 Express.js 应用程序，带有 Dockerfile，使用最新的上游社区 Node.js 14 映像。

## 如何部署

首先，对包含 Dockerfile 的 Git repo 使用`oc new-app`命令:

```
$ oc new-app https://github.com/nodeshift-starters/basic-node-app-dockerized

```

要访问您的应用程序，您需要使用以下简单命令公开它:

```
$ oc expose svc/basic-node-app-dockerized

```

或者，您可以使用 [Nodeshift 模块](https://www.npmjs.com/package/nodeshift)部署一个本地目录。假设您克隆了我们之前使用的项目，您可以运行以下命令:

```
$ npx nodeshift --build.strategy=Docker --expose

```

### 包裹

如您所见，今天在 Red Hat OpenShift 上使用 Node.js 14 非常简单。要了解 Node.js 14 的更多改进和特性，请查看 Node.js 官方博客文章。

*Last updated: June 26, 2020*