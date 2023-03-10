# quar kus:Kubernetes-native Java 栈快速入门指南

> 原文：<https://developers.redhat.com/articles/quarkus-quick-start-guide-kubernetes-native-java-stack>

四五年前，当不可变基础设施开始大步前进时，Java 开发人员一直在努力解决的一个问题是:“我的[微服务](https://developers.redhat.com/topics/microservices/)必须如何在 Linux 容器架构中进行优化，比如更快的启动和更小的内存占用？”

当时，在转变容器原生的、基于微服务的 [Java](https://developers.redhat.com/topics/enterprise-java/) 技术时有许多限制。一个主要原因是传统的云 Java 堆栈必须处理大多数所需的任务，如注释扫描，在运行时而不是构建时解析描述符，直到 [Quarkus](https://developers.redhat.com/topics/quarkus/) 出现。

[Quarkus](https://developers.redhat.com/topics/quarkus/) 是一个[Kubernetes](https://developers.redhat.com/topics/kubernetes/)——为 GraalVM 和 OpenJDK HotSpot 量身定制的原生 Java 栈。它由一流的 Java 库和标准精心打造，具有以下优势:

*   优化开发人员乐趣的内聚平台
    *   现场编码和统一配置
    *   轻松生成本地可执行文件
*   容器优先和 Kubernetes 本地 Java 堆栈
    *   超快启动(即 0.055 秒静止+积垢)
    *   内存占用小(即 35 MB 用于 REST + CRUD)
*   在同一个应用程序中统一命令式和反应式开发
    *   注入事件总线或 Vertx 上下文
    *   使用适合您的使用案例的技术
    *   基于事件驱动应用的反应式系统的关键
*   最新流行的开源项目的扩展

![Open source logos](img/0156457935b3ebc609fae2df165f6a71.png "Figure 1: Open source projects")

## 开始编码

让我们开始开发一个简单的微服务应用程序，从头开始公开 REST 端点。

### 引导项目

进入 [Quarkus.io](https://quarkus.io/) 中的[开始编码](https://code.quarkus.io/)页面，使用 RESTEasy JAX-RS 和 REST 客户端扩展引导第一个 Quarkus 应用程序:

![Screenshot of steps](img/f69c0c564046d1835400afa847a0aa42.png "quarkus-quickstart-f2")

它生成一个 ZIP 文件，包含:

*   Maven 结构
*   org.acme.ExampleResource 资源在 */hello* 上公开
*   关联的单元测试
*   启动应用程序后，可在 *http://localhost:8080* 上访问的登录页面
*   本地和 jvm 模式的 docker 文件示例
*   应用程序配置文件

通过您喜欢的 IDE(即 VSCode)解压缩并检查框架代码，如下所示:

```
unzip code-with-quarkus.zip && cd code-with-quarkus
code .
```

一旦你打开了一个 IDE，查看一下 *pom.xml* 。您将发现 Quarkus BOM 的导入，允许您省略不同 Quarkus 依赖项上的版本。此外，您可以看到负责应用程序打包和提供开发模式的 *quarkus-maven-plugin* 。

![You can see the quarkus-maven-plugin responsible for the packaging of the application and for providing the development mode.](img/f7bf487d8a31241f9c2c6bfaf773d41d.png "You can see the quarkus-maven-plugin responsible for the packaging of the application and for providing the development mode.")

### 使用开发模式运行应用程序

要运行该应用程序，您需要:

*   安装了 JDK 1.8+并适当配置了 JAVA_HOME
*   Apache 胃 3.5.3+

现在您已经准备好运行您的应用程序了。用途:`mvn compile quarkus:dev`:

```
INFO] Scanning for projects...
INFO] ---------------------< org.acme:code-with-quarkus >---------------------
[INFO] Building code-with-quarkus 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ code-with-quarkus ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 2 resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ code-with-quarkus ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- quarkus-maven-plugin:1.0.0.CR1:dev (default-cli) @ code-with-quarkus ---
Listening for transport dt_socket at address: 5005
2019-11-09 08:03:20,900 INFO  [io.qua.dep.QuarkusAugmentor] (main) Beginning quarkus augmentation
2019-11-09 08:03:21,685 INFO  [io.qua.dep.QuarkusAugmentor] (main) Quarkus augmentation completed in 785ms
2019-11-09 08:03:22,125 INFO  [io.quarkus] (main) Quarkus 1.0.0.CR1 started in 1.364s. Listening on: http://0.0.0.0:8080
2019-11-09 08:03:22,126 INFO  [io.quarkus] (main) Profile dev activated. Live Coding activated.
2019-11-09 08:03:22,126 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]

```

一旦应用程序启动，打开 web 浏览器请求提供的端点:*http://localhost:8080/hello*

![Once started, open a web browser to request the provided endpoint:  http://localhost:8080/hello](img/0501f04f41e406c0e267bd8ecbc56ae8.png "Once started, open a web browser to request the provided endpoint:  http://localhost:8080/hello")

### 更改代码

将 *hello()* 方法中的返回字符串修改为*“Welcome，quar kus Cheat Sheet”*。记得保存文件:

```
public String hello() {
    return "Welcome, Quarkus Cheat Sheet";
}
```

返回网络浏览器，然后重新加载页面。应用程序将被神奇地重新部署，而无需重启运行时:

![The application will be redeployed magically without restarting the runtime.](img/f0e1d946c9a3dc49eedf655dc1e57082.png "The application will be redeployed magically without restarting the runtime.")

点击 *CTRL+C* 来停止应用程序，但是你也可以让它继续运行，享受超快的热重装。

#### 打包并运行应用程序

应用程序是使用`mvn clean package -DskipTests`打包的。它生成两个 jar 文件:

*   code-with-quar kus-1 . 0 . 0-snapshot . jar:仅包含项目的类和资源，是 Maven 构建产生的常规工件。
*   **code-with-quar kus-1 . 0 . 0-SNAPSHOT-runner . jar**:可执行的 jar。请注意，它不是一个超级 jar，因为依赖项被复制到了*目标/库*目录中。

如果端点在开发模式下工作，则运行应用程序:

```
$ java -jar target/code-with-quarkus-1.0.0-SNAPSHOT-runner.jar
2019-11-09 08:01:14,240 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT (running on Quarkus 1.0.0.CR1) started in 0.660s. Listening on: http://0.0.0.0:8080
2019-11-09 08:01:14,289 INFO  [io.quarkus] (main) Profile prod activated. 
2019-11-09 08:01:14,289 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]

// Run the curl command in a new terminal 
$ curl http://localhost:8080/hello
Welcome, Quarkus Cheat Sheet
```

## 构建本机可执行文件

现在让我们为我们的应用程序生成一个本机可执行文件。它缩短了应用程序的启动时间，并产生了最小的磁盘占用空间。可执行文件将包含运行应用程序的所有内容，包括“JVM”(缩小到刚好足以运行应用程序)和应用程序。

![The executable would have everything to run the application including the "JVM" (shrunk to be just enough to run the application) and the application.](img/a2c6983d023d364faaa4b042cb720219.png "The executable would have everything to run the application including the "JVM" (shrunk to be just enough to run the application) and the application.")

### 安装 GraalVM 并配置环境

要构建本机可执行映像，您需要:

*   在 [GraalVM 网站](https://www.graalvm.org/downloads/)中安装 GraalVM community edition 至少 19.1.1 版本。
*   从您的 GraalVM 目录运行`gu install native-image`
*   GRAALVM_HOME 环境变量配置如下:

Linux:

```
export GRAALVM_HOME=$HOME/Development/graalvm/
```

macOS:

```
export GRAALVM_HOME=$HOME/Development/graalvm/Contents/Home/
```

在继续之前，请确保正确配置了 *GRAALVM_HOME* 环境变量。

您将使用“本地”概要文件在 pom.xml 中构建一个本地可执行映像:

![You will use “native” profile to build a native executable image in the pom.xml.](img/df65bbfb29b1f5a5a928f1f30d2ede47.png "You will use “native” profile to build a native executable image in the pom.xml.")

### 创建本地可执行文件并运行它

运行`mvn clean package -DskipTests -Pnative`。生成图像需要几分钟时间:

```

...
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]     (clinit):     438.26 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]     universe:   1,174.98 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]      (parse):   2,329.86 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]     (inline):   3,247.13 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]    (compile):  27,917.55 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]      compile:  35,768.02 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]        image:   2,067.69 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]        write:     885.91 ms
[code-with-quarkus-1.0.0-SNAPSHOT-runner:86834]      [total]:  70,647.15 ms
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 73212ms
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------
[INFO] Total time:  01:17 min
[INFO] Finished at: 2019-11-09T08:05:40+09:00
[INFO] ------------------------------------------------------------------

```

让我们运行本机可执行文件:

```

./target/code-with-quarkus-1.0.0-SNAPSHOT-runner
2019-11-09 08:07:24,786 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT (running on Quarkus 1.0.0.CR1) started in 0.013s. Listening on: http://0.0.0.0:8080
2019-11-09 08:07:24,787 INFO  [io.quarkus] (main) Profile prod activated. 
2019-11-09 08:07:24,787 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]

```

运行我们的微服务只需 13 毫秒。当您返回 web 浏览器并重新加载端点页面时，您会发现完全相同的结果:

![You will find the same result exactly when you go back to the web browser then reload the endpoint page.](img/ebb47316fdab0fd17871055672a2400e.png "You will find the same result exactly when you go back to the web browser then reload the endpoint page.")

在进入下一步之前，点击 *CTRL+C* 停止应用程序。

## 在 Kubernetes 部署

现在，您将在不变的基础设施 Kubernetes 集群上部署优化的微服务应用程序。你可以在这里安装自己的 Kubernetes 集群。

### 容器化应用程序

项目生成在 src/main/docker 目录中提供了一个 Dockerfile.native，内容如下:

```
FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

然后，需要将 Docker 映像推送到 Kubernetes 集群的映像注册中心。根据您的集群，有几种方法。对于 Minikube，执行:

```
eval $(minikube docker-env)

docker build -f src/main/docker/Dockerfile.native -t 
quarkus-cheatsheet/myapp .
```

### 在 Kubernetes 中部署应用程序

将图像推送到 Kubernetes 图像注册中心后，按如下方式实例化应用程序:

```
kubectl run quarkus-cheatsheet --image=quarkus-cheatsheet/myapp:latest --port=8080 --image-pull-policy=IfNotPresent

kubectl expose deployment quarkus-cheatsheet --type=NodePort
```

该应用程序现在作为内部服务公开。如果您使用的是 Minikube，您可以通过以下方式访问它:

```
curl $(minikube service quarkus-quickstart --url)/hello

Welcome, Quarkus Cheat Sheet
```

### 更多 quartus 指南

*   [开始 Quarkus](https://developers.redhat.com/courses/quarkus/getting-started/) 课程(预计时间:10 分钟)
*   [红帽开发者夸库课程](https://developers.redhat.com/courses/quarkus/)
*   [开始使用 Eclipse Che 7 和 Quarkus:概述](https://developers.redhat.com/blog/2019/08/19/get-started-with-eclipse-che-7-and-quarkus-an-overview/)
*   Quarkus:超音速亚原子 Java
*   [从零到夸夸其谈:最简单的方法](https://developers.redhat.com/blog/2019/04/09/from-zero-to-quarkus-and-knative-the-easy-way/)
*   [使用 Eclipse IDE(Red Hat code ready Studio)创建您的第一个 Quarkus 项目](https://developers.redhat.com/blog/2019/05/09/create-your-first-quarkus-project-with-eclipse-ide-red-hat-codeready-studio/)

[Quarkus 网站](https://quarkus.io/guides/)提供了关于如何使用 quar kus 扩展和常见用例开发微服务/云原生应用/无服务器的额外实用指南:

*   上下文和依赖注入
*   对本机映像使用 SSL
*   利用 JWT·RBAC
*   简洁华丽的 Hibernate ORM
*   使用阿帕奇卡夫卡流
*   使用密钥锁保护 JAX 遥感应用程序
*   在 Kubernetes 或 OpenShift 上部署本地应用
*   使用 OpenTracing 和收集指标
*   用 Maven 构建应用程序
*   为 Spring DI API 使用 Quarkus 扩展

*Last updated: January 13, 2022*