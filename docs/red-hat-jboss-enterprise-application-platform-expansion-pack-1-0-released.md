# 红帽 JBoss 企业应用平台扩展包 1.0 发布

> 原文：<https://developers.redhat.com/blog/2020/06/17/red-hat-jboss-enterprise-application-platform-expansion-pack-1-0-released>

红帽近日发布了首个[红帽 JBoss 企业应用平台扩展包(JBoss EAP XP)](https://developers.redhat.com/products/eap/overview)1.0 版本。这个版本使 JBoss EAP 开发人员能够使用 Eclipse MicroProfile 3.3 APIs 构建 [Java](https://developers.redhat.com/topics/enterprise-java/) 微服务，同时继续支持 Jakarta EE 8。这篇文章详细介绍了这种新产品的性质以及一种简单的入门方法。

## JBoss EAP 扩展包和 Eclipse MicroProfile 简介

已经开始或正在考虑开始数字化转型之旅的组织正在评估和寻找利用其 Java EE/Jakarta EE 专业知识的方法。多年来，IT 开发和运营部门积累了丰富的 Java 专业知识，他们面临着用新技术平衡现有技能基础的挑战，例如基于[微服务](https://developers.redhat.com/topics/microservices/)、[API](https://developers.redhat.com/topics/api-management/)、[容器](https://developers.redhat.com/topics/containers/)的架构，以及反应式编程。Eclipse MicroProfile 是一个开源项目，是那些在使用熟悉的 Java EE 技术和 API 的同时支持和优化微服务开发的技术之一。

您可以将 MicroProfile 视为 Java 微服务的最低标准概要文件。与 Jakarta EE 一样，不同厂商的 MicroProfile 实现完全可以互操作。你可以在免费电子书[中阅读更多关于 MicroProfile 的内容。](https://www.redhat.com/en/resources/enterprise-java-microservices-ebook)

通过将这个扩展包与[Red Hat JBoss Enterprise Application Platform](https://www.redhat.com/en/technologies/jboss-middleware/application-platform)一起使用，该平台是 [Red Hat Runtimes](https://www.redhat.com/en/products/runtimes) 的一部分，开发人员可以将 JBoss EAP 用作符合 MicroProfile 的平台。这个版本简化了使用 MicroProfile 在 JBoss EAP 上开发云原生应用程序的固有复杂性。扩展包是一个单独的可下载发行版，可以应用在现有的 JBoss EAP 服务器上，或者当在 OpenShift 上部署 JBoss EAP 时，您可以使用可用于 [Red Hat OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift) 的容器映像。

## 测试一个示例应用程序

如文档中所述，扩展包作为*补丁*分发，该补丁使用 JBoss EAP XP *补丁管理器*应用。为了快速了解这是如何工作的，让我们来试驾一下。您将需要一些开发工具，如 [OpenJDK 11](https://access.redhat.com/products/openjdk/) 、 [Git](https://git-scm.com/downloads) 、 [Maven](https://maven.apache.org/download.cgi) 、一个文本编辑器和类似`[curl](https://curl.haxx.se/)`的实用程序，并为第一步准备好您的 Red Hat 开发人员证书:

*   **[下载 JBoss EAP 7.3.0](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform&downloadType=distributions)** :(这一步需要你的红帽开发者凭证。)保存到你的本地桌面，解压到你喜欢的任何文件夹，在这个文件夹下你会发现一个名为`jboss-eap-7.3.0`的新文件夹。为了简洁起见，我将 zip 文件解压到`/tmp`文件夹:

    ```
    $ unzip -d /tmp ~/Downloads/jboss-eap-7.3.0.zip
    ```

*   [**下载 JBoss EAP 7.3 更新 01 补丁**](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform&downloadType=patches&version=7.3) :我们将用这个文件给我们的 7.3.0 打补丁到 7 . 3 . 1(EAP XP 的必备版本)。我保存到`/tmp/jboss-eap-7.3.1-patch.zip`。
*   [**下载 JBoss EAP XP 1.0.0 Manager**](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform.xp&downloadType=distributions) :是一个 JAR 文件。我也会把这个保存到`/tmp/jboss-eap-xp-1.0.0.GA-manager.jar`。

*   **[下载 JBoss EAP XP 1.0.0 运行时分发的累积补丁发布](#)** :我也会把这个保存到`/tmp/jboss-eap-xp-1.0.0-patch.zip`。

下载完成后，让我们应用补丁:

1.  使用以下命令应用补丁以将 EAP 从 7.3.0 升级到 7 . 3 . 1:

    ```
    $ /tmp/jboss-eap-7.3/bin/jboss-cli.sh "patch apply /tmp/jboss-eap-7.3.1-patch.zip"
    ```

2.  设置 JBoss EAP 补丁管理器:

    ```
    $ java -jar /tmp/jboss-eap-xp-1.0.0-manager.jar setup --jboss-home=/tmp/jboss-eap-7.3
    ```

3.  为 JBoss EAP XP 1.0 应用补丁:

    ```
    $ /tmp/jboss-eap-7.3/bin/jboss-cli.sh "patch apply /tmp/jboss-eap-xp-1.0.0-patch.zip"
    ```

4.  使用作为补丁的一部分安装的微配置文件启动 JBoss EAP，并在服务器上启用度量:

    ```
    $ /tmp/jboss-eap-7.3/bin/standalone.sh -Dwildfly.statistics-enabled=true -c=standalone-microprofile.xml
    ```

随着我们新的 JBoss EAP plus MicroProfile 服务器的启动，让我们部署一个示例应用程序。打开一个单独的终端，然后:

1.  使用 Git 将快速入门库克隆到您的本地机器上(我也将它放在`/tmp`中):

    ```
    $ git clone https://github.com/jboss-developer/jboss-eap-quickstarts /tmp/jboss-eap-quickstarts
    ```

2.  将示例`helloworld-rs`(一个使用 JAX-RS 的简单 RESTful 应用程序)构建并部署到正在运行的 JBoss EAP:

    ```
    $ mvn clean install wildfly:deploy -f /tmp/jboss-eap-quickstarts/helloworld-rs
    ```

既然我们的示例应用程序已经部署，让我们尝试访问 MicroProfile Metrics API 来收集关于我们的应用程序和服务器的指标:

```
$ curl -s http://localhost:9990/metrics

# HELP jboss_undertow_request_count_total The number of requests this listener has served
# TYPE jboss_undertow_request_count_total counter
jboss_undertow_request_count_total{https_listener="https",server="default-server",microprofile_scope="vendor"} 0.0
jboss_undertow_request_count_total{http_listener="default",server="default-server",microprofile_scope="vendor"} 3.0
jboss_undertow_request_count_total{deployment="helloworld-rs.war",servlet="org.jboss.as.quickstarts.rshelloworld.JAXActivator",subdeployment="helloworld-rs.war",microprofile_scope="vendor"} 0.0

```

您将看到 JBoss EAP 公开的 [OpenMetrics 格式](https://openmetrics.io/)中的许多指标。如果您想在不同的指标上设置警报，这个输出可以稍后连接到[普罗米修斯](https://prometheus.io/)。我们可以根据范围和指标名称进行过滤，以仅显示我们应用的子集:

```
$ curl -s http://localhost:9990/metrics/vendor/jboss_undertow_request_count | grep helloworld-rs.war

jboss_undertow_request_count_total{deployment="helloworld-rs.war",servlet="org.jboss.as.quickstarts.rshelloworld.JAXActivator",subdeployment="helloworld-rs.war",microprofile_scope="vendor"} 0.0

```

这个输出向我们展示了来自 Undertow 子系统的针对我们的`helloworld-rs`应用程序的指标，显示没有请求。

现在访问实际的应用程序一次:

```
$ curl http://localhost:8080/helloworld-rs/rest/json

{"result":"Hello World!"}

```

让我们再次访问微文件度量，期望请求计数应该是`1.0`:

```
$ curl -s http://localhost:9990/metrics/vendor/jboss_undertow_request_count | grep helloworld-rs.war

jboss_undertow_request_count_total{deployment="helloworld-rs.war",servlet="org.jboss.as.quickstarts.rshelloworld.JAXActivator",subdeployment="helloworld-rs.war",microprofile_scope="vendor"} 1.0

```

的确如此(请看`1.0`的末尾)。之前是`0.0`。注意，为了生成指标，您必须记住使用`-Dwildfly.statistics-enabled=true`在服务器上启用统计。

### 使用微配置文件 API

在前面的例子中，我们实际上没有在`helloworld-rs`应用程序中键入或使用任何微文件 API。取而代之的是，JBoss EAP XP 的核心微配置文件功能是基于标准度量进行报告的。让我们在我们的应用程序中实际使用 OpenAPI 的一个微文件 API。

JBoss EAP XP 可以为应用程序生成 OpenAPI 文档，即使不使用 OpenAPI。运行这个`curl`命令，看看它为我们的应用程序生成了什么:

```
$ curl http://localhost:8080/openapi
---
openapi: 3.0.1
info:
  title: helloworld-rs.war
  version: "1.0"
servers:
- url: /helloworld-rs
paths:
  /rest/json:
    get:
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: string
  /rest/xml:
    get:
      responses:
        "200":
          description: OK
          content:
            application/xml:
              schema:
                type: string

```

这为我们的示例 REST APIs 输出了 [OpenAPI 格式的](https://swagger.io/specification/)文档。虽然这很有用，但是让我们用 MicroProfile OpenAPI 来改进它吧！

在此之前，我们需要将依赖项添加到我们的`pom.xml`中。首先，添加一个`<dependencyManagement>`元素来拉入微概要材料清单(BOM)。将它添加到快速入门的基目录中的`pom.xml`——在我的例子中是在`/tmp/jboss-eap-quickstarts/helloworld-rs/pom.xml`—就在现有的`<dependency>`块之前:

```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.jboss.bom</groupId>
                <artifactId>jboss-eap-xp-microprofile</artifactId>
                <version>1.0.0.GA</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

接下来，`<dependency>`块的内的*，为微文件 OpenAPI 添加一个新的依赖项:*

```
<dependency>
  <groupId>org.eclipse.microprofile.openapi</groupId>
  <artifactId>microprofile-openapi-api</artifactId>
  <scope>provided</scope>
</dependency>

```

指定了依赖关系后，我们现在可以使用 MicroProfile APIs 了。在`src/main/java/org/jboss/as/quickstarts/rshelloworld/HelloWorld.java`文件中，查找`getHelloWorldJSON()`方法，并在`@GET`注释上方添加下面一行:

```
@Operation(description = "Get Helloworld as a JSON object")

```

这添加了一个简单的 OpenAPI 注释，该注释将在`/openapi`输出中添加描述。您还需要在文件的顶部添加一个新的 import 语句:

```
import org.eclipse.microprofile.openapi.annotations.Operation;

```

保存文件，并使用以下命令重新部署应用程序:

```
$ mvn clean install wildfly:deploy -f /tmp/jboss-eap-quickstarts/helloworld-rs

```

有了新的带 OpenAPI 注释的应用程序后，再次访问 OpenAPI 端点:

```
$ curl  http://localhost:8080/openapi
---
openapi: 3.0.1
info:
  title: helloworld-rs.war
  version: "1.0"
servers:
- url: /helloworld-rs
paths:
  /rest/json:
    get:
      description: Get Helloworld as a JSON object
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: string
  /rest/xml:
    get:
      responses:
        "200":
          description: OK
          content:
            application/xml:
              schema:
                type: string

```

您可以看到在`/rest/json`端点的文档中添加了新的`description`。您可以通过添加额外的 MicroProfile OpenAPI 注释来进一步增强/完善您的 OpenAPI 文档。您将需要重新构建/重新部署这些更改，以反映在 OpenAPI 文档中。

还有许多其他的 MicroProfile APIs 可以用来增强您的应用程序，包括容错、JWT 安全性、REST 客户端等等。JBoss EAP XP 快速入门展示了如何在 JBoss EAP 上创建 Java 微服务。 [CodeReady Studio](https://www.redhat.com/en/technologies/jboss-middleware/codeready-studio) 的用户也可以使用 MicroProfile APIs，如[这篇文章](https://developers.redhat.com/blog/2020/06/16/enable-eclipse-microprofile-applications-on-red-hat-jboss-enterprise-application-platform-7-3/)中所概述的。

### 在 OpenShift 上使用 JBoss EAP XP

JBoss EAP XP 也可以通过 OpenShift S2I 映像获得，就像 JBoss EAP 本身一样。让我们部署一个示例。首先，您需要一个能够访问 registry.redhat.io 和`oc`命令行的 OpenShift 4.x 集群，并登录到您的集群。然后:

您还可以在 OpenShift 控制台的 **OpenShift 拓扑**视图中看到部署的应用:

[![OpenShift Topology View with JBoss EAP Application](img/a26f89b4b31456d7294b2ada4658f580.png "Screen Shot 2020-06-16 at 10.25.14 AM")](/sites/default/files/blog/2020/06/Screen-Shot-2020-06-16-at-10.25.14-AM-e1616785022459.png)OpenShift Topology View with JBoss EAP Application

带有 JBoss EAP 应用程序的 OpenShift 拓扑视图">

## 证明文件

JBoss EAP 指南中新的[使用 Eclipse MicroProfile 可以在](#) [JBoss EAP 文档](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/)中找到。该指南涵盖了关于 MicroProfile 的重要细节，以及开发人员如何在他们的 JBoss 项目中快速开始使用 MicroProfile APIs。你也可以在[红帽 JBoss 企业应用平台扩展包 1.0 发布说明](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/red_hat_jboss_eap_xp_1.0.0_release_notes)中找到关于发布本身的重要信息。

## 获得对 JBoss EAP XP 的支持

Red Hat 客户可以通过订阅 Red Hat Runtimes 来获得对扩展包的支持。请联系您当地的 Red Hat 代表或 Red Hat 销售人员，了解如何享受 Red Hat 及其全球合作伙伴网络提供的世界级支持。

JBoss Enterprise Application Platform expansion pack(JBoss EAP XP 或 EAP XP)受制于它自己独立的支持策略和生命周期，以便更好地与 Eclipse MicroProfile 规范发布节奏保持一致。新的 EAP XP 策略和生命周期将完全涵盖带有 EAP XP 设置的 JBoss EAP 服务器实例。

通过设置 EAP XP，您的服务器将受 EAP XP 支持和生命周期策略的约束。更多详情请参见 [JBoss 企业应用平台扩展包生命周期页面](https://access.redhat.com/support/policy/updates/jboss_eap_xp_notes)。

## JBoss EAP XP 资源

*   [红帽 JBoss 企业应用平台扩展包 1.0 发行说明](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/red_hat_jboss_eap_xp_1.0.0_release_notes)
*   [在 JBoss EAP 中使用 Eclipse micro profile](#)
*   [JBoss 企业应用平台扩展包支持和生命周期政策](https://access.redhat.com/support/policy/updates/jboss_eap_xp_notes)
*   [微档案首页](https://microprofile.io)
*   [红帽博客上的微档案](https://middlewareblog.redhat.com/category/microprofile/)
*   [JBoss EAP 7 . 3 . 0 上的 JBoss EAP XP 1.0 发行说明](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.3/html/red_hat_jboss_eap_xp_1.0.0_release_notes)
*   [在 JBoss EAP 7.3.1 上安装 JBoss EAP XP 1.0](https://github.com/jboss-developer/jboss-eap-quickstarts/tree/xp-1.0.x)

*Last updated: January 10, 2022*