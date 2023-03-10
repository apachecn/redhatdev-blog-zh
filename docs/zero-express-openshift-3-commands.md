# 在三个命令中用零来表示 OpenShift

> 原文：<https://developers.redhat.com/blog/2018/04/16/zero-express-openshift-3-commands>

**(编辑:2019 . 11 . 22)本帖使用的节点图片，无论是社区`centos7`还是`product`，都不再更新维护。对于社区图像，请使用位于此处的基于通用基础图像(UBI)的节点图像:[registry.access.redhat.com/ubi8/nodejs-10](http://registry.access.redhat.com/ubi8/nodejs-10)**

**如需 Node.js 的完整支持产品版本，请查看红帽软件收藏 Node.js 图片， [RH SCL Node.js](https://access.redhat.com/containers/#/registry.access.redhat.com/rhscl/nodejs-10-rhel7) 。**

随着最近[宣布 Node.js 作为 Red Hat OpenShift 应用程序运行时的一部分普遍可用，](https://developers.redhat.com/blog/2018/03/12/rhoar-nodejs-annoucement/)我想看看在 OpenShift 上部署 [Express.js](https://expressjs.com/) 应用程序有多容易。

### 入门指南

在我们开始之前，有一些必要的先决条件。您需要安装 [Node 8.x](https://nodejs.org) 和 [npm 5.2](https://www.npmjs.com/) 或更高版本。npm 附带了官方的 node 发行版，所以如果你安装来自 Nodejs.org[的 Node](https://nodejs.org)，你应该没问题。

您还需要访问 OpenShift 环境或 Red Hat Container Development Kit(CDK)minishift 环境。对于这个例子，我将使用 minishift。你可以在这里找到关于启动和运行 minishift[的说明。对于我的本地微移位，我使用以下命令启动它:](https://developers.redhat.com/products/cdk/hello-world/)

```
$ minishift start --memory=6144 --vm-driver virtualbox
```

您还需要使用`oc login`登录到您正在使用的任何 OpenShift 集群(OpenShift 或 minishift)。

### 剧透警报

对于那些不想阅读整篇文章也不想滚动到最后的人来说，这里有三个需要运行的命令:

```
$ npx express-generator .
```

```
$ npx json -I -f package.json -e 'this.scripts.start="PORT=8080 node ./bin/www"'
```

```
$ npx nodeshift --strictSSL=false --expose
```

### 生成快速应用程序

你说什么是快递？嗯，根据 [Express 网站](https://expressjs.com/)的说法，Express 是“Node.js 的快速、非个人化、极简主义的网络框架。”

关于 Express 的一个非常酷的东西是 *Express 应用生成器工具* : `express-generator`。这是一个命令行工具，[“快速创建应用程序框架”](https://expressjs.com/en/starter/generator.html)。但是等等:我刚才不是说了快递是无人售票的吗？的确是，但这就是固执己见的骷髅创造者。_(ツ)_/

Express 网站推荐全局安装`express-generator`模块，像这样:

```
npm install -g express-generator
```

但是我们不会那么做。相反，我们将使用 npm 的一个相当新的特性，叫做`npx`。

使我们能够运行一次性命令，而不必全局安装。除此之外还有更多功能，所以如果你对`npx`能做的所有酷的事情感兴趣，在这里查看[。](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

有了这些新发现的知识，我们现在可以生成这样的 Express 应用程序:

```
$ npx express-generator .
```

让我们快速看一下这个命令实际上发生了什么。首先，`npx`看到我们想要运行`express-generator`命令，所以`npx`变了些戏法，看看我们是否在本地安装了它(在我们当前的目录中)，然后它检查我们的全局模块。因为它不在那里，所以它下载它以供一次性使用。

`express-generator`在我们当前的目录中运行，这个目录用**表示。**命令结束。

结果应该是这样的:

![](img/40301971758deda9036a739489ede189.png)

`express-generator`也给了我们一些关于如何安装依赖项以及如何运行应用程序的说明。你现在可以跳过这一步。

### 更新 package.json 文件

既然我们使用一个命令创建了基本的 Express 应用程序，我们需要在部署应用程序之前向`package.json`添加一个东西。

我们需要将一个`PORT`环境变量传递给我们的启动脚本。

一种方法是打开一个文本编辑器，这样做，但这将增加几个步骤。要在一个命令中完成这个任务，我们可以使用 [json 模块](https://www.npmjs.com/package/json)。

```
$ npx json -I -f package.json -e 'this.scripts.start="PORT=8080 node ./bin/www"'
```

和以前一样，我们使用`npx`命令来允许我们不必全局安装`json`模块。

让我们看看传递给`json`模块的选项是怎么回事。

`-I -f package.json`表示我们想要就地编辑文件`package.json`。`-e`选项将执行一些 JavaScript 代码，在本例中是用字符串`"PORT=8080 node ./bin/www"`设置来自`package.json`的`scripts.start`属性。

有关`json`模块的更多信息，请查看[文档](http://trentm.com/json/)。

### 将应用程序部署到 OpenShift

现在，最后一步是运行这个命令:

```
$ npx nodeshift --strictSSL=false --expose
```

这里，我们使用 [nodeshift 模块](https://www.npmjs.com/package/nodeshift)来部署我们的应用程序。是一个 CLI 或可编程 API，有助于将节点应用部署到 OpenShift。

`npx`和前面的例子做着同样的事情。

`nodeshift`正在使用两个标志。第一个是`strictSSL=false`，当部署到 minishift 或使用自签名证书的某个地方时需要它。如果我们部署到一个真正的 OpenShift 集群，我们可以不考虑它。

第二个标志`expose`告诉`nodeshift`它应该为我们创建一条*路线*，这允许我们的应用程序被外界看到。(如果您在本地运行 minishift，则只有您可以看到该应用程序。)

该命令的输出如下所示:

![](img/e61b8af5cd0a91d60b5155eaca227a05.png)

如果我们转到正在运行的 minishift 的 web UI，我们可以看到创建的 pod 现在正在成功运行。

![](img/a5c3668c993329b3de2dad58d1977930.png)

然后，如果我们单击该链接，我们可以看到我们的示例应用程序正在运行:

![](img/e9606453b7930f2f4547db9c56b9b29b.png)

**注意:**上面的例子将使用最新的[社区 s2i 图像](https://hub.docker.com/r/bucharestgold/centos7-s2i-nodejs/)(撰写本文时为 9.x)。要在 OpenShift 上使用 Node.js 的完整支持版本，您只需添加“- dockerImage”标志。

这将集成 Red Hat OpenShift 应用程序运行时版本 Node.js (8.x ),作为我们产品订阅的一部分，您可以获得完整的生产和开发人员支持。

这可能看起来像这样:

```
$ npx nodeshift --strictSSL=false --expose --dockerImage=registry.access.redhat.com/rhoar-nodejs/nodejs-8
```

### 概述

在本文中，这些命令有点分散，所以让我们再一起看看:

```
$ npx express-generator .
```

```
$ npx json -I -f package.json -e 'this.scripts.start="PORT=8080 node ./bin/www"'
```

```
$ npx nodeshift --strictSSL=false --expose
```

我们创建的示例应用程序非常简单，但它展示了在 OpenShift 上使用 Node.js 有多快。

*Last updated: October 6, 2022*