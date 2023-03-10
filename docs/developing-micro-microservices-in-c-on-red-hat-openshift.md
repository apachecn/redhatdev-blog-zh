# 在红帽 OpenShift 上用 C 开发微微服务

> 原文：<https://developers.redhat.com/blog/2020/08/27/developing-micro-microservices-in-c-on-red-hat-openshift>

Java 在企业中间件中占据主导地位是有原因的；然而，用“微”来描述 Java 中的任何东西都需要一个宽松的解释。基于 Java 的[微服务](https://developers.redhat.com/topics/microservices)需要半个千兆字节的内存来在适度的负载下提供适度的功能，这并不罕见。向[无服务器架构](https://developers.redhat.com/topics/serverless-architecture)发展的趋势，即根据需求启动和停止服务，并没有改善这种情况。

最近，使用像 [GraalVM](https://developers.redhat.com/blog/2020/06/05/mandrel-a-community-distribution-of-graalvm-for-the-red-hat-build-of-quarkus/) 这样的工具将 Java 编译成本地可执行文件已经成为可能。这种技术，加上像 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 这样的优化 Java 运行时，在某种程度上驯服了 Java 的资源消耗。

然而，我们不应该忽视那些从一开始就被设计成编译成本机代码的编程语言，这些语言几乎没有运行时开销。像 [Rust](https://developers.redhat.com/blog/category/rust/) 和 [Go](https://developers.redhat.com/blog/category/go/) 这样的语言已经变得流行起来，这是有道理的。然而，对于最佳的运行时资源使用和毫秒级启动时间，仍然很难超越 [C](https://developers.redhat.com/topics/c/) 。

相对而言，IT 行业中很少有人有用 C 实现中间件组件的经验。这一事实具有讽刺意味，因为 C 是实现真正微服务的理想工具。使用[容器](https://developers.redhat.com/topics/containers/)消除了使用 C 的一个主要障碍——在二进制级别缺乏跨平台兼容性。类似于 Java 程序的 JVM，容器是本机代码的运行时环境。

本文讨论了用 c 实现基于 REST 的 web 服务的一些含义。我的例子是一个名为`solunar_ws`的组件，它计算太阳和月亮升起的时间，并设置任意一天任意位置的时间。我特意选择了一个独立的例子，但是它做了真正的计算工作。这个例子有大约 8000 行 C 代码，比“Hello，World”要复杂得多尽管如此，完整的可执行文件，包括它的所有依赖项，大小还不到 1MB。即使在负载情况下，它的内存使用量也是以千字节为单位来衡量的。容器映像的总大小约为 10MB，其中包括必要数据的 3MB。

完整的源代码请看我的 [GitHub 库](https://github.com/kevinboone/solunar_ws)。这也是在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/getting-started) 上构建和部署示例的说明。

## 关于 web 服务

本文不是关于天文计算的，所以我不会描述程序的内部操作。

正如我前面提到的，`solunar_ws`组件是一个基于 REST 的 web 服务，它提供特定城市特定日期的日出和日落信息。使用以下形式的 URL 调用它:

```
http://host:8080/day/[city]/[date]
```

这个 URL 产生 JSON 格式的结果，其中`city`是类似于`london`、`minsk`或`detroit`的名字，`date`的形式是`aug 20 2020`(根据通常的 HTTP 规则进行了转义)。

`solunar_ws`组件使用 [GNU libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/) 作为其 HTTP 引擎。这是一个完善的轻量级 HTTP 库，它将传入的请求交给程序员定义的处理函数(有点像 Java 中的`Servlet`接口)。

我选择了 libmicrohttpd 库，因为它体积小，资源使用率低。另一种方法是将 web 服务作为 [Apache HTTPD](https://httpd.apache.org/) 的插件来实现。阿帕奇 HTTPD 更久经沙场，在敌对环境中可能是更好的选择。在任何情况下，计算代码都不会改变。

## 关于集装箱

web 服务驻留在 Linux 容器中(例子包括 docker、 [Podman](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools) 和 [Buildah](https://developers.redhat.com/courses/red-hat-enterprise-linux/containers-buildah) )，可以部署在 OpenShift 上。尽管我在 [Fedora](https://getfedora.org/) 上做了大部分开发，即使是最轻的主流 Fedora 映像也提供了比这个微服务所需的更多的功能。因此，容器的基础层是 Alpine Linux。

Alpine 的基本层只有大约 6MB 大小。这是因为它使用 BusyBox 来提供外壳和实用程序，而这些是针对 [MUSL C 库](https://musl.libc.org/)而不是 [glibc](https://www.gnu.org/software/libc/) 构建的。MUSL 是一个最小的，符合 POSIX 的 C 标准库。原则上，没有什么可以阻止我们将 web 服务应用程序链接到 glibc，并在映像中包含 glibc 二进制文件。然而，这个练习的全部目的是创造尽可能小的图像；在第二个标准 C 库中包含额外的 3 兆字节无助于我们实现这个目标。

## 挑战

用 C 语言为 Alpine 构建微服务面临一些挑战，我将在接下来讨论这些挑战。然而，由于编写高质量、可服务的 C 代码所涉及的挑战已经众所周知，它们超出了本文的范围。

### 巴布克斯

Alpine 映像使用 BusyBox 而不是 GNU 核心实用程序(coreutils ),这对我们如何构建容器有潜在的影响。使用一个充满链式 shell 命令的 Dockerfile 文件来构建容器映像是很常见的。使用轻量级基础层原则上不会改变这种做法，但是命令可能会有所不同或者有不同的选项。实际上，我在这个领域没有发现任何重大问题——这是容器开发中问题较少的方面之一。像`cp`和`wget`这样的命令按预期工作。BusyBox 有自己的方式来完成系统设置任务，比如添加用户和组，但它们在概念上与我们习惯的方式没有什么不同。

### 阿尔卑斯附属地

构建容器几乎总是涉及到将某些依赖项导入到映像中。Alpine 有自己的库，有自己的安装命令(`apk add`)。

然而，一个问题是 Alpine 库中的一些库是针对 glibc 构建的，所以天真地使用`apk add`会引入大量额外的二进制文件。解决这个问题是可能的，但是我发现从源代码构建大多数依赖项比从存储库中导入更容易。我稍后将回到这一点。

`solunar_ws`组件只有两个重要的依赖项:`libmicrohttpd`，我们将从源代码构建它，以及`tzdata`—[全球时区数据库](https://developers.redhat.com/blog/2020/04/03/whats-new-with-tzdata-the-time-zone-database-for-red-hat-enterprise-linux/)。后者不是可执行文件，也没有子依赖项，所以我们可以从 Alpine 存储库中安全地安装它。

然而，一般来说，当目标是一个真正小的容器时，非常小心地使用存储库来构建映像是值得的。

### MUSL

Alpine 的核心实用程序都链接到 MUSL，而不是 glibc，默认情况下，Alpine 不包含其他 C 库。对于我们这些在开发 Linux 时已经习惯了 glibc 扩展的人来说，使用 MUSL 有点问题。让我们举几个例子。首先，MUSL 没有 glibc `qsort_r()`函数的等价物，该函数用于对任意数据结构进行排序。老实说，我甚至没有意识到这是一个扩展，直到我开始与 Alpine 合作。其次，MUSL 在履行某些职能方面存在一些无法解释的差距。例如，格式化时间数据的`strftime()`函数缺少 glibc 实现所具有的说明符。

只要我们了解这些怪癖，我们就可以解决它们。在目标平台上进行常规测试是无可替代的，无论是在容器中还是在虚拟机中。

### TLS 问题

如果您需要加密到微服务的 HTTP 流量，那么您需要决定它是需要在 OpenShift 集群中加密，还是只需要在 OpenShift 集群中加密。加密到集群的流量很简单，因为我们可以配置一个 OpenShift 路由来进行边缘终止。在这种配置中，OpenShift 路由器和微服务之间的内部流量将是纯文本。

另一方面，如果您希望即使在 OpenShift 集群中也能加密流量，您需要为微服务提供自己的传输层安全性(TLS)支持。libmicrohttpd 库支持 TLS，但是为了实现这种支持，我们需要用许多 GNU TLS 库的开发版本来构建它。当然，这些库也必须在运行时对容器可用。

此外，您需要提供一个服务器证书，以及一个让客户端管理员获得该证书的方法。您可以在 OpenShift secret 或 ConfigMap 中提供证书，并将其作为文件挂载到 pod 的文件系统中。这种技术相对来说比较常见，在 C 中使用它与在 Java 或任何其他语言中使用它在原则上没有任何不同。

与不同的是，Java 虚拟机(JVM)隐式地提供 TLS 支持，但是 C 开发人员必须在构建和运行时安装和配置必要的依赖项。为了简单起见，我假设`solunar_ws`使用边缘终止，所以它不包含或不需要自己的 TLS 支持。

## 开发超轻容器

我们可以在主流桌面或服务器 Linux 安装上，甚至在使用 WSL Linux 子系统的 Windows 10 上，进行基于 C 的微服务的大部分开发和测试。然而，正如我所说的，认为可以在不改变应用程序代码的情况下改变标准 C 库是错误的。为 Alpine/MUSL 开发确实需要在该平台上进行定期测试，无论是在虚拟机(VM)中还是在容器中。

### 在虚拟机中测试

在 Fedora 和其他平台上的 VirtualBox VM 中运行 Alpine 非常容易，甚至在 Microsoft Windows 上也是如此。谨慎使用共享文件夹或网络存储允许在不同的环境之间共享源代码。这是一种构建类似容器的环境的简单而直观的方法:在桌面上进行大部分开发，并在具有类似平台配置的 VM 中进行增量测试。

我们可以在根本不构建容器的情况下完成大部分开发工作，因为我们知道容器的平台层与 VM 中的平台层基本相同。当然，你必须在某个点上构建一个容器。假设应用程序在容器中的行为与在 VM 中的行为相同当然是不安全的，即使有相同的平台配置。尽管如此，在兼容的 VM 中进行开发会限制您需要执行构建容器的耗时过程的频率。

### 在开发容器中测试(可能还有开发)

如果您乐于使用控制台工具和脚本，那么完全有可能通过用您需要的所有开发和文件共享工具构建 Alpine 容器，直接在该容器中进行开发工作。您可以创建这样的 docker 文件:

```
FROM alpine:3.12

RUN apk add git build-base rsync && \
  addgroup -g 1000 mygroup && \
  mkdir /myuser && \
  adduser -G mygroup -u 1000 -h /myuser -D myuser && \
  chown -R myuser:mygroup /myuser && \

USER myuser

CMD ["/bin/sh"]

```

这段代码定义了一个以 Alpine 为基础的容器图像。然后，它添加了在命令行进行开发和将文件从一个地方复制到另一个地方所需的工具。它还定义了具有工作目录的单个用户。当然，这只是设置开发容器的一种方式——还有许多其他方式。

如果您交互式地运行这个容器(例如，使用`podman -it`)，那么您在容器中有一个交互式会话，您可以使用它来编辑和构建您的代码。当然，如果您想使用复杂的交互式开发工具，您将需要一个更加复杂的容器设置。

理解容器的存储是短暂的是至关重要的:虽然用户`myuser`可以读写`/myuser`目录中的文件，但是这些文件不会被保留。即使是有经验的开发人员有时也会忘记这一点，导致不愉快的结果。

我通常使用 Git 存储库作为我正在处理的代码的权威来源，无论它是在容器中还是其他地方。当在没有持久存储的环境中工作时，使用存储库尤其重要。

## 构建生产容器

构建用于测试目的的开发容器很容易，但最终，我们希望构建尽可能最轻的容器。我们当然不想包含开发工具或源代码。至少有两种方法可以构建这种类型的生产容器。

首先，我们可以使用具有适当操作系统版本(在本例中是 Alpine)的虚拟机，或者使用填充了开发工具的容器来构建二进制文件。我们使二进制文件在存储库中可用，然后创建一个 docker 文件来检索二进制文件。

其次，我们可以使用多阶段构建，并从开发容器生成生产容器。只要应用程序的源代码可用，这就是一个完全独立的操作。这是一个慢得多的构建操作，但是它的优点是从源代码库直接进入生产容器。这种方法大大减少了版本错误的机会。

对于标准的容器构建工具来说，多阶段构建相对较新，所以我将提供一个例子。有关更多信息，请参见 docker 文档。

## 多阶段集装箱建造

多阶段容器构建使用一个阶段的输出作为下一个阶段的输入。以下是`solunar_ws`的文档框架:

```
FROM alpine:3.12

RUN apk add git build-base tzdata zlib-dev && \
  get https://ftp.gnu.org/gnu/libmicrohttpd/libmicrohttpd-latest.tar.gz && \
  tar xfvz libmicrohttpd-latest.tar.gz && \
  (cd libmi*; ./configure; make install) && \
  git clone https://github.com/kevinboone/solunar_ws.git && \
  make -C solunar_ws
  # Binary solunar_ws ends up in / directory
  ...

  FROM alpine:3.12

  RUN apk add tzdata

  COPY --from=0 /solunar_ws/solunar_ws /
  COPY --from=0 /usr/local/lib/libmicrohttpd.so.12 /usr/local/lib

  USER 1000
  CMD ["/solunar_ws"]

```

### 一期建筑

第一阶段构建使用所需的所有构建工具填充基于 Alpine 3.12 的容器映像。然后它下载 libmicrohttpd 的源代码并编译它，然后用`solunar_ws`做同样的事情。这些资源来自不同的地方，但它们都是以相同的方式编译的。在这个例子中，注意我们必须在构建 web 服务之前构建 libmicrohttpd 这是因为 web 服务依赖于它。

此第一阶段映像的大小约为 210MB，可能需要 30 秒到 1 分钟的时间来构建，具体取决于互联网带宽。

### 二期建筑

第二阶段从同一个 Alpine 3.12 基础层开始，只安装运行时需要的包——在本例中为`tzdata`。然后，它从之前的构建中复制容器在运行时需要的两个文件:二进制文件`solunar_ws`和库`libmicrohttpd.so.12`。

当 Alpine Linux 存储库已经有了一个二进制包时，我们为什么需要从源代码构建 libmicrohttpd 呢？原因是为了消除所谓的依赖性蔓延。存储库中的二进制包有将近 20MB 的依赖项，这个应用程序不需要这些依赖项。这种“蔓延”是使用通用存储库的一个相对常见的副作用。从源代码构建的一个替代方法是安装二进制包，然后挑选出我们需要的特定依赖项。在这种情况下，这种方法不像从源代码构建依赖项那样容易，但有时可能是这样。

在这个例子中，我使用了`USER 1000`而没有定义任何用户。这在生产容器中是合理的，因为运行中的进程永远不需要修改容器中的任何文件。

在这种情况下，第二级的输出是大约 10MB 大小的图像。

## 在 OpenShift 上部署

我不想说太多关于部署的内容，因为对于 C 应用程序和 Java 应用程序或任何其他应用程序来说，这并没有本质上的不同。

一旦我们有了生产容器映像，有许多方法可以在 OpenShift 上部署它。事实上，在 OpenShift 4 上，我们可以直接从 Dockerfile 构建，前提是 Dockerfile 需要的所有资源都在存储库中。

`solunar_ws`源码包中的 README 文件大概解释了如何将构建好的生产 pod 从开发系统推送到[红帽码头](http://www.quay.io)仓库。一旦映像在存储库中，您就可以使用`oc new-app`创建一个默认部署:

```
$ oc new-app --docker-image=quay.io/kboone/solunar_ws:latest \
  --name=solunar-ws -l app=solunar-ws
```

为了更好地控制，您可以通过在 YAML 文件上运行`oc create -f`来创建部署配置，如下所示:

```
 kind: DeploymentConfig
    apiVersion: apps.openshift.io/v1
    metadata:
      name: solunar-ws
    spec:
      replicas: 1
      strategy:
	type: Rolling
      selector:
	name: solunar-ws
      template:
	metadata:
	  name: solunar-ws
	  labels:
	    name: solunar-ws
	spec:
	  containers:
	    - env:
	      - name: SOLUNAR_WS_LOG_LEVEL
		value: "1"
	      name: solunar-ws
	      image: quay.io/kboone/solunar_ws:latest
	      imagePullPolicy: Always
	      ports:
		- containerPort: 8080
		  protocol: TCP
	      livenessProbe:
		failureThreshold: 3
		initialDelaySeconds: 30
		periodSeconds: 10
		successThreshold: 1
		tcpSocket:
		  port: 8080
		timeoutSeconds: 1
	      readinessProbe:
		failureThreshold: 3
		initialDelaySeconds: 30
		periodSeconds: 10
		successThreshold: 1
		tcpSocket:
		  port: 8080
		timeoutSeconds: 1
	      resources:
		limits:
		  memory: 128Mi
	      securityContext:
		privileged: false
    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: solunar-ws
    spec:
      ports:
	- name: solunar-ws
	  port: 8080
	  protocol: TCP
	  targetPort: 8080
      selector:
	name: solunar-ws

```

我已经设置了一个简单的 TCP 端口测试来检测活跃度和就绪性，因为应用程序可能会在这个端口打开后的一毫秒内就绪。

## 结果

要使用浏览器测试 web 服务，您需要将`solunar-ws`服务公开为一个路由。您可以使用 OpenShift 控制台、`oc create route`命令或许多其他方式来完成此操作。图 1 显示了浏览器中的 JSON 输出。

[![The program's output as viewed in a browser.](img/d9aa3ed5f0f950ec76419fa9e99f6cfa.png "solunar-ws output")](/sites/default/files/blog/2020/08/solunar-ws-output.png)

Figure 1\. Output for the solunar-ws service.

以下是运行窗格中的内存使用数据:

```
%top -S
PID VSZ VSZRW RSS (SHR) DIRTY (SHR) STACK COMMAND
1 1384 508 860 480 92 0 132 /solunar_ws

```

是的，这些内存数据是以*千字节*表示的。这些内存数据没有显示的是亚毫秒级的启动时间。

## 结束语

这个练习的目的是研究使用 C 代码和一个超轻量级的基础层，一个 web 服务可以在一个容器中做得多小。然而，我必须指出，尽管创建一个小容器*是可能的*，但它不一定是*可取的*。特别是，这个容器映像没有任何类型的诊断工具。此外，对于没有与容器的基础层完美匹配的开发环境的人来说，从这样的容器中检查核心转储将是一种不愉快的体验。与以往一样，效率和可维护性之间需要权衡。

最后，我并不提倡在安装中间件时大规模回归 C——只是说对于某些应用程序的某些部分来说，这仍然是值得考虑的。至少，研究用 C 实现这个服务所涉及的内容会让我们更加欣赏 Java。

## 承认

我要感谢 Federico Valeri 对这篇文章的有益评论，并让我相信写这篇文章是值得的。

*Last updated: August 26, 2020*