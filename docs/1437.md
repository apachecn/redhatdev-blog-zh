# 开始撰写

> 原文：<https://developers.redhat.com/blog/2017/08/02/getting-started-with-kompose>

我们之前在这里写过 kom pose[，当时它还年轻到 0.1.0。这篇博文将展示 Kompose 现在的立场。](https://developers.redhat.com/blog/2016/12/01/kompose-up-openshift-and-kubernetes/)

Kompose 是一个将 Docker 合成文件转换为 Kubernetes 或 OpenShift 工件的工具。Kompose 最初是由 Skippbox(现在是 Bitnami 的一部分)作为 Kubernetes 用户的入职工具而启动的。它继续接受来自谷歌和红帽的早期捐款。

Kompose 已经从 Kubernetes 孵化器毕业，我们达到了 1.0.0 的里程碑，因此它现在正式成为 Kubernetes 社区项目的一部分。

让我们看看 Kompose 到底是怎么回事！这里有一个 Kubernetes 社区使用的标准例子，由一个 web 界面(Go)和一个数据库集群(Redis)组成。

让我们用一个官方的例子！

```
version: "2"
services:
redis-master:
 image: grc.io/google_containers/redis:e2e
 ports:
 -"6379"
redis-slave:
 image: gcr.io/google_samples/gb-redisslave:v1
 ports:
  - "6379"
  environment:
  - GET_HOSTS_FROM=dns
frontend:
 image: gcr.io/google-samples/gb-frontend:v4
 ports:
 - 80:80
 environment:
 -GET_HOSTS_FROM=dns
 labels:
  kompose.service.type:LoadBalancer
```

### Kompose 的特点:

以下部分讨论 Kompose 的各种功能:

#### 直接部署 Kubernetes 工件

以下示例演示了如何运行“kompose up”来将应用程序部署到 kubernetes 集群。此示例将使用*minishift*或*minikube*在本地测试它。

```
$ kompose up
```

信息我们将为您的 Dockerized 应用程序创建 Kubernetes 部署、服务和持久卷声明。如果您需要不同种类的资源，请使用“kompose convert”和“kubectl create - f”命令。

```
INFO Deploying application in "default" namespace
INFO Successfully created Service: frontend
INFO Successfully created Service: redis-master
INFO Successfully created Service: redis-slave
INFO Successfully created Deployment: frontend
INFO Successfully created Deployment: redis-master
INFO Successfully created Deployment: redis-slave

Your application has been deployed to Kubernetes. You can run 'kubetcl get deployment, svc, pods, pvc' for details.
```

### 生成 Kubernetes 工件

如果您想简单地生成工件而不是立即部署，您可以运行“kompose convert ”,如下所示:

```
$ kompose convert
INFO Kubernetes file "frontend-service.yaml" created
INFO Kubernetes file "redis-master-service.yaml" created
```

您可以在以后需要时使用以下工具进行部署:

```
$ kubectl create -f frontend-service.yaml, redis-master-service.yaml, redis-slave-service.yaml, frontend-deployment.yaml, redis-master-deployment.yaml, redis-slave-deployment.yaml
service "frontend" created
service "redis-master" created
```

#### **部署/生成 OpenShift 工件**

OpenShift 是 Kubernetes 的另一个发行版，也是 Kompose 的支持提供者之一。您可以通过提供命令行参数'- provider': 在 OpenShift 上部署工件

```
$ kompose up --provider openshift

```

您可以使用以下方式生成工件:

```
$ kompose convert --provider openshift

```

#### **将输出工件存放到特定位置**

如果您需要将输出工件存储到某个位置，使用命令行参数'-o '，如下所示。

```
$ kompose convert -o out/
INFO Kubernetes file "out/frontend-service.yaml" created
INFO Kubernetes file "out/redis-master-service.yaml" created
```

#### 复制品

您可以使用命令行参数'- replicas '来指定副本的数量。

```
$ kompose convert --replicas=4

```

#### **删除已部署的清单**

您可以删除实例化的工件，如服务、部署等。如下:

```
$ Kompose down
INFO Deleting application id=n "default" namespace
INFO Successfully deleted Service: frontend
INFO Successfully deleted Service: redis-master
INFO Successfully deleted Service: redis-alive
INFO Successfully deleted Deployment: frontend
INFO Successfully deleted Deployment: redis-master
INFO Successfully deleted ployment: redis-slave
```

#### 标签

到目前为止，并不是所有的 Docker 组合键都可以映射到 Kubernetes 工件。因此，对于服务类型和入口，我们定义了两种类型的 Kompose 特定标签，如下:

```
'kompose.service.expose' defines if service needs to be made accessible from outside of a cluster
labels:
 Kompose.service.expose: true
 Kompose.service.expose: "example.com"
'kompose.service.type' defines types of services to be created
labels:
Kompse.service.type: nodeport # or clusterip or loadbalancer
```

## **Kompose 1.0.0**

以下章节讨论了 Kompose 1.0.0 对 Docker 构建和推送以及 Docker 编写版本 3 的支持。

一些 Docker 合成文件包含“build”键，这意味着 Docker 文件在代码库中。当你这样做时，“docker-compose build”或“docker-compose up”，它应该是建立 docker 映像。

以前,“构建”只支持 OpenShift 提供程序，而忽略了 Kubernetes 提供程序。 在 1.0.0 版本中，Kompose 支持为 Kubernetes 和 OpenShift 提供者构建容器映像。

```
For example, in case our Docker compose file is:
   version: "3"
services:
   foo:
       build: "./build"
       image" docker.io/foo/bar
```

它将简单地生成如下工件(缺省值'- build '标志为 none):

```
 $ kompose convert
 INFO file "foo-service.yaml" created
 INFO file "foo-deployment.yaml created
```

而在 OpenShift 提供程序的情况下，使用以下命令生成 buildconfig:

```
$ kompose convert --provider=openshift --build build-config
INFO Buildconfig using git@github.com:surajnarwade/Kompose.git::master as a source.
INFO file "foo-service.yaml" created
INFO file "foo-deploymentconfig.yaml" created
INFO file "foo-imagestream.yaml" created
INFO file "foo-buildconfig.yaml" created

```

当您运行“kompose up”时，图像的本地构建发生并被推送到各自的 docker 注册表，其凭证存储在“~/”中。用户主目录中的 docker/config.json '文件。('- build '标志设置为本地)

您可以按如下方式部署它:

```
$ kompose up
INFO Build key detected. Attempting to build and push image 'docker.io/foo/bar'
INFO Building image 'docker.io/foo/bar' from directory 'build'
INFO Image 'docker.io./foo/bar' from directory 'build' built successfully
INFO Pushing image 'foo/bar:latest' to registry 'docker.io'
INFO Attempting authentication credentials 'https://index.docker.io/v1/
INFO Successfully pushed image 'foo/bar:latest' to registry 'docker.io'
INFO We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. If you need a different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead.
INFO Deploying application in "default" namespace
INFO Successfully created Service: foo
INFO Successfully created Deployment: foo
Your application has been deployed to Kubernetes. You can run 'kubectl get deployment, svc, pods, pvc' for details.

```

如果您已经有了所需的映像，您需要使用'- build '参数禁用映像的构建和推送:

```
$ kompose up --build none
```

### **Docker 撰写版本 3 支持:**

从 1.0.0 开始，我们为 Docker Compose 版本 3 提供支持。

这里有一个使用我们默认示例的演示。[(](https://github.com/kubernetes/Kompose/blob/master/examples/docker-compose-v3.yaml)[https://github . com/kubernetes/kom pose/blob/master/examples/docker-compose-counter-v3 . YAML](https://github.com/kubernetes/kompose/blob/master/examples/docker-compose-counter-v3.yaml))

```
version: "3"
services:
web:
 image: tuna/docker-counter23
 ports:
   - "5000:5000"
 deploy:
   restart_policy:
     condition: any
 labels:
   kompose.service.type: Nodeport
redis:
 image: redis:3.0
 deploy:
    replicas: 1
 ports:
   - "6379"
```

这是我们添加的其他东西，包括错误修复和新的密钥，如“副本”，“重启策略”等。

以前，我们必须使用“- replicas”标志来提及“副本”，但现在有了 Docker Compose 版本 3 的支持，我们可以在 Docker Compose 文件本身中提及副本。

试试看吧:)

* * *

**下载 Kubernetes** [**备忘单**](https://developers.redhat.com/promotions/kubernetes-cheatsheet/)****跨主机集群自动部署、扩展和操作应用容器，提供以容器为中心的基础设施。****

***Last updated: September 3, 2019***