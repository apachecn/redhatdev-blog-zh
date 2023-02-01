# 使用 Red Hat OpenShift 应用程序运行时的源代码与二进制 S2I 工作流

> 原文：<https://developers.redhat.com/blog/2018/09/26/source-versus-binary-s2i-workflows-with-red-hat-openshift-application-runtimes>

Red Hat OpenShift 支持两种为应用程序构建容器映像的工作流:源*和 T2 二进制*工作流。二进制工作流是 [Red Hat OpenShift 应用程序运行时](https://developers.redhat.com/products/rhoar/overview/)和 [Red Hat Fuse](https://developers.redhat.com/products/fuse/overview/) 产品文档和培训的主要焦点，而源代码工作流是大多数 [Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview/)产品文档和培训的焦点。所有标准的 OpenShift 快速应用程序模板都基于源工作流。

开发人员可能会问，“我可以在同一个项目中使用两种工作流吗？”或者，“有理由选择一个工作流而不是另一个吗？”作为为 OpenShift 和 Red Hat Fuse 开发 Red Hat 认证培训的团队成员，我自己也有这些问题，我希望这篇文章能帮助您找到自己对这些问题的答案。

## 比较二进制和源工作流

因为两个工作流都基于*源到图像(S2I)* 功能，所以有一个*二进制*选项听起来可能有点奇怪。实际上，两个工作流都依赖于使用*源*策略的 S2I 构建。关键的区别在于，源工作流在 OpenShift 内部生成应用程序的可部署工件，而二进制工作流在 OpenShift 外部生成这些二进制工件。他们都在 OpenShift 中构建应用程序容器映像。

在某种意义上，二进制工作流提供了一个应用程序二进制文件，作为生成容器映像的 OpenShift S2I 构建的源。

举一个简单的应用程序，比如在 https://github.com/flozanorht/vertx-hello.git 的 GitHub 上基于 Vert.x 的“Hello，World”。它可以作为一个独立的 Java 应用程序在本地运行，并且可以使用来自相同源的两个工作流部署在 OpenShift 上。

### 使用二元工作流

使用二进制工作流时，开发人员将:

1.  将项目克隆到本地文件夹，这样他们就可以根据自己的喜好修改代码，也可以在 OpenShift 之外进行测试。
2.  登录 OpenShift，新建一个项目。
3.  使用`mvn`命令，该命令又使用 *Fabric8 Maven 插件(FMP)* 来构建容器映像并创建描述应用程序的 OpenShift 资源。Maven 执行以下任务:

    1.  生成应用程序包(可执行的 JAR)
    2.  启动创建应用程序容器映像的二进制构建，并将应用程序包传输到构建窗格
    3.  创建 OpenShift 资源来部署应用程序
4.  使用 curl 或 web 浏览器来测试应用程序。

### 使用源工作流

使用源工作流时，开发人员将:

1.  将项目克隆到本地文件夹，这样他们就可以根据自己的喜好修改代码，也可以在 OpenShift 之外进行测试。
2.  提交并将任何更改推送到原始 git 存储库。
3.  登录 OpenShift，新建一个项目。
4.  使用`oc new-app`命令构建容器映像并创建描述应用程序的 OpenShift 资源。

    1.  OpenShift 客户端命令创建 OpenShift 资源来构建和部署应用程序。
    2.  构建配置资源启动一个运行 Maven 的源构建，以生成应用程序包(JAR)并创建包含应用程序包的应用程序容器映像。
5.  向外界公开应用程序服务。
6.  使用 curl 或 web 浏览器来测试应用程序。

从操作角度来看，两个工作流程的主要区别在于使用`mvn`命令还是`oc new-app`命令:

*   二进制工作流依靠 Maven 来完成繁重的工作，这是大多数 Java 开发人员所欣赏的。
*   另一方面，源工作流依赖于 OpenShift 客户端(`oc`命令)。

### 何时使用每个工作流程

在宣布您更喜欢二进制工作流(因为您已经了解 Maven)之前，请考虑开发人员使用 OpenShift 客户端来监控和诊断他们的应用程序，因此使用 Maven 执行一些任务可能并不是一个很大的胜利。

源工作流要求将所有更改都推送到网络可访问的 git 存储库，而二进制工作流处理本地更改。这种差异来自于这样一个事实，即源工作流在 OpenShift 内部构建您的 Java 包(JAR ),而二进制工作流在不接触您的源代码的情况下在本地获取您构建的 Java 包。

对于一些开发人员来说，使用二进制工作流可以节省时间，因为他们无论如何都会执行本地构建来运行单元测试和执行其他任务。对于其他开发者来说，源码工作流带来的好处是所有繁重的工作都由 OpenShift 完成。这意味着开发人员的工作站不需要安装 Maven、Java 编译器和其他任何东西。相反，开发人员可以使用低性能 PC，甚至平板电脑来编写代码，提交和启动 OpenShift 构建来测试他们的应用程序。

没有什么可以阻止您使用这两种工作流来满足不同的目标。例如，开发人员可以使用二进制工作流在本地 minishift 实例中测试他们的更改，包括运行单元测试，而 QA 环境使用源工作流来构建应用程序容器映像，由 webhook 触发，并在专用的多节点 OpenShift 集群中执行一组集成测试。

## 使用二进制和源工作流部署应用程序

让我们通过基于 [Vert.x 的“Hello，World”应用程序](https://github.com/flozanorht/vertx-hello.git)来试验这两种工作流。如果您不熟悉 Vert.x，也不用担心，因为该应用程序非常简单，随时可以运行。重点关注 Maven POM 设置，它允许同一个项目同时处理二进制和源工作流。

与您可能在其他地方看到的大多数示例不同，我的 Vert.x 应用程序被配置为使用来自 Red Hat OpenShift 应用程序运行时的受支持的依赖项，而不是上游社区构件。需要成为红帽付费客户才能按照说明操作吗？一点也不:你只需要在[红帽开发者网站](http://developers.redhat.com/)注册，就可以获得红帽企业 Linux、红帽 OpenShift 容器平台、红帽 OpenShift 应用运行时以及红帽所有中间件和 DevOps 组合的免费开发者订阅。

启动您的 minishift 实例，让我们在 OpenShift 上尝试源代码和二进制编译。如果您没有 minishift 实例，请向 Red Hat Developers 注册，[下载并安装 Red Hat Container Development Kit(CDK)](https://developers.redhat.com/products/cdk/download/)，并按照[的说明来设置 minishift](https://developers.redhat.com/products/cdk/hello-world/) ，它提供了一个 VM 来在您的本地机器上运行 OpenShift。或者，如果您可以访问一个真正的 OpenShift 集群，请登录到它并按照下一节中的说明进行操作。

## 使用二进制工作流的 vert . x“Hello，World”

首先，克隆示例 Vert.x 应用程序。以下命令假设您使用的是 Linux 机器，但是将它们适用于 Windows 或 Mac 机器应该不难。你可以在其中任何一个运行 CDK。

```
$ git clone https://github.com/flozanorht/vertx-hello.git
$ cd vertx-hello
```

因为我们使用受支持的 Maven 工件，所以您需要配置您的 Maven 安装以使用 Red Hat Maven 存储库。`conf`文件夹包含一个样本 Maven `settings.xml`文件，你可以把它复制到你的`~/.m2`文件夹中，或者作为修改的例子。

登录 OpenShift 并创建一个测试项目。下面的说明假设您正在使用 minishift，并且您的 minishift 实例已经在运行，但是应该不难使它们适应外部 OpenShift 集群。

```
$ oc login -u developer -p developer
$ oc new-project binary
```

现在有趣的部分来了:让 Fabric8 Maven 插件(FMP)做所有的工作。

```
$ mvn -Popenshift fabric8:deploy
…
[INFO] Building Vert.x Hello, World 1.0
…
[INFO] --- fabric8-maven-plugin:3.5.38:resource (fmp) @ vertx-hello ---
[INFO] F8: Running in OpenShift mode
[INFO] F8: Using docker image name of namespace: binary
[INFO] F8: Running generator vertx
[INFO] F8: vertx: Using ImageStreamTag 'redhat-openjdk18-openshift:1.3' as builder image
…
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ vertx-hello ---
…
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
…
[INFO] --- fabric8-maven-plugin:3.5.38:build (fmp) @ vertx-hello ---
[INFO] F8: Using OpenShift build with strategy S2I
…
[INFO] F8: Starting S2I Java Build .....
[INFO] F8: S2I binary build from fabric8-maven-plugin detected
[INFO] F8: Copying binaries from /tmp/src/maven to /deployments ...
[INFO] F8: ... done
[INFO] F8: Pushing image 172.30.1.1:5000/binary/vertx-hello:1.0 …
…
[INFO] F8: Pushed 6/6 layers, 100% complete
[INFO] F8: Push successful
[INFO] F8: Build vertx-hello-s2i-1 Complete
…
[INFO] --- fabric8-maven-plugin:3.5.38:deploy (default-cli) @ vertx-hello ---
…
[INFO] BUILD SUCCESS
…
```

FMP 的配置在名为`openshift`的 Maven 概要文件中。这个概要文件允许您在本地构建和运行应用程序，而不需要 OpenShift。如果您愿意，调用 Maven `package`目标并使用`java -jar`从目标文件夹运行 JAR 包。

构建需要一些时间；大部分是用来下载 Maven 神器的。生成容器映像并将其推送到内部注册表需要几秒到几分钟的时间。

FMP 可能会失去与构建器 pod 的连接，并显示如下警告消息，您可以忽略这些消息:

```
[INFO] Current reconnect backoff is 1000 milliseconds (T0)
```

也可以忽略以下错误消息:

```
[ERROR] Exception in reconnect
java.util.concurrent.RejectedExecutionException: Task java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask@741d5132 rejected from
…
```

FMP 已经修复了这些问题，这些修复将在不久的将来出现在 Red Hat OpenShift 应用程序运行时中。

尽管有这些消息，您的构建还是成功的。FMP 还创建了允许访问您的应用程序的路由。查找分配给路由的主机名:

```
$ oc get route
NAME          HOST/PORT                                  PATH     SERVICES      PORT      TERMINATION   WILDCARD
vertx-hello   vertx-hello-binary.192.168.42.180.nip.io            vertx-hello   8080          None
```

然后使用 curl 测试您的应用程序:

```
$ curl http://vertx-hello-binary.192.168.42.180.nip.io/api/hello/Binary
Hello Binary, from vertx-hello-binary.192.168.42.180.nip.io.
```

### 用二进制工作流打开 Shift 资源

请注意，FMP 还创建了一些 OpenShift 资源:服务、部署配置和应用程序单元:

```
$ oc status
In project binary on server https://192.168.42.180:8443

http://vertx-hello-binary.192.168.42.180.nip.io to pod port 8080 (svc/vertx-hello)
 dc/vertx-hello deploys istag/vertx-hello:1.0 <-
    bc/vertx-hello-s2i source builds uploaded code on openshift/redhat-openjdk18-openshift:1.3
    deployment #1 deployed 8 minutes ago - 1 pod
…
```

FMP 创建 OpenShift 资源内部默认值。您可以提供存储在`src/main/fabric8 project`文件夹中的资源片段文件来覆盖这些缺省值。如果您想要为您的 pod 自定义就绪性探测或资源限制，您必须编辑这些文件。

您可以查看 FMP 为您创建的 OpenShift 构建配置。注意策略是`Source`，有二进制输入。

```
$ oc get bc
NAME              TYPE      FROM      LATEST
vertx-hello-s2i   Source    Binary    1
$ oc describe bc vertx-hello-s2i
…
Strategy:    Source
From Image:    ImageStreamTag openshift/redhat-openjdk18-openshift:1.3
Output to:    ImageStreamTag vertx-hello:1.0
Binary:       provided on build
…
```

### 使用二进制工作流重建

构建需要二进制输入的事实意味着您不能简单地使用`oc start-build`命令开始一个新的构建。您也不能使用 OpenShift webhooks 来开始新的构建。如果您需要执行应用程序的新构建，请再次使用`mvn`命令:

```
$ mvn -Popenshift fabric8:deploy
…
[INFO] Building Vert.x Hello, World 1.0
…
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
…
[INFO] --- fabric8-maven-plugin:3.5.38:build (fmp) @ vertx-hello ---
[INFO] F8: Using OpenShift build with strategy S2I
…
[INFO] F8: Pushed 6/6 layers, 100% complete
[INFO] F8: Push successful
[INFO] F8: Build vertx-hello-s2i-2 Complete
…
[INFO] BUILD SUCCESS
…
```

最后，您会得到一个新的容器映像和一个新的应用程序 pod。尽管构建日志似乎暗示 FMP 重新创建(或更新)了 OpenShift 资源，但实际上并没有改变它们。如果您需要 FMP 来更新 OpenShift 资源，您需要调用`fabric8:undeploy` Maven 目标，然后再次调用`fabric8:deploy`。

# 使用源工作流的 vert . x“Hello，World”

首先，克隆示例 Vert.x 应用程序。以下命令假设您使用的是 Linux 机器，但是将它们适用于 Windows 或 Mac 机器应该不难。你可以在其中任何一个运行 CDK。如果您已经从前面的指令中克隆了，那么您可以重用同一个克隆的 git 存储库并跳过后面的命令。

```
$ git clone https://github.com/flozanorht/vertx-hello.git
$ cd vertx-hello
```

不需要配置 Maven 来使用 Red Hat Maven 存储库。OpenShift builder 映像已经预先配置好了。您将不再执行本地 Maven 构建。

登录 OpenShift 并创建一个测试项目。下面的说明假设您使用的是 minishift，但是将它们应用到外部 OpenShift 集群应该不难。

```
$ oc login -u developer -p developer
$ oc new-project source
```

现在有趣的部分来了:让 OpenShift 的 S2I 特性做所有的工作。

```
$ oc new-app redhat-openjdk18-openshift:1.3~https://github.com/flozanorht/vertx-hello.git
…
--> Creating resources ...
    imagestream "vertx-hello" created
    buildconfig "vertx-hello" created
    deploymentconfig "vertx-hello" created
    service "vertx-hello" created
--> Success
    Build scheduled, use 'oc logs -f bc/vertx-hello' to track its progress.
…
```

按照`oc new-app`命令的建议，遵循 OpenShift 构建日志，其中包括 Maven 构建日志:

```
$ oc logs -f bc/vertx-hello -f
Cloning "https://github.com/flozanorht/vertx-hello.git" ...
…
Starting S2I Java Build .....
Maven build detected
Initialising default settings /tmp/artifacts/configuration/settings.xml
Setting MAVEN_OPTS to -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseParallelOldGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:MaxMetaspaceSize=100m -XX:+ExitOnOutOfMemoryError
Found pom.xml ...
…
[INFO] Building Vert.x Hello, World 1.0
…
[INFO] BUILD SUCCESS
…
Copying Maven artifacts from /tmp/src/target to /deployments ...
…
Pushed 6/6 layers, 100% complete
Push successful
```

注意，源构建调用 Maven 时使用了阻止它运行 FMP 的参数。另外，请注意构建的最终结果是一个容器映像。构建器将容器映像推入内部注册表。

构建需要一些时间；大部分是用来下载 Maven 神器的。生成和推送容器映像需要几秒到几分钟的时间。

构建完成后，您需要创建一个路由来允许访问您的应用程序。

```
$ oc expose svc vertx-hello
route "vertx-hello" exposed
```

现在查找分配给路由的主机名:

```
$ oc get route
NAME          HOST/PORT                                  PATH      SERVICES      PORT       TERMINATION   WILDCARD
vertx-hello   vertx-hello-source.192.168.42.180.nip.io             vertx-hello   8080-tcp           None
```

然后使用`curl`测试您的应用程序:

```
$ curl http://vertx-hello-source.192.168.42.180.nip.io/api/hello/Source
Hello Source, from vertx-hello-source.192.168.42.180.nip.io.
```

### 使用源工作流打开班次资源

注意，`oc new-app`命令还创建了一些 OpenShift 资源:一个服务、一个部署配置和应用程序单元:

```
$ oc status
In project source on server https://192.168.42.180:8443

http://vertx-hello-source.192.168.42.180.nip.io to pod port 8080-tcp (svc/vertx-hello)
 dc/vertx-hello deploys istag/vertx-hello:latest <-
    bc/vertx-hello source builds https://github.com/flozanorht/vertx-hello.git on openshift/redhat-openjdk18-openshift:1.3
    deployment #1 deployed 3 minutes ago - 1 pod
…
```

`oc new-app`命令使用硬编码的默认值创建 OpenShift 资源。如果您想要自定义它们，例如，为您的 pod 指定就绪探测器或资源限制，您有两种选择:

*   找到(或创建)一个合适的 OpenShift 模板，并使用该模板作为`oc new-app`命令的输入。名称空间中的模板是一个很好的起点。
*   使用 OpenShift 客户端命令，如`oc set`和`oc edit`来改变资源的位置。

您可以查看`oc new-app`命令为您创建的 OpenShift 构建配置的内部。注意策略是`Source`，有一个 URL 输入。

```
$ oc get bc
NAME          TYPE      FROM      LATEST
vertx-hello   Source    Git       1
$ oc describe bc vertx-hello
…
Strategy:    Source
URL:       https://github.com/flozanorht/vertx-hello.git
From Image:    ImageStreamTag openshift/redhat-openjdk18-openshift:1.3
Output to:    ImageStreamTag vertx-hello:latest
…
```

### 使用源工作流重建

如果您需要执行应用程序的新构建，请使用`oc start-build`命令:

```
$ oc start-build vertx-hello
build "vertx-hello-2" started
```

您可以使用相同的`oc logs`命令来查看 OpenShift 构建日志。最后，您有了一个新的容器映像和一个准备好并正在运行的新应用程序 pod。您不需要重新创建路线和其他 OpenShift 资源。

请注意，Maven 会再次下载所有依赖项，因为 build pod 只使用容器临时存储。默认情况下，在 S2I 构建之间不会重用 Maven 缓存。执行本地 Maven 构建的开发人员依靠他们的本地 Maven 缓存来加速重建。OpenShift 提供了一个增量构建特性，允许在 S2I 构建之间重用 Maven 缓存。增量构建是未来文章的主题。

如果您计划使用源代码构建，我建议您配置一个 Maven 存储库服务器，比如 Nexus。`MAVEN_MIRROR_URL`构建环境变量指向 Maven 存储库服务器。这个建议适用于任何开发 Java 应用程序的组织，但是 OpenShift 使得这种需求更加迫切。

# 结论

理解 OpenShift 二进制和源代码工作流可以让开发人员就何时使用哪一个做出明智的决定。由 Jenkins 或任何其他工具管理的持续集成/持续交付(CI/CD)管道可以使用这两者。

二进制和源工作流使用相同的 OpenShift S2I 构建器图像。两者都在 OpenShift 内部构建容器映像，并将它们推送到内部注册表。出于大多数实际目的，由二进制或源工作流生成的容器映像是等效的。

二进制工作流可能更适合当前的开发人员工作流，尤其是当项目的 POM 需要大量定制时。源工作流允许基于云的开发，如 [Red Hat OpenShift.io](https://developers.redhat.com/products/openshiftio/overview/) ，并使开发人员不再需要高端工作站。

Red Hat Training 提供了两个面向开发人员的课程，内容涉及 OpenShift 构建:

*   Red Hat OpenShift 开发 I:容器化应用( [DO288](https://www.redhat.com/en/services/training/do288-red-hat-openshift-development-i-containerizing-applications) )
*   Red Hat OpenShift 开发 II:使用 Red Hat OpenShift 应用程序运行时创建微服务( [DO292](https://www.redhat.com/en/services/training/red-hat-openshift-development-ii) )

你还可以免费得到格雷厄姆·邓普顿的《部署到 OpenShift 的书[。](https://www.openshift.com/deploying-to-openshift/)