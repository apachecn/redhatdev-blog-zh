# fabric8 Maven 插件如何将 Java 应用程序部署到 OpenShift

> 原文：<https://developers.redhat.com/blog/2020/06/02/how-the-fabric8-maven-plug-in-deploys-java-applications-to-openshift>

[fabric8 Maven 插件](https://fabric8.io/)，通常缩写为 FMP，可以添加到 Maven Java 项目中，并负责将应用程序部署到 [Red Hat OpenShift](https://developers.redhat.com/openshift) 集群中所涉及的管理任务。这些任务包括:

1.  创建 OpenShift 生成配置(BC)。
2.  协调源到映像(S2I)过程，从应用程序编译的字节码创建容器映像。
3.  根据项目中的信息创建和实例化部署配置(DC)。
4.  定义和实例化 OpenShift 服务和路由。

该流程的所有相关组件都有详细的文档记录。本文汇集了一些文档资料，概述了插件的工作原理以及它生成的图像的结构——这可能会使插件更易于使用和故障排除。

## 关于版本的注释

fabric8 Maven 插件的上游版本和红帽版本略有不同。它们不仅在配置和使用方式上不同，而且在 OpenShift 上所需的设置上也不同。特别是，该插件假设某些容器映像将在 OpenShift 安装中可用。红帽和上游版本在这方面做了不同的假设。

本文重点介绍红帽版。这个插件的 OpenShift 设置在 OpenShift 3 的[中有记录，在 OpenShift 4](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.5/html-single/fuse_on_openshift_guide/index#install-fuse-on-openshift3) 的[中有记录，尽管可能会有更高的版本。](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.5/html-single/fuse_on_openshift_guide/index#install-fuse-on-openshift4)

并非所有文档中的设置都是使用部署插件所必需的——强制性的部分是安装图像流。当然，您可能需要安装的其余部分用于其他目的。

**注意:**您也可以查看这篇快速[开始使用 fabric8 Kubernetes Java 客户端的文章](https://developers.redhat.com/blog/2020/05/20/getting-started-with-the-fabric8-kubernetes-java-client/)以了解更多信息。

## 将插件添加到 Maven 项目中

要在零配置模式下使用 FMP，只需将`plugin`规范添加到 Maven `pom.xml`中:

```
<build>
  <plugins>
    <plugin>
      <groupId>org.jboss.redhat-fuse</groupId>
      <artifactId>fabric8-maven-plugin</artifactId>
      <version>${fuse.bom.version}</version>
    </plugin>
  </plugins>
  ...
```

这样做使得 Maven 操作`fabric8:deploy`、`fabric8:build`等。，可用。为了使构建和部署成为一步操作，我们可以像这样绑定各种目标:

```
    <plugin>
    ...
      <executions>
        <execution>
          <id>fabric8</id>
          <goals>
            <goal>resource</goal>
            <goal>build</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
    ....
```

**注意:**不同版本的 Maven fabric8 插件在目标之间的依赖关系上有细微差别，并不总是需要这种绑定配置。

在零配置操作模式下，Maven fabric8 插件(像 Maven 中的其他东西一样)*固执己见*。这种模式对其输入的结构以及应该如何操作做出了许多假设。然而，许多配置参数可用于调整其行为。例如，OpenShift 资源限制可以在`pom.xml`中的插件配置中设置，如下所示:

```
        <configuration>
          <resources>
            <openshiftBuildConfig>
              <limits>
                <cpu>100m</cpu>
                <memory>256Mi</memory>
              </limits>
            </openshiftBuildConfig>
          </resources>
        </configuration>
```

配置生成的 OpenShift 部署的另一种方法是在应用程序源代码中包含 YAML 片段，这将在下一节中解释。

## 开始部署

在简单的情况下，我们可以像这样向 OpenShift 发起完整的组装和部署:

```
$ mvn fabric8:deploy
```

在常规的 Maven 构建之后，fabric8 Maven 插件创建(在适当的时候)一个 OpenShift 映像和域配置。默认情况下，DC 指定一个复本(pod)。所有创建的 OpenShift 实体都将根据`pom.xml`中的 Maven 工件 ID 命名。

请注意，该插件不使用`oc`命令。然而，除非我们提供特定的配置，否则 fabric8 将使用`oc`存储的关于用户凭证和 OpenShift 名称空间的信息。这些信息通常存储在`$HOME/.kube/config`中。实际上，通常在`oc login`和`oc project`之后运行 Maven 部署。

## 部署流程

概括地说，FMP 使用二进制源到映像(二进制 S2I)过程来创建一个 OpenShift 映像，其中包含由常规 Maven 构建提供的二进制文件。在许多情况下，应用程序的二进制文件将是一个 [Java](https://developers.redhat.com/topics/enterprise-java/) *fat* (自包含)JAR。在这种情况下，S2I 进程将 fat JAR 传递给构建器映像，后者创建一个新映像。这个映像包含 fat JAR、JVM 和各种脚本。fat JAR 中的插件结果并不支持所有的应用程序类型。在某些情况下，插件在向 OpenShift 部署任何东西之前，可能有一个更重要的组装任务。

**注:**`fabric8:deploy`目标暗示`fabric8:build`、`fabric8:resource`、`fabric8:apply`。

`fabric8:build`步骤调用 OpenShift 为应用程序生成图像流。该插件创建并安装一个 OpenShift 构建配置(BC ),其名称是 Maven 工件名称加上`-s2i`。BC 指定构建的基础映像。

检查 YAML 格式的典型 BC，我们看到:

```
    strategy: 
      sourceStrategy:
        from:
          kind: ImageStreamTag
          name: fuse7-java-openshift:1.5
          namespace: openshift
      type: Source
```

BC 表明 OpenShift 将使用(二进制)源到映像策略构建映像，使用`fuse7-java-openshift`作为构建器映像。所有的 fat-JAR 项目类型都使用相同的构建器映像。

当插件创建了 BC 后，它会在其上调用一个构建。这导致构建 pod 被实例化和执行。构建窗格将具有以下形式的名称:

```
    [artifact_id]-s2i-NNN-build 

```

其中 NNN 是内部版本号。如果一切顺利，build pod 会运行到完成，并产生一个新的映像。如果这是第一次构建，它将为图像创建一个新的图像流。但是，该映像还不能实例化到 pod 中，因为没有部署配置。

不管项目类型如何，默认情况下，应用程序编译后的二进制文件最终会保存在生成的 pod 的`/deployments`目录中。如果项目类型需要，其他支持基础结构也可以放在该目录中。

`fabric8:resource`步骤生成指定如何在 pod 中实例化应用程序所需的特定 OpenShift 资源。这些资源以 YAML 格式编写，并且将始终包含一个部署配置。其他 OpenShift 资源，比如服务定义，也可以在这个阶段生成。`fabric8:resource`操作主要是本地的——它在项目的`target/`目录中生成文件。

`fabric8:apply`步骤获取由`resource`步骤生成的配置，并将其应用于 OpenShift 安装。这里的主要步骤是由 OpenShift 上的`resource`步骤生成的 DC 的实例化。这个 DC 将与 Maven 工件同名，并将构建器生成的图像指定为其容器。这一步应该会产生一个运行应用程序的 pod。

S2I 工艺的一个特点是，由建造者创造的形象来自建造者本身。生成的映像几乎是构建器的精确副本，添加了可执行的应用程序代码和一些配置。因此，由 FMP 创建的 OpenShift 映像将包含 Maven 和 Java 编译器的完整安装，尽管它们永远不会被使用。各种技术可用于后处理图像以去除这些不必要的内容。

## 发电机

Maven fabric8 插件可以生成基于 [Spring Boot](https://developers.redhat.com/topics/spring-boot/) 、卡拉夫、[、红帽 JBoss 企业应用平台(JBoss EAP)](https://developers.redhat.com/products/eap/overview) 、普通 Java 以及其他项目类型的图像。可插拔的*生成器*用于控制从 Maven 项目构建 OpenShift-ready 二进制文件的过程，并提供适当的配置。我将特别概述 Java、Spring Boot 和 Karaf 生成器，因为它们之间的相似之处和不同之处很有启发性。

除非另行配置，否则所有已安装的生成器都可用，并将通过某些项目功能激活。例如，Spring Boot 生成器被项目中存在的`spring-boot-starter`依赖项激活。如果其他更具体的生成器都没有被激活，那么这个项目可能会被视为一个普通的 Java 可执行文件。对于一个被视为普通 Java 的项目，它必须生成一个在其清单中带有`Main-Class`属性的 JAR。

如果 Maven 项目没有激活任何生成器，这个错误可能不会导致构建失败，这可能会相当混乱。构建可能看似成功，但对 OpenShift 没有任何影响。因此，您可能会看到如下警告消息:

```
 [WARNING] F8: No image build configuration found or detected
```

在某种程度上，如果插件没有选择正确的生成器，可以在配置中控制生成器的选择。每个发生器都有自己的特定配置，可用于微调其操作。除非在配置中被覆盖，否则生成器将选择要使用的构建器映像。

目前所有独立的 Java 应用程序，包括 Spring Boot，都以`fuse7-java-openshift`作为构建者。Karaf 和 EAP 应用程序有自己特定的构建器。

### Java 生成器

Java 是所有支持的项目类型中最基本的。生成器可以从任何自包含的可执行 JAR 文件中创建 Maven 部署，创建一个基本的 DC，用滚动更新策略指定单个副本(pod)。DC:

*   显示各种端口:9779 用于 Prometheus 监控工具，8778 用于 Jolokia JMX 代理。这些服务在生成的映像中是默认启用的，我将在后面解释。
*   暴露端口 8080，缺少任何其他配置。它这样做没有特别的原因，只是因为这是一个为 HTTP 请求提供服务的应用程序常用的端口。
*   不创建活动或就绪探测。生成器无法猜测这些值的合适值，即使它们存在。

### Spring Boot 发电机

Spring Boot 生成器是 Java 生成器的一个专门化，它共享大部分相同的配置。像 Java 生成器一样，Spring Boot 生成器接受一个胖罐子作为输入。但是，Spring Boot 生成器知道 Spring Boot 应用程序通常的构造方式中的某些约定。因此，它可以为此类应用提供更有效的 DC。

例如，如果`spring-boot-starter-actuator`依赖关系包含在项目中，生成器假设执行器健康检查端点可用于活性和准备就绪探测。生成的 DC 将包含以下附加配置:

```
       readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 8080
```

端口 8080 是默认端口，这可能不合适。如果致动器被启用，生成器也将从应用程序的源代码中读取`application.properties`，以确定是否有这样的设置:

```
management.port=8081
```

如果此设置存在，它将在 DC 中用于活动/就绪探测。如果 Spring Boot 配置建议，其他端口可能会暴露在 DC 中。

应该清楚的是，Spring Boot 生成器依赖于遵循关于源格式的约定的开发人员。然而，由于 Spring Boot Maven build 或多或少地强制使用了这些约定，因此使用该插件可能不需要做额外的工作。

### 卡拉夫发电机

与 Spring Boot 和 Java 生成器不同，Karaf 生成器不接受自包含的可执行 JAR 作为输入。相反，它需要一个或多个 OSGi 包。这些仍然是 JAR 文件，但是具有特定的符合 OSGi 标准的元数据，描述了包之间的交互契约。

OSGi 应用程序需要一个支持框架；这就是卡拉夫扮演的角色。

应用程序 JAR 中特定元数据的存在使得部署一个包含 Karaf 框架的独立 JAR 变得不切实际。相反，Karaf 生成器将整个 Karaf 安装复制到`target/assembly/`中。然后，它将这个设置和应用程序的 JARs 一起转移到生成的映像中。所有这些内容最终都保存在`/deployments`目录中，还有用应用程序的包启动 Karaf 的脚本。

生成的 Karaf 安装包括一个端口 8181 上的通用 HTTP 服务器。这通常不仅服务于应用程序组件，还服务于 Karaf 基础设施的一部分。此端口可用于运行状况检查，生成的 DC 将根据这些运行状况检查指定活动和就绪性探测。

## 服务和路线

正如我们已经看到的，各种生成器将公开 OpenShift DC 中的端口，或者基于通过探测项目发现的信息，或者基于公共缺省值。这些端口分配可以用多种方式覆盖，如 FMP 文档中所述。

然而，仅仅公开 DC 中的端口并不能使应用程序对外部可用。为此，我们需要创建 OpenShift 服务和路由。默认情况下，FMP 生成器假设有一个 web 端口作为服务和路由的基础。对于 Karaf 应用程序，生成器创建了 OpenShift DC *和*应用程序的 HTTP 基础设施。因此，插件总是能够正确地定义服务——当然，前提是开发人员确实想要公开 HTTP 服务。

Spring Boot 生成器假设应用程序将公开一个 HTTP 服务，它要么在端口 8080 上，要么在`application.properties`中指定。同样，只要有一个服务，并且它实际上应该被公开，生成器就会创建正确的定义。

对于普通的 Java 项目，生成器只是猜测服务应该在端口 8080 上公开。如果这个设置不正确，您将需要覆盖生成器的行为或者指定您自己的服务定义。当然，其他发电机也可以这样做。

在`fabric8:resource`步骤中，服务定义以 YAML 格式在`target/classes/META-INF/`目录中生成。它们在`fabric8:apply`步骤中安装在 OpenShift 上。当然，这些单独的步骤很可能包含在对`fabric8:deploy`的一次调用中。

**注意:**尽管生成的映像将包括 Prometheus 和 Jolokia 代理——每个代理都有一个 HTTP 端口——但在默认情况下，这些代理不被定义为服务，因为它们完全用于点对点通信。

默认情况下，FMP 创建服务的方式也是自动创建路径。实例化的服务定义包含以下部分:

```
    metadata:
      labels:
        expose: "true"
```

自动创建的路由将被解密。这种设置通常不是必需的，而且对于处理除 HTTP 之外的任何其他协议的任何应用程序来说，绝对不是必需的。为什么？如果没有 TLS 加密通信中的服务名标识(SNI)信息，OpenShift 路由器就无法路由其他协议。

可以配置 FMP 来创建其他类型的路径，或者根本不创建路径。该功能在 [fabric8 Maven 插件文档](https://maven.fabric8.io/)中有描述。

## 使用 YAML 片段的配置

我们已经看到了 FMP 如何生成一个缺省值合理的开放式 DC。然而，通常有必要对生成的 DC 至少做一点小小的修改。在某种程度上，可以对`pom.xml`中的插件配置进行这些修改，但是更灵活的方法是随应用程序一起提供完整或部分的 DC。

在大多数情况下，提供完整的 DC 既不方便也不合适。相反，FMP 会将来自文件`src/main/fabric8/deployment.yml`的 YAML 代码片段合并到它从 Maven 项目生成的 DC 中。合并是分层次进行的:我们可以通过将更改放在层次结构中的适当位置来对 DC 的多个部分进行添加或修改。

下面是一个为 pod 指定资源限制的`deployment.xml`示例:

```
    spec:
      template:
        spec:
          containers:
            -
              resources:
                requests:
                  cpu: "0.2"
                  memory: 128Mi
                limits:
                  cpu: "1.0"
                  memory: 512Mi
```

下面是一个设置环境变量的示例:

```
    spec:
      template:
        spec:
          containers:
            -
              env:
              - name: JAVA_OPTIONS
                value: '-verbose:gc'
```

注意:这里的 YAML 语法有点复杂。我们经常需要小心地添加相关的部分，而不是完全替换它们。

在 DC 中设置环境变量的能力可能很重要，因为应用程序不直接控制 JVM 配置——这是由生成的映像中的脚本完成的，我将在后面解释
。

## 生成的图像

生成的映像将包含 JVM、应用程序的二进制文件、由 FMP 生成器创建的任何支持基础设施，以及启动应用程序的脚本。出于我前面描述的原因，它还将包含运行时不使用的构建工具，并且您可能希望在生产部署时删除这些工具。

对于所有项目类型，映像配置为通过运行脚本开始执行:

```
/usr/local/s2i/run
```

该脚本的内容因项目类型而异。对于 fat-JAR 项目，脚本将调用:

```
/opt/run-java/run-java.sh
```

使用环境变量，`run-java.sh`脚本是高度可配置的；但是，除非给出一个特定的应用程序，否则它会在`/deployments`目录中搜索一个可执行的 JAR，并运行它。构建映像时，S2I 进程将应用程序的 JAR 放在该目录中。

相比之下，Karaf 发生器创建的图像执行:

```
/deployments/karaf/bin/karaf
```

也就是说，该映像运行 Karaf 框架，该框架加载应用程序的 OSGi 包。

无论项目类型如何，JVM 的执行都是由环境变量控制的。虽然这些变量都有文档记录，但是文档分布在不同的源上，登录到正在运行的 pod 并检查脚本以查看它们接受什么配置可能更容易。然后，环境变量可以写入 DC，正如我上面解释的那样。

不管项目类型如何，默认情况下，生成的 JVM 调用将为 Prometheus 监控框架和 Jolokia JMX 代理安装 Java 代理。这两个代理的操作都由构建器映像中的配置文件控制，不容易更改。然而，这两个代理都被配置为集成到 Red Hat 的 OpenShift 监控和管理框架中，因此更改配置可能会适得其反。

`run-java.sh`脚本提供了默认的 JVM 配置设置，广泛适用于在容器环境中运行。它对容器的资源限制进行一些相当复杂的询问，以计算出例如要分配的垃圾收集器线程的数量。没有为 JVM 堆大小设置具体的限制；比如没有`-Xmx`设置。这种设置通常适用于容器环境，在这种环境中，JVM 是容器中唯一运行的进程，可以访问容器的所有内存。但是，有时微调堆管理设置可能是合适的，例如通过将不同的内存部分分配给不同的堆代。如果需要，可以通过环境变量进行这些设置。

## 摘要

fabric8 Maven 插件自动化了许多相当复杂的任务，并且可以适应许多不同的基于 Java 的应用程序。然而，如果我们把它分解成单独的步骤，它的操作是可以理解的。

*Last updated: June 25, 2020*