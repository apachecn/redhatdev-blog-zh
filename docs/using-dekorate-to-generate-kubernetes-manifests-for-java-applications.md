# 使用 de Korea 为 Java 应用程序生成 Kubernetes 清单

> 原文：<https://developers.redhat.com/blog/2021/03/17/using-dekorate-to-generate-kubernetes-manifests-for-java-applications>

要在 [Kubernetes](/topics/kubernetes/) 或 [Red Hat OpenShift](/products/openshift/overview) 上部署应用程序，首先需要创建[对象](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)，以允许平台从容器映像安装应用程序。然后，您需要使用一个 [pod](https://kubernetes.io/docs/concepts/workloads/pods/) 启动应用程序，并将其公开为一个具有静态 IP 地址的服务。做所有这些可能是乏味的，但是有一些方法可以简化这个过程。

Kubernetes 遵循一个*声明模型*，这意味着用户声明所需的应用程序状态，集群进行调整以匹配。开发人员使用名为*清单*的文件来描述期望的状态。清单通常在 YAML 或 JSON 文件中定义，通过其 [REST API 端点](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)与服务器通信。

对象格式很复杂，有许多字段需要处理。使用工具来帮助创建清单是一个好主意。如果您正在部署 [Java](/topics/enterprise-java/) 应用程序，请考虑使用 [Dekorate](http://dekorate.io) 。它不仅会简化您作为开发人员的工作，而且还会在您采用 Kubernetes 时拉平您的学习曲线。

在本文中，我们将使用 Dekorate 为通用 Java 应用程序生成 Kubernetes 和 OpenShift 清单。我们的例子是一个简单的 REST API 应用程序。

## 用 decorate 简化 java 项目

de Korea 项目提供了一组 Java 注释和处理器，可以在应用程序编译期间自动更新 Kubernetes 清单。开发人员可以使用配置文件中的注释或属性来自定义清单，而不需要编辑单独的 XML、YAML 或 JSON 模板。

每个注释字段都映射到一个属性，所以用 Java 注释指定的任何东西也可以用属性来指定，比如:`dekorate.[simplified-annotation-name].[[kebab-cased-property-name](https://medium.com/better-programming/string-case-styles-camel-pascal-snake-and-kebab-case-981407998841)]`。

简化名称对应于注释的类名，设置为小写并去掉后缀(例如，`application`)。你可以在这里找到支持属性[的完整参考。](https://github.com/dekorateio/dekorate/blob/master/assets/config.md)

Dekorate 几乎可以与任何 Java 构建工具一起工作，包括 Maven、Gradle、Bazel 和 sbt。为了让生活更简单，Dekorate 还检测 Java 框架，如 [Spring Boot](/topics/spring-boot/) 、[夸尔库斯](/topics/quarkus/)和索恩泰尔，并相应地调整生成的清单。参见文章 [*如何使用 de Korea 创建 Kubernetes 清单*了解更多细节](/blog/2019/08/15/how-to-use-dekorate-to-create-kubernetes-manifests/)。

**注意** : [如果你想玩 Dekorate，自己发现这个神奇工具的威力，试试这些例子](https://github.com/dekorateio/dekorate/blob/master/examples)。

## 一个简单的 Java REST 应用程序

Dekorate 被设计成可以与几种 Java 框架一起工作，但是它与普通的 Java 一样好。在接下来的小节中，我们将使用 Dekorate 为一个通用的 Java 应用程序生成 Kubernetes 和 OpenShift 清单。我们的例子是一个简单的 REST API。你可以在 GitHub 上找到完整的源代码。

### 部署对象

Dekorate 必须生成三个 Kubernetes 对象来部署应用程序。这些对象中的每一个——以及它们的 OpenShift 平台等价物——都扮演着不同的角色，如表 1 所示。

**Table 1: Objects used to deploy applications on Kubernetes and OpenShift.**

| 库伯内特资源 | OpenShift 资源 | 目的 |
| [部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) | [部署配置](https://docs.openshift.com/container-platform/4.6/applications/deployments/what-deployments-are.html#deployments-and-deploymentconfigs_what-deployments-are) | 作为一个单元部署和更新应用程序。 |
| [服务](https://kubernetes.io/docs/concepts/services-networking/service/) | [服务](https://docs.openshift.com/online/pro/architecture/core_concepts/pods_and_services.html#services) | 提供一个端点来访问作为服务的 pod。 |
| [入口](https://kubernetes.io/docs/concepts/services-networking/ingress/) | [路线](https://docs.openshift.com/online/pro/architecture/networking/routes.html) | 向集群外部的客户端公开服务。 |

接下来，我们将回顾使用 de Korea 创建清单的步骤，以便它们包含表 1 中列出的资源。

### 配置清单

启用 Dekorate 最简单的方法是使用 Maven 依赖项将相应的 JAR 文件添加到类路径中:

```
<dependency>
     <groupId>io.dekorate</groupId>
     <artifactId>kubernetes-annotations</artifactId>
     <version>0.13.6</version>
</dependency>
```

**注意**:为了让你的项目保持最新，确保你使用[最新版本的 de Korea](https://github.com/dekorateio/dekorate/releases)。

我们需要指定我们希望在 Maven 构建生命周期的编译阶段使用 de Korea。我们可以通过向主 Java 类添加`@Dekorate`注释来实现这一点。这里，我们将使用`@KubernetesApplication`注释，它提供了特定于 Kubernetes 的配置选项。编辑`App.java`类并添加注释:

```
@KubernetesApplication(
       . . .
       name = "hello-world-fwless-k8s",
       ...
)
```

这个配置在 Maven 编译目标期间创建了一个[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)。de Korea 使用`name`参数值来指定资源名称，并填充相应的 Kubernetes 标签(`app.kubernetes.io/name`)。在`target/classes/META-INF/dekorate/*.{json,yml}`文件夹中创建了`.yml`和`.json`清单文件。

正如我在介绍中提到的，Dekorate 与 Maven 和 Gradle 等构建自动化系统进行了轻量级集成。因此，它可以从工具配置中读取信息，而无需将构建工具本身放入类路径中。在我们的例子中，默认情况下，Dekorate 会使用名称 [maven artifactId](https://github.com/aureamunoz/hello-world-fwless/blob/dc58a3358f1e6618ff9b743803c521d8411bfacc/pom.xml#L5) ，但是我们用[注释名称参数](https://github.com/aureamunoz/hello-world-fwless/blob/dekorate-4-k8s/src/main/java/org/acme/App.java#L21)覆盖了它。Dekorate 将把这个名字作为 [Kubernetes 标签](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)添加到所有资源中，包括图像、容器、部署和服务。

### 注释端口参数

下一步是添加[注释端口参数](https://github.com/aureamunoz/hello-world-fwless/blob/cea00f8199f8c2bf04fb431b0766a701a5f6ce62/src/main/java/org/acme/App.java#L22)。这让我们可以配置`Service`端点，并指定它应该如何与 Java HTTP 端口进行映射:

```
@KubernetesApplication(
     ...
     ports = @Port(name = "web", containerPort = 8080),
     ...
)
```

配置注释端口参数会在[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)资源中添加容器端口。这也生成了一个[服务](https://kubernetes.io/docs/concepts/services-networking/service/)资源，允许访问 Kubernetes 集群内部的端点。同样，如果你使用像 Spring Boot 或 Quarkus 这样的框架，你可以绕过端口定义。de Korea 将使用现有的[框架元数据](https://github.com/dekorateio/dekorate#framework-integration)来生成资源。

### 入口资源

在集群内部有一个可用的服务或端点是非常好的。但是为了简化您的生活，我们现在将配置一个额外的`Ingress`资源，使您可以从笔记本电脑上看到该服务。这两种资源都在群集上配置代理应用程序(入口控制器),以重定向外部流量和内部应用程序。

将以下`expose`参数添加到 Dekorate 注释中:

```
@KubernetesApplication(
     ...
     expose = true,
     ...
)
```

**注意**:记住，如果您修改注释参数，您可以通过触发编译并检查清单文件中收集的值来验证该更改是否反映在清单中。

接下来，我们将添加一个`host`参数来指定本地 IP 地址或 DNS 可解析的主机名以访问服务:

```
@KubernetesApplication(
     ...
     host = "fw-app.127.0.0.1.nip.io",
     ...
)
```

`host`决定了应用程序将在哪个主机下公开。运行在 Kubernetes 集群上的`Ingress` [控制器](https://v1-16.docs.kubernetes.io/docs/concepts/architecture/controller/)使用`host`来添加一个新规则，以便将发往该主机的请求转发给相应的服务(例如，`fw-app`)。

### 部署控制器

最后，我们需要告诉`Deployment`控制器，当容器注册表中有新图像时，该如何操作。将`ImagePullPolicy`参数设置为`Always`可确保控制器在新的容器映像可用时立即重新部署新的应用程序:

```
@KubernetesApplication(
     ...
     imagePullPolicy = ImagePullPolicy.Always,
     ...
)
```

## 部署应用程序

总而言之，下面是使用我们讨论过的 Java 注释参数的 Dekorate 配置:

```
@KubernetesApplication(
     name = "hello-world-fwless-k8s",
     ports = @Port(name = "web", containerPort = 8080),
     expose = true,
     host = "fw-app.127.0.0.1.nip.io",
     imagePullPolicy = ImagePullPolicy.Always
)

```

生成的清单将作为`kubernetes.yml`和`kubernetes.json`文件出现在`target/classes/META-INF/dekorate`文件夹中。

或者，您可以使用特定于 OpenShift 的依赖项在 OpenShift 上部署应用程序。编辑`pom.xml`并添加以下内容:

```
<dependency>
    <groupId>io.dekorate</groupId>
    <artifactId>openshift-annotations</artifactId>
    <version>0.13.6</version>
</dependency>
```

现在，编辑`App.java`类并添加`@OpenShiftApplication`注释:

```
@OpenshiftApplication(
     name = "hello-world-fwless-openshift",
     ports = @Port(name = "web", containerPort = 8080),
     expose = true,
     imagePullPolicy = ImagePullPolicy.Always
)
```

这次，清单被命名为`openshift.yml`和`openshift.json`，它们将包含表 1 中列出的 OpenShift 资源: [DeploymentConfig](https://docs.openshift.com/container-platform/4.6/applications/deployments/what-deployments-are.html#deployments-and-deploymentconfigs_what-deployments-are) 、[服务](https://docs.openshift.com/online/pro/architecture/core_concepts/pods_and_services.html#services)和[路由](https://docs.openshift.com/online/pro/architecture/networking/routes.html)。

## 配置映像构建策略

在 Kubernetes 和 OpenShift 上运行应用程序要求我们将它们打包到一个容器映像中，pod 将在启动时引导该映像。将源代码打包到容器映像、容器注册表或映像名称中的过程就是我们所说的*映像构建策略*。Dekorate 目前支持几种模式，具体取决于你使用的是 Kubernetes 还是 OpenShift。

让我们看看如何用 Dekorate 配置映像构建策略。

### 对接器(kubernetes and openshift)

要使用 de Korea 配置 Docker 映像构建策略，以便在本地使用 Docker 客户端作为构建工具，请向`pom.xml`文件添加以下依赖项:

```
<dependency>
  <groupId>io.dekorate</groupId>
  <artifactId>docker-annotations</artifactId>
</dependency>
```

如果没有指定注册表，默认情况下将使用 Docker.io。使用 Maven 或 Gradle 重新编译项目。生成的清单文件将包含容器图像，作为`Deployment`或`DeploymentConfig`资源的一部分:

```
image: amunozhe/hello-world-fwless:1.0-SNAPSHOT
```

当您启动负责构建容器映像的工具时，将会用到这些参数。我们将在[后](#creating_and_sharing)对此进行更详细的研究。

如果您想使用另一个图像注册中心，编辑`App.java`类来添加`@DockerBuild`注释，以指定客户端将图像推送到的`registry`:

```
@DockerBuild(registry = "quay.io")
```

### 源到图像(S2I)(仅限 OpenShift)

[源到图像(S2I)](https://docs.openshift.com/container-platform/4.5/builds/understanding-image-builds.html#build-strategy-s2i_understanding-image-builds) 是一个在 OpenShift 上构建 Docker 格式图像的工具。使用 S2I，OpenShift 控制器执行映像构建，因此您不必在本地构建容器映像。

直到最近，Dekorate 还只支持 S2I 的映像构建；当使用`@OpenShiftApplication`注释时，S2I 资源`BuildConfig`是由 Dekorate 生成的。要避免生成`BuildConfig`资源，您可以使用`@S2iBuild(enabled=false)`参数。

## 生成清单

一旦我们配置了 Kubernetes 或 OpenShift 清单，我们就可以执行一个`mvn package`命令来生成实际的清单。如前所述，在`target/classes/META-INF/dekorate`目录中会创建两个文件:`kubernetes.json`和`kubernetes.yml`。

现在，我们准备根据指定的映像构建策略来构建容器映像。

## 创建和共享容器映像

现在，有许多技术可以在本地构建容器映像。这个例子介绍了其中的两个: [Docker](https://docs.docker.com/engine/reference/commandline/image_build/) 和 [Jib](https://github.com/GoogleContainerTools/jib) 。

### 码头工人

Dockerfile 是一个文本文件，用于使用 Docker 或 [Podman](https://podman.io/) 构建图像。它包含了容器化 Java 应用程序的说明。一个 [Dockerfile](https://github.com/aureamunoz/hello-world-fwless/blob/dekorate-4-k8s/Dockerfile) 随示例项目的源代码一起提供，可以在项目的根目录中找到。

**注意**:de Korea 不生成 Dockerfiles。它也不提供对执行映像构建和推送的内部支持。

如果需要本地安装 Docker 客户端，可以[在这里](https://docs.docker.com/get-docker/)下载。

运行以下命令来构建映像:

```
docker build -f Dockerfile -t amunozhe/hello-world-fwless:1.0-SNAPSHOT .
```

该图像仅在您的笔记本电脑上的 Docker 注册表中可用。我们需要将图像推送到一个外部图像注册中心，以允许 Kubernetes 或 OpenShift 运行它。对于本例，我们将使用我的 Docker Hub ID (amunozhe)将图像推送到 Docker Hub。你可以在 Docker Hub 网站上注册自己的 ID，或者使用另一个公共注册，如[红帽码头](https://quay.io/)。

登录到注册表后，您最终可以推送映像，使其可用于群集:

```
docker push amunozhe/hello-world-fwless:1.0-SNAPSHOT
```

#### 通过装饰建立和推广形象

您可以将任务委托给 Dekorate，而不是手动执行构建和推送 Java 容器映像的命令，de korate 使用[钩子](https://github.com/dekorateio/dekorate#docker-build-hook)来支持这些特性。这意味着您可以用一个命令执行所有必要的步骤:

```
mvn clean package -Ddekorate.build=true  -Ddekorate.push=true
```

### 移转

Jib 通过让您避免编写 Dockerfile 文件，简化了创建容器图像的过程。你甚至不需要安装 Docker 客户机来用 Jib 创建和发布容器图像。使用 Jib(通过 Maven 插件)也很好，因为它可以捕捉每次构建时对应用程序所做的任何更改。

要启用 Jib，用`pom.xml`文件中的`jib-annotations`替换`docker-annotations`依赖关系:

```
<dependency>
   <groupId>io.dekorate</groupId>
   <artifactId>jib-annotations</artifactId>
</dependency>
```

我们将使用`@JibBuild`注释而不是`@DockerBuild`来配置注册表。编辑`application.java`类来添加注释:

```
@JibBuild(registry = "docker.io")
```

运行以下命令，利用 Dekorate 挂钩构建并推送容器映像:

```
mvn clean package -Ddekorate.push=true
```

## 部署应用程序

一旦生成了 Kubernetes 和 OpenShift 清单，并将映像推送到映像注册中心，我们就可以在集群上的 demo [名称空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)下部署应用程序。

要在云平台上创建您的微服务，您必须能够访问活动的 Kubernetes 或 OpenShift 集群。下面是为 Kubernetes 执行的命令:

```
kubectl create ns demo
kubectl apply -f target/classes/META-INF/dekorate/kubernetes.yml -n demo
```

这是 OpenShift 的一个例子:

```
oc new-project demo
oc apply -f target/classes/META-INF/dekorate/openshift.yml

```

稍等片刻，直到创建了 pod，并且可以通过注册为`Ingress`或`Route`资源的外部 URL 进行访问。您可以通过查看 pod 状态是否为`RUNNING`来验证应用程序是否准备好接受请求。

Kubernetes 用户应该使用这个命令:

```
kubectl get pods -n demo
```

对于 OpenShift 用户，它是:

```
oc project demo
oc get pods
```

接下来，您需要获得在浏览器中访问应用程序的 URL。对 Kubernetes 使用以下命令:

```
kubectl get ingress -n demo
NAME                     CLASS   HOSTS                      ADDRESS     PORTS   AGE

hello-world-fwless-k8s   <none>  fw-app.127.0.0.1.nip.io    localhost   80      147m
```

正如您在[主应用程序类](https://github.com/aureamunoz/hello-world-fwless/blob/master/src/main/java/org/acme/App.java#L14)中看到的，`/api/hello`路径提供了端点。要检查应用程序是否可访问，请打开浏览器并转至`[http://fw-app.127.0.0.1.nip.io/api/hello](http://fw-app.127.0.0.1.nip.io/api/hello)`。您应该会看到以下消息:

```
Hello From k8s FrameworkLess world!
```

以下是如何使用 OpenShift 检查应用程序:

```
oc get route -n demo

NAME                          HOST/PORT                                              PATH       PORT

hello-world-fwless-openshift  hello-world-fwless-openshift-demo.88.99.12.170.nip.io   /         8080
```

在浏览器中，转到`[http://hello-world-fwless-openshift-demo.88.99.12.170.nip.io/api/hello](http://hello-world-fwless-openshift-demo.88.99.12.170.nip.io/api/hello)`。您应该会看到以下消息:

```
Hello from OpenShift FrameworkLess world!
```

## 结论

作为一名开发人员，我喜欢创建新的特性，改善用户体验，并发现快速、轻松地部署应用程序的方法。正如您在本文中所看到的，Dekorate 使得编写 Kubernetes 或 OpenShift 清单几乎毫不费力。

*Last updated: March 16, 2021*