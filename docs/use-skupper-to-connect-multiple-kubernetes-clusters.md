# 使用 skipper 连接多个 kubernetes 集群

> 原文：<https://developers.redhat.com/blog/2021/04/20/use-skupper-to-connect-multiple-kubernetes-clusters>

Skupper 是第 7 层服务互连，支持跨 [Kubernetes](/topics/kubernetes/) 集群的多云通信。在开发过程中，有几个原因可能需要在本地集群和远程集群之间进行通信:

*   一个服务部署在远程集群上，您希望通过本地集群来使用它。
*   工作负载消耗大量资源，在给定可用资源的情况下，将其部署在本地集群上是不可行的。
*   现有的 stage/test 数据库服务部署在远程集群上，您的工作负载需要连接到它。

关于 Skupper 最好的事情之一是，用户不需要拥有集群的管理特权就可以部署它。参见文章 *[Skupper.io:让您的服务跨 Kubernetes 集群](https://developers.redhat.com/blog/2020/01/01/skupper-io-let-your-services-communicate-across-kubernetes-clusters/)* 进行通信，了解关于这个开源项目的更多信息。

本文将向您展示如何使用 Skupper 将远程集群服务与使用[Red Hat code ready Containers](/products/codeready-containers/overview)和用于 Red Hat OpenShift 的[开发者沙箱的本地集群连接起来。如需更详细的概述，请参阅](/developer-sandbox) [Skupper 入门指南](https://skupper.io/start/index.html)。

## 关于这个例子

在这个例子中，我将[Red Hat code ready Containers(CRC)](/products/codeready-containers/overview)用于我的本地集群。CodeReady Containers 是一个开发者工具，可以让你在 [Red Hat OpenShift 4](/products/openshift/overview) 上创建本地 Kubernetes 集群。我有另一个来自红帽 OpenShift 的[开发者沙箱的集群，我可以从任何地方访问它。](/developer-sandbox)

## 步骤 1:安装 Skupper 命令行工具

首先，在您的 Linux 系统上安装 Skupper 命令行工具。

**注意**:如果您使用不同的平台，请参考[入门指南](https://skupper.io/start/index.html#step-1-install-the-skupper-command-line-tool-in-your-environment)中的说明。

```
$ curl -fL https://github.com/skupperproject/skupper/releases/download/0.3.2/skupper-cli-0.3.2-linux-amd64.tgz | tar -xzf -

$ sudo mv skupper /usr/local/bin

$ which skupper 
/usr/local/bin/skupper

$ skupper --version
skupper version 0.3.2

```

## 步骤 2:配置对多个名称空间的访问

正如[入门指南](https://skupper.io/start/index.html#step-2-configure-access-to-multiple-namespaces)所描述的那样，`skupper`命令使用`kubeconfig`文件和当前上下文来选择它运行的名称空间。您必须为每个名称空间使用不同的`kubeconfig`或上下文，所以最好使用不同的控制台终端、选项卡或会话。

为每个命名空间启动控制台会话，并登录到集群。

**CodeReady 容器控制台:**

```
$ export KUBECONFIG=$HOME/.kube/config-crc

$ oc login -u developer -p developer https://api.crc.testing:6443
Login successful.

$ oc config get-contexts
CURRENT   NAME                              CLUSTER                AUTHINFO    NAMESPACE
*         /api-crc-testing:6443/developer   api-crc-testing:6443   developer

```

**开发者沙盒控制台:**

```
$ export KUBECONFIG=$HOME/.kube/config-devsandbox

$ oc login --token=<token> --server=https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443
Logged into "https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443" as "prkumar" using the token provided.
You have access to the following projects and can switch between them with ' project <projectname>':
* prkumar-code
prkumar-dev
prkumar-stage

$ oc config get-contexts
CURRENT   NAME                                                                 CLUSTER                                         AUTHINFO   NAMESPACE
*         prkumar-code/api-sandbox-m2-ll9k-p1-openshiftapps-com:6443/prkumar   api-sandbox-m2-ll9k-p1-openshiftapps-com:6443   prkumar    prkumar-code

```

接下来，在 CodeReady 容器上创建一个新项目。出于演示的目的，我们将使用`prkumar-code`作为开发人员沙箱端的默认上下文，因为我们没有创建新项目的权限。

**CodeReady 容器控制台:**

```
$ oc new-project demo
Now using project "demo" on server "https://api.crc.testing:6443".

```

## 步骤 3:在每个名称空间中安装 Skupper 路由器

运行`skupper init`命令在每个名称空间中安装路由器。

**CodeReady 容器控制台:**

```
$ skupper init --cluster-local
Skupper is now installed in namespace 'demo'.  Use 'skupper status' to get more information.
```

**开发者沙盒控制台:**

```
$ skupper init
Skupper is now installed in namespace 'prkumar-code'.  Use 'skupper status' to get more information.
```

现在，检查路由是否安装成功。

**CodeReady 容器控制台:**

```
$ skupper status
Skupper is enabled for namespace "prkumar-code" in interior mode. It is not connected to any other sites. It has no exposed services.

```

## 步骤 4:连接名称空间

生成[连接令牌](https://skupper.io/start/index.html#step-4-link-your-namespaces)。

**开发者沙盒控制台:**

```
$ skupper connection-token $HOME/secret.yaml
Connection token written to /home/prkumar/secret.yaml

```

然后，使用令牌连接到您的 CRC 群集。

**CodeReady 容器控制台:**

```
$ skupper connect $HOME/secret.yaml
Skupper configured to connect to skupper-inter-router-prkumar-code.apps.sandbox-m2.ll9k.p1.openshiftapps.com:443 (name=conn1)
```

## 步骤 5:公开您的前端和后端服务

这里我们将使用一个后端和一个前端服务，正如在[入门指南](https://skupper.io/start/index.html#step-5-expose-your-services)中提到的。首先，我们将在开发人员沙箱上部署后端服务，并将前端服务部署到 CRC。然后，我们将连接后端和前端。

**CodeReady 容器控制台:**

```
$ oc  create deployment hello-world-frontend --image quay.io/skupper/hello-world-frontend
deployment.apps/hello-world-frontend created

```

**开发者沙盒控制台:**

```
$ oc create deployment hello-world-backend --image quay.io/skupper/hello-world-backend
deployment.apps/hello-world-backend created

```

使用`skipper expose`命令公开后端服务，这样它将在您的 CRC 集群上可用。

**开发者沙盒控制台:**

```
$ skupper expose deployment hello-world-backend --port 8080 --protocol http
```

正如您所看到的，一旦您使用 Skupper 公开了一个服务，它对 CRC 集群是可见的。

**CodeReady 容器控制台:**

```
$ oc get svc
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)               AGE

hello-world-backend   ClusterIP   10.217.5.209   <none>        8080/TCP              96s

```

最后，测试您的前端应用程序，以验证它可以与后端服务通信。

**CodeReady 容器控制台:**

```
$ oc expose deployment hello-world-frontend --port 8080
service/hello-world-frontend exposed

$ oc expose svc hello-world-frontend
route.route.openshift.io/hello-world-frontend exposed

$ curl hello-world-frontend-demo.apps-crc.testing
I am the frontend.  The backend says 'Hello from hello-world-backend-7dfb45b98d-j8q8t (2)'.

```

## 结论

正如我们在本文中看到的，当两个集群需要连接时，Skupper 就派上了用场，CodeReady Containers 为 OpenShift 提供了很好的本地体验。快乐的偷懒！

*Last updated: October 14, 2022*