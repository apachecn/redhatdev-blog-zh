# 在 Kubernetes 和 Red Hat OpenShift 上封装和部署 Strapi 应用程序

> 原文：<https://developers.redhat.com/blog/2021/04/09/containerize-and-deploy-strapi-applications-on-kubernetes-and-red-hat-openshift>

Strapi 是领先的开源无头内容管理系统(CMS)。它是 100% [JavaScript](/topics/javascript) ，完全可定制，并且采用了开发者至上的方法。Strapi 为您提供了一个创建和管理网站所有资源的界面。然后，您可以构建一个前端，用您喜欢的工具和框架连接到您的 Strapi API。内容编辑者可以使用友好的管理面板来管理和分发内容。Strapi 也基于一个插件系统，这使得 CMS 具有灵活性和可扩展性。

一旦您使用 Strapi 的管理面板构建了您的资源，并设计了一个很好的前端来提供内容，您将需要在某个地方部署应用程序。本文向您展示了如何在一个 [Kubernetes](/topics/kubernetes) 或 [Red Hat OpenShift](/products/openshift/overview) 集群上部署一个 Strapi 应用程序。

## 步骤 1:设置开发环境

要在容器化的开发环境中使用 Strapi，您需要三个独立的容器:一个用于运行数据库，另一个用于 Strapi，还有一个用于前端。本节向您展示如何设置您将在开发中使用的三个容器。

### 初始设置

数据库和后端服务器必须能够通信。您可以使用 Docker 网络进行这种通信。使用以下命令创建您的网络:

```
$ docker network create strapi

```

您还需要三个文件夹来保存容器中的数据。下面是创建`/data`、`/app`和`/front`文件夹的命令:

```
$ mkdir ./data && mkdir ./app && mkdir ./front

```

### 创建数据库容器

要启动一个 Strapi 实例，您需要一个数据库来保存您的数据。在这个例子中，我们将使用一个运行在容器中的 MySQL 数据库服务器。这样就不需要经历安装 MySQL 的过程了。

要运行服务器，您可以使用带有`-d`参数的`docker run`命令在后台运行。包括以下参数:

*   *`--name`给容器命名。*
*   `-v`指定包含数据的文件夹，以便下次启动服务器时重复使用。
*   `-e`设置环境变量来配置数据库。

启动容器的命令应该如下所示:

```
$ docker run --rm -d --name strapi-db -v $(pwd)/data:/var/lib/mysql:z --network=strapi -e MYSQL_DATABASE=strapi -e MYSQL_USER=strapi -e MYSQL_PASSWORD=strapi -e MYSQL_ROOT_PASSWORD=strapi-admin mysql:5.7

```

注意，我们使用`--network`参数将数据库容器连接到我们之前创建的网络。

执行这个命令后，尝试一个`docker ps`来验证容器已经启动。

### 创建后端容器

现在您已经配置了数据库，您可以启动您的`strapi`实例，它将从一个容器中运行。这一次，您将使用`strapi/strapi`基础映像。您仍然可以使用`-d`参数在后台运行它，并使用`--name`来命名您的容器。一定要将 Strapi 容器添加到与数据库相同的网络中。

您还应该将本地的`/app`文件夹映射到`/srv/app`:

*   使用`-v`参数，这样您就可以使用机器上的本地文件夹持久保存 Strapi 创建的文件。
*   将操作系统上的一个端口映射到容器内部的访问端口 1337。如果您使用端口 8080，连接到 Strapi 管理控制台的地址将是`localhost:8080`。
*   使用环境变量配置 Strapi 以使用您在上一步中启动的数据库。

下面是启动 Strapi 后端容器的命令:

```
$ docker run --rm -d --name strapi-dev -p 8080:1337 -v $(pwd)/app:/srv/app:z --network=strapi -e DATABASE_CLIENT=mysql -e DATABASE_NAME=strapi -e DATABASE_HOST=strapi-db -e DATABASE_PORT=3306 -e DATABASE_USERNAME=strapi -e DATABASE_PASSWORD=strapi strapi/strapi

```

如果 Strapi 在您映射的本地文件系统中找不到任何文件，它将自动创建一个新的 Strapi 服务器实例。这可能需要几分钟时间。您可以使用`docker logs`来关注应用程序的创建状态:

```
$ docker logs -f strapi-dev

```

如果您想停止控制台中的日志，请输入 **Ctrl-C** 。

一旦您看到一条消息，说明您的 Strapi 服务器已经启动，您就可以转到[http://localhost:8080/admin](http://localhost:8080/admin)来创建您的 admin 用户。

创建 admin 用户后，继续创建一个新的内容类型，并将其公开。对于下一步要处理的内容，为**帖子**创建一个**内容类型**。它将有四个字段:**标题**、**作者**(与**用户**的关系)、**发布日期**和**内容**，如图 1 所示。

[![The Posts content type has four fields: Title, Author, Publish_date, and Content.](img/1aaa8c32be27f6a23351e77f2f793ea3.png "Screenshot_20210408_120444")](/sites/default/files/blog/2021/04/Screenshot_20210408_120444.png)

Figure 1: The new content type with four fields for posts.

**注意**:参见 [Strapi](https://strapi.io) 团队的[视频](https://www.youtube.com/watch?v=VC9X9O5OFyc)，获得关于创建新内容类型的完整教程。

### 创建前端容器

接下来，您将创建一个前端。这个用户界面(UI)将由一个简单的 HTML 文件组成，该文件获取 Strapi 应用程序编程接口(api)数据并将其显示在页面上。

我们将使用 Nginx 服务器来显示内容。您可以像启动其他两个容器一样启动容器。这一次，将容器中的端口 80 映射到本地机器上的端口 8888，并挂载`/front`文件夹以映射到容器中的`/usr/share/nginx/html`。`/front`文件夹是 Nginx 提供文件的默认文件夹:

```
$ docker run --rm -d --name strapi-front -p 8888:80 -v $(pwd)/front:/usr/share/nginx/html:z nginx:1.17

```

现在，继续创建您的前端应用程序。您可能会使用 React、VueJS 或 Angular 应用程序，但我们将使用一个简单的 HTML 文件进行演示。该文件将从 Strapi API 执行一个`fetch`来下载数据，然后使用 JavaScript 在页面上创建必要的元素。

HTML 页面将有一个单独的`div`，JavaScript 代码在这里附加 API 的内容。在`/front`文件夹中创建以下`index.html`文件:

```
<body>
  <div id="content"></div>
</body>

```

您将需要添加一个`script`标签来包含一个配置文件，这将使得稍后覆盖您的 Strapi API 位置更加容易。在`index.html`内添加以下内容:

```
<script type="text/javascript" src="config.js">

```

`front/config.js`文件应该创建一个全局常量，配置如下:

```
const config = {
  BASE_URL: "http://localhost:8080"
}

```

最后，在`index.html`文件中，添加另一个`script`标记，它包含下载内容并在页面上显示的代码:

```
window.addEventListener("DOMContentLoaded", e => {
  console.log("Loading content from Strapi");

  const BASE_URL = config.BASE_URL;

  const BLOG_POSTS_URL = `${BASE_URL}/posts`;

  fetch(BLOG_POSTS_URL).then(resp => resp.json()).then(posts => {
    for(let i = 0; i < posts.length; i++) {
      let postData = posts[i];
      let post = document.createElement("div");
      let title = document.createElement("h2");
      title.innerText = postData.title;
      let author = document.createElement("h3");
      author.innerText = `${postData.author.firstname} ${postData.author.lastname} -- ${postData.publish_date}`;
      let content = document.createElement("div");
      content.innerText = postData.content;
      post.appendChild(title);
      post.appendChild(author);
      post.appendChild(content);
      document.querySelector("#content").appendChild(post);
    }
  });
});

```

现在您已经创建了所有的文件，请转到 [http://localhost:8888](http://localhost:8888/) 查看您的应用程序。您应该看到您的奇特 UI 提供来自 Strapi 的内容。

## 步骤 2:设置生产环境

当您准备好部署应用程序时，您需要创建自己的容器，其中包含所有必需的文件和数据。这些容器将在网络上运行。

对于每个容器，您需要创建一个 docker 文件。您将使用 docker 文件来创建包含实际内容的容器。然后，将容器部署到 Kubernetes 或 OpenShift。

### 创建数据库容器

很有可能您已经有了一个生产数据库，并且您可能不想覆盖它的内容。因此，您将使用在生产数据库开发中使用的相同的默认 MySQL 映像。如果您想稍后导入 SQL 内容，您可以使用 Docker 在您的数据库上运行一个`mysqldump`命令:

```
$ docker exec strapi-db /bin/bash -c 'mysqldump strapi -ustrapi -pstrapi' | tee strapi-db.sql

```

如果需要的话，这个文件将被导入到生产数据库中。

**注意**:`mysqldump`命令使用`tee`将内容复制到一个文件中。如果没有`tee`命令，可以将`docker`命令的输出复制到一个名为`strapi-db.sql`的文件中。

### 创建后端容器

接下来，您将创建一个`Dockefile.back`来为后端构建您的容器。

从`strapi`基础图像`FROM strapi/base`开始。将工作目录更改为`/opt/app`，并将所有本地文件复制到容器中。接下来，公开端口 1337 并设置所有的环境变量。不要忘记为`NODE_ENV=production`添加一个环境变量。最后，执行`yarn build`来构建所有的生产资源，并在容器启动后使用`CMD`命令来启动后端。

**注意**:关于使用 Strapi 基础镜像的更多信息，请参见 GitHub 上的 [Strapi 文档。](https://github.com/strapi/strapi-docker#how-to-use-strapibase)

```
FROM strapi/base
WORKDIR /opt/app
COPY ./app/package.json ./
COPY ./app/yarn.lock ./
RUN yarn install
COPY ./app .
ENV NODE_ENV production
ENV DATABASE_CLIENT=mysql
ENV DATABASE_NAME=strapi
ENV DATABASE_HOST=strapi-db
ENV DATABASE_PORT=3306
ENV DATABASE_USERNAME=strapi
ENV DATABASE_PASSWORD=strapi
RUN yarn build
EXPOSE 1337
CMD ["yarn", "start"]

```

### 创建前端容器

您必须编写一些 bash 脚本来使用环境变量来指定您的 Strapi 服务器的 URL。

**注意**:参见我的【JavaScript 前端容器最佳实践了解更多关于前端容器使用环境变量的信息。

首先，从`nginx:1.17`基础映像开始，将工作目录改为`/usr/share/nginx/html`。在那里，将本地系统中的所有文件复制到容器中。

下一步涉及使用`sed`将`BASE_URL`值更改为`$BASE_URL`。然后，您将把结果通过管道传输到一个名为`config.new.js`的新文件中，并将该文件重命名为`config.js`，覆盖原来的文件。

容器内的结果是一个新的`config.js`文件，如下图所示。请注意，本地文件系统中的原始文件保持不变:

```
const config = {
  BASE_URL: "$BASE_URL"
}

```

最后，您需要使用`envsubst`将`$BASE_URL`的值更改为环境变量的实际值。在`ENTRYPOINT`中进行以下更新，这样当有人发出 Docker 运行时，这些更改就会发生:

*   使用一个`cat`命令将`config.js`文件传输到`envsubst`。
*   将输出传输到`tee`以创建一个新的`config.new.js`文件，并重命名该文件以覆盖之前的文件。
*   使用`nginx -g 'daemon off;'`命令启动 Nginx 服务器:

    ```
    FROM nginx:1.17
    WORKDIR /usr/share/nginx/html
    COPY ./front/*.* ./
    RUN sed s/BASE_URL\:\ \"[a-zA-Z0-9:\/]*\"/BASE_URL\:\ \"\$BASE_URL\"/g config.js > config.new.js && mv config.new.js config.js
    ENTRYPOINT cat config.js |  envsubst | tee config.new.js && mv config.new.js config.js && nginx -g 'daemon off;'

    ```

更新入口点而不是`RUN`允许您根据容器运行的位置为基本 URL 指定不同的值。

### 建造容器

现在您已经准备好了所有的 docker 文件，您可以构建容器并将它们推到您最喜欢的图像注册中心。不要忘记更改图像的名称，以使用您在该注册表中的用户名:

```
$ docker build -t $DOCKER_USERNAME/strapi-front -f Dockerfile.front .
$ docker build -t $DOCKER_USERNAME/strapi-back -f Dockerfile.back .
$ docker push $DOCKER_USERNAME/strapi-front
$ docker push $DOCKER_USERNAME/strapi-back

```

## 步骤 3:打包并运行应用程序

现在您已经有了包含所有代码和数据的容器，您已经准备好将容器部署到某个地方了。我们将使用 Docker 和 Docker Compose 来运行应用程序，并使用 Kubernetes 或 OpenShift 集群来部署它。

### 用 Docker 打包并运行应用程序

如果您想要运行这个应用程序，您可以像在生产环境中一样启动所有容器。

启动容器的命令类似于您在开发模式中使用的命令，但是*带有*挂载的卷，而*没有*环境变量。我们处理 docker 文件中的源代码和环境变量。请注意，我们添加了一个环境变量来指定用于启动前端的 Strapi API 的位置:

```
$ docker run --rm -d --name strapi-db -v $(pwd)/data:/var/lib/mysql:z --network=strapi -e MYSQL_DATABASE=strapi -e MYSQL_USER=strapi -e MYSQL_PASSWORD=strapi -e MYSQL_ROOT_PASSWORD=strapi-admin mysql:5.7
$ docker run --rm -d --name strapi -p 1337:1337 --network=strapi $DOCKER_USERNAME/strapi-back
$ docker run --rm -d --name strapi-front -p 8080:80 -e BASE_URL=http://localhost:1337 $DOCKER_USERNAME/strapi-front

```

### 用 Docker Compose 打包并运行应用程序

如果您想与他人共享您的应用程序代码和配置，您可以为他们提供一个`docker-compose.yaml`文件。这个工具允许您一次管理多个容器，而不需要多个 bash 命令:

```
version: '3'
services:
  strapi-db:
    image: mysql:5.7
    volumes:
      - ./data:/var/lib/mysql
    networks:
      - strapi
  strapi-back:
    image: $DOCKER_USERNAME/strapi-back
    ports:
      - '1337:1337'
    networks:
      - strapi
  strapi-front:
    image: $DOCKER_USERNAME/strapi-front
    ports:
      - '8080:80'
    environment:
      BASE_URL: http://localhost:1337
networks:
  strapi:

```

## 步骤 4:部署应用程序

一旦创建了所有的容器，就可以将应用程序部署到 Kubernetes 或 OpenShift 集群中。我将向您展示如何做到这两点。

### 在 Kubernetes 上部署应用程序

在 Kubernetes 集群中部署应用程序之前，您需要使用 YAML 文件来创建所有必要的资产。有关这些资产的更多详细信息，请参见[](http://kubernetesbyexample.com/)*。为了测试部署，您可以使用一个较小版本的 Kubernetes 在您自己的机器上本地运行。我在下面的例子中使用了[迷你库贝](https://kubernetes.io/docs/tutorials/hello-minikube/)。*

 *#### 部署数据库

[持久卷](https://kubernetesbyexample.com/pv/) (PVs)和持久卷声明(PVC)的设置因云提供商而异。因此，本例中的数据库不会持久存储数据。有关如何持久保存数据的更多信息，请查看您的云提供商的文档。

对于数据库，我们需要创建一个[部署](https://kubernetesbyexample.com/deployments/)。您将首先创建一个描述您的部署的 YAML 文件。您可以给它一个名称，并且在规范中，您将为 pod 创建一个模板。每个 pod 都有一个单独的容器，也就是您已经推送到您的注册表中的容器。以下是该示例的部署(`deploy-db.yaml`):

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: strapi-db
spec:
  selector:
    matchLabels:
      component: db
  template:
    metadata:
      labels:
        component: db
    spec:
      containers:
      - name: strapi-db
        image: mysql:5.7
        env:
          - name: MYSQL_DATABASE
            value: strapi
          - name: MYSQL_USER
            value: strapi
          - name: MYSQL_PASSWORD
            value: strapi
          - name: MYSQL_ROOT_PASSWORD
            value: strapi-admin

```

一旦有了文件，就可以使用`kubectl`将它应用到集群中:

```
$ kubectl apply -f ./deploy-db.yaml

```

#### 部署后端

您的后端需要能够找到集群内部的 pod，因此您需要创建一个[服务](https://kubernetesbyexample.com/services/)来公开每个 pod。我们在这里使用默认值，所以您可以使用`kubectl`来创建这个服务:

```
$ kubectl expose deployment strapi-db --port 3306

```

如果要从开发环境 SQL 导入数据，可以运行以下命令:

```
$ kubectl cp ./strapi-db.sql $(kubectl get pod -l component=db | awk 'NR>1 {print $1}'):/tmp/strapi-db.sql
$ kubectl exec -t $(kubectl get pod -l component=db | awk 'NR>1 {print $1}') -- /bin/bash -c 'mysql strapi -ustrapi -pstrapi < /tmp/strapi-db.sql'

```

这些命令将 SQL 文件复制到 pods，然后运行 MySQL 命令在数据库中运行它。

您还可以为应用程序的后端和前端部分创建部署。除了名称、标签和容器映像之外，Strapi 后端(`deploy-back.yaml`)与数据库部署相同:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: strapi-back
spec:
  selector:
    matchLabels:
      app: strapi
      component: back
  template:
    metadata:
      labels:
        app: strapi
        component: back
    spec:
      containers:
      - name: strapi-back
        image: joellord/strapi-back

```

#### 部署前端

前端(`deploy-front.yaml`)使用与后端类似的结构，但是您还需要为后端的`BASE_URL`设置环境变量。现在，只需将该变量的值设置为`/api`。您还需要将容器暴露给端口 80，以便它最终对外界可用:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: strapi-front
spec:
  selector:
    matchLabels:
      component: front
  template:
    metadata:
      labels:
        component: front
    spec:
      containers:
      - name: front
        image: joellord/strapi-front
        ports:
          - containerPort: 80
        env:
          - name: BASE_URL
            value: /api

```

#### 在集群中创建和公开应用程序服务

现在您已经创建了部署文件，您可以将它们应用到您的集群并为每个文件创建服务:

```
$ kubectl apply -f ./deploy-back.yaml
$ kubectl apply -f ./deploy-front.yaml
$ kubectl expose deployment strapi-back --port 1337
$ kubectl expose deployment strapi-front --port 80

```

现在，一切都在集群中运行。您只需要向外界公开前端和后端服务。为此，您将使用一个[入口](https://kubernetes.io/docs/concepts/services-networking/ingress/)。

在这里，您将创建一个入口，将前端公开为默认服务。默认情况下，对集群的任何传入请求都会转到前端。您还将添加一个规则，将发送到`/api/*`的任何流量重定向到后端服务。当请求被发送到该服务时，它将被重写，以删除 URL 的`/api`部分。我们将在元数据中添加一个 Nginx 注释来实现这一改变。这里是`ingress.yaml`文件:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: main-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - http:
      paths:
        - path: /api(/|$)(.*)
          pathType: Prefix
          back end:
            service:
              name: strapi-back
              port:
                number: 1337
        - path: /()(.*)
          pathType: Prefix
          backend:
            service:
              name: strapi-front
              port:
                number: 80

```

继续将该文件应用到您的集群。如果您正在使用 Minikube 并且以前从未使用过 ingress，您可能需要启用以下加载项:

```
# For minikube users
$ minikube addons enable ingress

$ kubectl apply -f ./ingress.yaml

```

现在，您已经拥有了在 Kubernetes 集群中运行 Strapi 应用程序所需的一切。将浏览器指向集群 URL，您应该会看到集群中运行的完整应用程序。如果您正在使用 Minikube，您可以使用命令`minikube ip`来获取您的集群的地址。

### 在 OpenShift 上部署应用程序

在 OpenShift 上部署应用程序甚至比在 Kubernetes 集群中部署更容易。

在这种情况下，您可以使用[开发人员沙箱](/developer-sandbox)来测试您的部署，这使您可以免费访问 OpenShift 集群 14 天。

#### 从映像创建部署

用于管理集群的命令行界面(CLI)工具(`oc`)可以直接从映像创建部署。要部署您的应用程序，请输入:

```
$ oc new-app mysql:5.7 MYSQL_USER=strapi MYSQL_PASSWORD=strapi MYSQL_DATABASE=strapi -l component=db --name strapi-db
$ oc new-app joellord/strapi-back-openshift --name strapi-back
$ oc new-app joellord/strapi-front-openshift --name strapi-front

```

**注意**:open shift 上的图像需要以非 root 用户身份运行。参见我的[前端最佳实践指南](https://github.com/joellord/frontend-containers)，了解更多关于非根映像的信息。这个项目使用的 docker 文件可以在本文的的 [Git 存储库中的`Dockerfile.rootless.back`和`Dockerfile.rootless.front`下找到。](https://github.com/joellord/strapi)

Seed your database with the data that you exported earlier. This data should be in your current working directory and have the name `strapi-db.sql`.

```
$ oc exec -it $(oc get pods -l component=db | awk 'NR>1 {print $1}') -c strapi-db -- bash -c 'mysql -ustrapi -pstrapi strapi' < ./strapi-db.sql
```

#### 公开应用程序

接下来，您将希望向外界公开应用程序。OpenShift 为此提供了一个简洁的对象`Route`，您可以在 OpenShift CLI 中使用它。使用`oc expose`命令将后端和前端暴露给外界:

```
$ oc expose service strapi-back
$ oc expose service strapi-front --port=8080

```

既然您的后端已经公开，那么您需要将您的前端环境变量设置为后端路由。首先获取 Strapi API 的公共路由:

```
$ oc get routes

```

您应该会看到到目前为止您已经创建的所有路线。您可以将后端路由存储在一个变量中，然后使用`oc set env`将其设置为一个环境变量:

```
$ export BACKEND_ROUTE=$(oc get routes | grep strapi-back | awk '{print $2}')
$ oc set env deployment/strapi-front BASE_URL=http://$BACKEND_ROUTE

```

现在，您可以使用`strapi-front`服务的路由来访问您的 Strapi 应用程序。

## 摘要

当您准备将您的 Strapi 应用程序投入生产时，第一步是将您的整个设置容器化。一旦完成了这些，就可以在 Kubernetes 中部署这些容器了。您还看到了将 Strapi 应用程序部署到 OpenShift 是多么容易。

*Last updated: October 14, 2022**