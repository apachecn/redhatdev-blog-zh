# 使用 OpenShift 加速 Node.js 的开发

> 原文：<https://developers.redhat.com/blog/2017/11/28/accelerating-development-node-js-using-openshift>

在这篇博客中，我想介绍一种“不同的”方式来使用 OpenShift。在部署 Pod 到 OpenShift 的典型方式中，我们有一组非常有用的对象，我们有 [**构建/映像**](https://docs.openshift.com/enterprise/3.1/dev_guide/builds.html) 配置。这通过隐藏关于映像构造的细节减轻了我们的痛苦，但是，有时我们只是想看到一些运行在云中的代码。或者我们想看看我们的服务/应用程序是否能够与附近的服务交互，或者我们有一些代码，但我们还不想使用 git repo。为了解决这个问题，我将展示 InitContainers 的概念，以及我们如何通过一点创造性来实现一些很酷的东西，比如在运行的容器中部署我们的代码。

### 入门指南

本指南取决于您是否拥有 OpenShift 安装权限，或者您是否已经使用 [Minishift](https://github.com/minishift/minishift) 或 oc cluster up 在本地机器上安装了 OpenShift。

一旦你有权登录。

```
oc login <your-url>
```

### 配置我们的工作空间

一旦 OpenShift 启动并运行，并且您已经登录，下一步就是创建一个项目:

```
oc new-project **my-project** 
```

### 图像

我们需要用我们需要的工具配置 Node.js，为了导入它，我们需要一个 [ImageStream](https://docs.openshift.com/enterprise/3.0/architecture/core_concepts/builds_and_image_streams.html#image-streams) 对象，它将抓取我们的图像，并使它可用于我们的项目。

```
oc import-image cvr-node:latest --from=docker.io/cvaldezr/nodejs --confirm 
```

这将抓取这个映像 [cvaldezr/ nodejs](http://cvaldezr / nodejs) 基于 [mhart/alpine](https://github.com/mhart/alpine-node) 映像这包括节点，NPM， **Nodemon，**和所有必要的工具来构建本地插件，这个映像只有 89MB，所以它将部署得非常快。

### 模板

接下来，我们需要为 Pod 获取一个模板定义，稍后我将更详细地解释这个结构。

```
curl -o pod.yml https://gist.githubusercontent.com/cesarvr/2dedd0bb912be441aa98b67e1ac4bcc6/raw/2cf75a5512014fd40086375d5a46c81940c53fc8/pod.yml 
```

一旦您得到这个文件，您需要修改第 12 行并添加您的图像的 URL，您可以通过执行以下操作来获得 URL:

```
oc get is #<DOCKER REPO is the url we need to copy>
```

这就是模板的样子，正如你所看到的，它很漂亮，也很短:

https://gist.github.com/cesarvr/2dedd0bb912be441aa98b67e1ac4bcc6

接下来要做的是使用我们的模板创建我们的 Pod。

```
oc create -f pod.yml
```

要检查状态，我们可以使用以下命令。

`oc get pods`

我们应该看到创建是成功的，如果不是，只需确保模板具有来自 ImageStream 的正确图像 URL，并且您有权限将它拉入您的项目。

### 写一些代码

现在是有趣的部分，让我们在 Node.js 中编写一个小的 hello world 服务器应用程序。

`const express = require('express')`

const app = express()

app.get('/'，(req，res) => res.send('Hello World！！!'))
`app.listen(8080, () => console.log('Example app listening on port 8080!'))`

将此文件保存为 **app.js** ，转到 package.json 并设置您的“main”属性，您将看到的模板配置正在寻找该属性来定位和执行您的应用程序的入口点，您可以更改和改进它以满足您的需求。

```
 {
  "name": "hello",
  "version": "1.0.0",
  "description": "",
**  "main": "app.js",**
  "scripts": {
  "test": "echo \"Error: no test specified\"exit 1",
  "start": "node app.js"
},
  "author": "",
  "license": "GPL",
  "dependencies": {
    "express": "^4.16.2"
  }
} 
```

使用`npm install express --save`安装依赖项，仅用于在我们的 package.json 中注册依赖项。

### 部署

首先，我们需要将文件发送到我们的 Pod，在我的例子中它被命名为 **node-dev** 。您可以使用`oc get pods`查看您的姓名。

`oc rsync -c folder . **node-dev**:app/`

暴露我们的吊舱。

`oc expose pod node-dev --port=8080`


现在访问您新创建的服务。
`oc get route -o wide`

`**node-dev-devel.127.0.0.1.nip.io **`

### ![](img/85076bc9ad7f9f23f82197c963adfc08.png)

### 修正

现在让我们改变一些事情。

```
const express = require('express')

const app = express()  app.get('/', (req, res) => res.send('**Hola Mundo**!!!')) app.listen(8080, () => console.log('Example app listening on port 8080!'))
```

完成修改后，到你的控制台写。

`oc rsync . node-dev:app/`

现在刷新你的浏览器。

![](img/52093a7395c3cc80c731f9eaf3af656b.png)

注意，在我们的例子中，我们没有使用-c 文件夹，这是因为我们现在的目标是运行时容器，稍后将对此进行更详细的解释。

### 现场演示

这只是一个小视频，演示了这个过程，并与 Pod 同步文件。

[https://www.youtube.com/embed/z3H2vq3wrg8?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/z3H2vq3wrg8?autoplay=0&start=0&rel=0)

### 刚刚发生了什么

为了解释发生了什么，让我们快速看一下模板。

```
apiVersion: v1
kind: Pod
metadata:
name: node-dev
labels:
app: node-js-dev
```

这里，我们定义了 pod 的名称和标签，没有什么特别有趣的。

#### 我们的 Node.js 运行时容器

```
spec:
containers: 
- name: nodejs
image: 172.30.1.1:5000/devel/cvr-node
command: ['/bin/sh', '-c']
args:
- cd /app/;
echo folder:$PWD;
npm install;
nodemon $(node -e "console.log(require('./package.json').main)") volumeMounts:
- mountPath: /app
name: app-volume
- mountPath: /.npm
name: npm-cache
ports:
- containerPort: 8080 
```

这是主窗格，你可以看到它使用了我们之前用`oc import-image`导入的图像。我还包含了一些启动 Pod 命令`sh -c`，它将使用`args`运行一些 shell 命令，基本上，它会转到 **app/** 文件夹**运行 npm** 安装并启动 **nodemon** ，但如果我们只是这样做并发布一个映像，它会立即崩溃，因为 nodemon 将无法找到任何东西，如果我们有办法等到我们的挂载点中有一些文件，我们就可以避免无限的崩溃循环。

#### 使用 InitContainer 的解决方案

因此，Pods 对象具有这种惊人的功能，称为 InitContainers，这意味着您可以有一个容器来为您做一些初始化工作，这在您想要一个运行轻量级容器和一个所需的完整编译工具的情况下非常有用。例如，如果您想要一个包含所有编译/构建工具的 InitContainer，然后想要一个运行时容器，其中只包含一个非常精简的容器，只包含要运行的基本内容。

```
 initContainers: # This is the init container will wait until app/ folder is in sync.
- name: folder
image: busybox
command: ['/bin/sh', '-c']
args: ['until [ "$(ls -A ./app/)" ]; do echo "waiting for user to push..."; sleep 2; done']
volumeMounts:
- mountPath: /app
name: app-volume
```

这就是我们的 InitContainer 的样子，我只选择了一个非常小的映像 **Busybox** ，并运行一个小脚本来停止 Pod 在 PodInit 状态下的执行。

![](img/e5b7989f69c819846b1e42e6f2dbe01e.png)

如果你很好奇，你可以通过执行`oc logs **-c folder** node-dev -f`来获得这个 Pod 的日志，你将每两秒钟看到这个跟踪`"waiting for user to push..."`，然后当你运行`oc rsync -c folder . **node-dev**:app/`时，你正在与 InitContainer 同步，通过这样做，条件`until [ "$(ls -A ./app/)" ];`将不再为真，它将终止与 InitContainer 相关联的 **sh** 命令。

![](img/0c04e359afc1274be68a5ce5e104578c.png)

### 结论

我在尝试寻找使用 Openshift/Kubernetes 的创造性方法时获得了很多乐趣，所以我希望您能发现该模板很有用，并且您可以根据自己的用例对其进行调整，甚至更好地对其进行改进。此外，我使用 Node.js 来实现这一点，因为我在日常工作中使用这种语言，但我认为用 Java 实现这一点没有任何问题。如果运行时容器只是一个在某个目录中等待 EAR、WAR(也许我有点过时了)的 JVM，并且每次文件系统改变时都热部署它，等等，那将会很酷。

此外，我想补充的是，这种方法不是水平扩展友好的，或者基本上你需要将代码推送到每个 Pod，因为在这个例子中，我只是使用了容器的文件系统。您可以通过将文件系统设置为 PVC(永久卷声明)来克服这一限制，然后在您的容器之间共享它，这有一些挑战，但我认为它可以工作，但这是另一篇文章。

关于 InitContainers 的更多信息。

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: October 21, 2022*