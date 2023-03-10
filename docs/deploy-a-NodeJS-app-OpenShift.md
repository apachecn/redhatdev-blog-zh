# 在 Red Hat OpenShift 上部署 3 层 Node.js 应用程序

> 原文：<https://developers.redhat.com/articles/deploy-a-NodeJS-app-OpenShift>

通过这里展示的练习，您将看到如何在 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 上部署一个三层 Node.js 应用程序。示例应用程序是一个特许售货亭，用于简化下特许订单的过程。用户将通过前端 web 应用程序下订单。订单信息将保存在 MongoDB 数据库中，然后用户将获得一个订单号。一旦他们的订单号被调用，用户就可以去特许窗口付款并收到他们的食物。

### 先决条件

在开始之前，您需要访问一个 Red Hat OpenShift 集群。如果您还没有访问权限，您可以安装[Red Hat CodeReady Containers](https://developers.redhat.com/products/codeready-containers)，这将允许您在本地计算机上的 code ready Containers 虚拟机中运行 OpenShift。要安装 CodeReady 容器，请访问[https://www . red hat . com/en/technologies/cloud-computing/open shift/try-it](https://www.redhat.com/en/technologies/cloud-computing/openshift/try-it)，并选择*笔记本电脑*作为您的基础设施提供商:

![illustration of a laptop, which says "Powered by Red Hat CodeReady Containers" below it, and below that says "Developer Preview"](img/fd5f3bc5aa33d201ca78f09910ade016.png "kleinert_f1_deploy-3tier-nodejs")

另一个选择是注册 OpenShift Online 的[免费初级版](https://www.openshift.com/trial/)或 [30 天免费试用的专业版](https://www.openshift.com/trial/)。

一旦您可以访问 OpenShift 集群，请确保您可以访问 oc 命令行工具。如果您使用 CodeReady 容器，它会附带 oc。如果你用另一种方式访问 OpenShift，你可能需要下载 oc。您可以在这里下载适合您操作系统的版本[。](https://mirror.openshift.com/pub/openshift-v4/clients/oc/4.2/)

使用`oc`登录有两种不同的选择。一种选择是使用您的用户名和密码:

```
oc login <cluster addr>:<cluster port> -u <username> -p <password>
```

另一个选择是首先登录 OpenShift Web 控制台，然后点击右上角的用户名。将出现一个下拉菜单。

![dropdown screen with two options: Copy Login Command and Log out](img/7817d166e811a3fff89edda9dc1db352.png "kleinert_f2_deploy-3tier-nodejs")

点击*复制登录命令*并按照指示操作，你将得到一个带有你的 API 令牌的`oc`登录命令。它的格式如下所示:

```
oc login --token=<abc…> --server=<cluster addr>:<cluster port>
```

登录后，您就可以进入下一部分了。

## 第 1 部分:部署和链接后端

#### 后端应用程序概述

后端使用 Node.js 和 Express 编写，将作为前端和数据库(我们将在第 3 部分稍后创建)之间的中介。它会将订单信息保存到数据库中(如果数据库可用)，并生成一个订单编号提供给客户。

#### 登录到 OpenShift 集群

如果您刚刚完成了先决条件，您可能仍然登录到您的 OpenShift 集群。如果没有，您可以通过多种方式进行验证:您可以使用令牌进行验证，令牌可以从 web 控制台获得，或者您可以使用用户名和密码。选择以下选项之一:

```
oc login <cluster addr>:<cluster port> --token=<abc…>
```

```
oc login <cluster addr>:<cluster port> -u <username> -p <password>
```

#### 在 OpenShift 上部署后端

首先，我们需要创建一个项目。我们将在这个项目中部署后端、前端和数据库。

```
oc new-project concession-kiosk
```

我们后端的代码在这个 GitHub repo 中:[https://github.com/jankleinert/concession-kiosk-backend](https://github.com/jankleinert/concession-kiosk-backend)。

我们将使用`oc new-app`命令从这个源代码构建一个容器映像，然后将它部署在 OpenShift 上。`oc new-app`命令使用[源到映像](https://github.com/openshift/source-to-image)，它可以根据代码的特征推断使用哪个构建器映像。在这种情况下，由于我们在 repo 中有一个 package.json 文件，new-app 知道使用 Node.js 构建器映像。

```
oc new-app [https://github.com/jankleinert/concession-kiosk-backend](https://github.com/jankleinert/concession-kiosk-backend) --name backend
```

您的输出应该如下所示:

```
--> Found image a7092f1 (3 days old) in image stream 
"openshift/nodejs" under tag "10" for "nodejs"

    Node.js 10.16.3
    ---------------
    Node.js 10.16.3 available as a container is a base platform for 
building and running various Node.js 10.16.3 applications and 
frameworks. Node.js is a platform built on Chrome's JavaScript runtime 
for easily building fast, scalable network applications. Node.js uses 
an event-driven, non-blocking I/O model that makes it lightweight and 
efficient, perfect for data-intensive real-time applications that run 
across distributed devices.

    Tags: builder, nodejs, nodejs-10.16.3

    * The source repository appears to match: nodejs
    * A source build using source code from 
https://github.com/jankleinert/concession-kiosk-backend will be 
created
      * The resulting image will be pushed to image stream tag 
"backend:latest"
      * Use 'oc start-build' to trigger a new build
    * This image will be deployed in deployment config "backend"
    * Port 8080/tcp will be load balanced by service "backend"
      * Other containers can access this service through the hostname 
"backend"

--> Creating resources ...
    imagestream.image.openshift.io "backend" created
    buildconfig.build.openshift.io "backend" created
    deploymentconfig.apps.openshift.io "backend" created
    service "backend" created
--> Success
    Build scheduled, use 'oc logs -f bc/backend' to track its 
progress.
    Application is not exposed. You can expose services to the outside 
world by executing one or more of the commands below:
     'oc expose svc/backend'
    Run 'oc status' to view your app. 
```

运行`oc logs -f bc/backend`来观察构建的状态。一旦它显示了`Push successful`，构建就完成了。

通过运行此命令创建的资源之一是服务。服务为我们提供了一种访问集群中后端应用程序的方法。运行以下命令，查看有关后端服务的一些基本信息:

```
oc get svc backend
```

输出应该如下所示:

```
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend   ClusterIP   172.30.236.87           8080/TCP   7m53s 
```

通过服务名`backend`并使用端口 8080，可以在 OpenShift 集群中访问后端服务。在第 2 部分中，您将看到当我们链接前端和后端时，前端将能够通过其服务名和端口与后端通信。

## 第 2 部分:部署前端

该应用程序的前端是一个简单的 web 应用程序，使用 Node.js 和 EJS(嵌入式 JavaScript)构建模板。它显示了特许售货亭的菜单。一旦用户下了订单，就会显示一个带有订单号的确认页面，这样他们就可以等待订单被调用。

#### 部署前端

类似于我们为后端所做的，我们也将使用`oc new-app`来部署前端。不过这一次，我们传入了一些环境变量。这些是需要设置的环境变量，以便前端通过其服务名与后端通信，如前一节所述。

```
oc new-app https://github.com/jankleinert/concession-kiosk-frontend --name frontend -e COMPONENT_BACKEND_HOST=backend -e COMPONENT_BACKEND_PORT=8080
```

和以前一样，您可以在构建过程中查看日志:

```
oc logs -f bc/frontend
```

#### 为前端创建一个 URL

随着我们的前端部署，我们希望能够在浏览器中访问它，所以我们需要一个公开访问它的 URL。为此，我们将创建一个路由来公开服务。

`oc expose svc/frontend`

然后，我们可以运行以下命令来获取创建的路由。

`oc get routes`

复制该 URL 并将其粘贴到您的浏览器中。您应该会看到如下所示的特许售货亭菜单。

![menu with hotdog, hamburger, salad, pizza, and drink](img/77d78b7be50d088368b3e5698f56fec3.png "kleinert_f3_deploy-3tier-nodejs")

试试看效果如何。用户可以选择他们想要订购的商品和数量，一旦下了订单，他们就会收到一个订单号。订单号现在只是一个占位符，直到我们有一个合适的数据库。

在第 3 部分中，我们将部署并链接数据库，以便存储订单并生成真实的订单号。

## 第 3 部分:部署和链接数据库

到目前为止，我们一直使用一些硬编码的值作为订单号，实际上我们并没有在任何地方存储订单信息。在这一部分中，我们将通过部署和连接 MongoDB 数据库来更新它。

#### 创建一个 MongoDB 数据库服务

确保您已登录到您的 Red Hat OpenShift 集群，并确认您仍在特许售货亭项目中:

```
oc project concession-kiosk
```

我们将使用一个模板来创建一个 MongoDB 数据库。模板描述了一组对象，可以对这些对象进行参数化和处理，以生成 OpenShift 创建的对象列表。OpenShift 中包含一组模板。我们将使用的是[MongoDB-短命模板](https://github.com/openshift/library/blob/master/official/mongodb/templates/mongodb-ephemeral.json)。

**注意:**MongoDB-periodic 模板不使用持久存储，因此应该只用于测试。如果您需要持久存储，最好使用 mongodb-persistent 模板。

使用模板创建数据库:

```
oc new-app --template=mongodb-ephemeral
```

您应该会看到类似这样的输出:

```
--> Deploying template "openshift/mongodb-ephemeral" to project 
concession-kiosk

     MongoDB (Ephemeral)
     ---------
     MongoDB database service, without persistent storage. For more 
information about using this template, including OpenShift 
considerations, see 
https://github.com/sclorg/mongodb-container/blob/master/3.2/README.md.

     WARNING: Any data stored will be lost upon pod destruction. Only 
use this template for testing

     The following service(s) have been created in your project: 
mongodb.

            Username: 
            Password: 
       Database Name: sampledb
      Connection URL: mongodb://:@mongodb/sampledb

     For more information about using this template, including 
OpenShift considerations, see 
https://github.com/sclorg/mongodb-container/blob/master/3.2/README.md.

     * With parameters:
        * Memory Limit=512Mi
        * Namespace=openshift
        * Database Service Name=mongodb
        * MongoDB Connection Username= # generated
        * MongoDB Connection Password= # generated
        * MongoDB Database Name=sampledb
        * MongoDB Admin Password= # generated
        * Version of MongoDB Image=3.6

--> Creating resources ...
    secret "mongodb" created
    service "mongodb" created
    deploymentconfig.apps.openshift.io "mongodb" created
--> Success
    Application is not exposed. You can expose services to the outside 
world by executing one or more of the commands below:
     'oc expose svc/mongodb'
    Run 'oc status' to view your app. 
```

#### 更新后端以与数据库通信

数据库启动并运行后，我们需要为后端应用程序设置一个环境变量，告诉它 MongoDB 连接 URL。从输出中复制连接 URL。它将采用以下格式:

```
mongodb://<generated username>:<generated password>@mongodb/sampledb
```

如果您查看 index.js 文件的后端，您会看到下面一行。我们将使用`oc set env`命令来设置 MONGODB_URL 环境变量。

```
const dbConnectionUrl = process.env.MONGODB_URL || 
'mongodb://localhost:27017/sampledb’;
```

运行这个命令，用您自己的字符串替换 MongoDB 连接字符串:

```
oc set env dc/backend MONGODB_URL=mongodb://<generated username>:<generated password>@mongodb/sampledb
```

这一更改将触发新的部署，这应该不会花费很长时间。在您的浏览器中重新加载前端并下新订单。现在，对于您下的每个订单，您应该会看到一个新的订单号，并且您的订单内容也应该会显示在感谢页面上。

![message that says Concession Kiosk - Thank you, your order number is 126, your order contains 2 hotdogs, 1 hamburger, 4 salads, 1 pizza, and 4 sodas](img/f3405837e8ec4ce1bd1df97e59997018.png "kleinert_f4_deploy-3tier-nodejs")Code repos for concession kiosk exercise![Github code](img/f0537c48bb7b26086ce2bf026abafc76.png)*Github Code**[github.com/jankleinert/concession-kiosk-backend](https://github.com/jankleinert/concession-kiosk-backend)**![Github code](img/f0537c48bb7b26086ce2bf026abafc76.png)*Github Code**[github.com/jankleinert/concession-kiosk-frontend](https://github.com/jankleinert/concession-kiosk-frontend)*****Last updated: January 9, 2023***