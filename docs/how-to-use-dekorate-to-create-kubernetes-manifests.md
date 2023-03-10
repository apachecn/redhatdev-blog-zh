# 如何使用装饰创建 kubernetes 清单

> 原文：<https://developers.redhat.com/blog/2019/08/15/how-to-use-dekorate-to-create-kubernetes-manifests>

“编写一次，到处运行”是 Sun Microsystems 创造的一个口号，用来说明 [Java](https://developers.redhat.com/topics/enterprise-java/) 的跨平台优势。在云原生世界中，这个口号比以往任何时候都更加准确，虚拟化和容器进一步加大了代码和硬件之间的距离。但是这种转变对开发者意味着什么呢？

开发人员需要负责将他们的应用程序容器化，还需要为 Kubernetes 提供一组清单。在本文中，我们将关注后者，更具体地说，关注如何使用[de Korea](https://github.com/dekorateio/dekorate)以最小的努力创建和维护这些清单。

### 装饰

Dekorate 是一组清单生成器、装饰器和工具，使得创建 Kubernetes 清单像在类路径中添加一个 jar 一样简单。它基于 Java 注释处理器，这意味着它可以与任何支持注释的 JVM 语言一起工作，而不管您使用的构建工具是什么(与 Maven、Gradle、Basel 等一起工作)。

de Korea 支持 vanilla [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat Openshift](https://developers.redhat.com/openshift/) 以及流行的扩展(例如[服务目录](https://svc-cat.io)和操作者(例如 [Prometheus](https://prometheus.io) 和 [Jaeger](https://www.jaegertracing.io) )。更令人兴奋的是，它为 [Spring Boot](https://developers.redhat.com/blog/category/spring-boot/) 提供了类支持，这意味着它可以理解代码中出现的 beans、扩展和 Spring Boot 配置，并相应地调整生成的清单。例如，它可以检测到 [Spring Cloud Kubernetes](https://github.com/spring-cloud/spring-cloud-kubernetes) 的存在，并配置所需的角色绑定和所需的服务帐户。

#### 入门指南

最简单的开始方法是在类路径中添加对应于您的目标平台的 jar。Kubernetes 用户需要:

```
<dependency>
  <groupId>io.dekorate</groupId>
  <groupId>kuberentes-annotations</groupId>
  <version>0.7.5</groupId>
</depndency>

```

Spring Boot 用户可能更喜欢包含所有与 Spring Boot 集成相关的模块的“starter”模块。

```
<dependency>
  <groupId>io.dekorate</groupId>
  <groupId>kubernetes-spring-starter</groupId>
  <version>0.7.5</groupId>
</depndency>

```

Openshift 用户希望使用:

```
<dependency>
  <groupId>io.dekorate</groupId>
  <groupId>openshift-spring-starter</groupId>
  <version>0.7.5</groupId>
</depndency>

```

在编译过程中，[decorate](https://github.com/dekorateio/dekorate)将被编译器触发，并将在`target/classes/META-INF/dekorate`下生成 Kubernetes 或 Openshift(甚至两者都有，基于哪些模块被添加到类路径中)清单。然后，这些清单可以直接应用到集群，以便部署应用程序。

```
kubectl apply -f target/classes/dekorate/kubernetes.yml

```

或者

```
oc apply -f target/classes/dekorate/openshift.yml

```

#### 修饰生成的清单

生成的清单将包含一个基本的*部署*资源(或者在 Openshift 的情况下包含*部署配置*)。清单还可能包含额外的配置或资源，这取决于 Dekorate 能从您的代码中得到什么。例如，如果 de Korea 检测到一个 HTTP 端点，HTTP 端口将被添加到容器中，并作为一个*服务*公开。

这些清单的进一步定制/修饰可以使用:

*   释文
*   应用程序配置
*   以上两者

### **使用注释**

所提供的注释允许用户指定几乎任何与应用程序的部署清单相关的内容，例如:

*   注释和标签
*   港口
*   环境变量
*   卷和装载
*   边车
*   更大的

以下是如何添加标签的示例:

```
import io.dekorate.kubernetes.annotation.KubernetesApplication;

@KubernetesApplication(labels=@Label(key="foo", value="bar"))
public class Main {
   public static void main(String[] args) {
       //your code here.
   }
} 

```

Openshift 的模拟:

```
import io.dekorate.openshift.annotation.OpenshiftApplication;

@OpenshiftApplication(labels=@Label(key="foo", value="bar"))
public class Main {
   public static void main(String[] args) {
       //your code here.
   }
} 

```

使用注释来修饰清单的好处在于配置的有效性由编译器来验证。除此之外，现代 ide 还提供了语法高亮和代码完成功能，帮助用户进行配置。

与编写长长的 JSON 或 YAML 相比，使用注释来指定小块的配置可能是更好的体验。

**在没有主类的情况下工作**

对于以主类为特色的 Java 应用程序(简单的 Java 应用程序、Spring Boot 等)，上面的代码示例非常简单。不一定有主类的应用程序框架怎么办？

不需要在主类中添加注释。任何类都可以，即使是一个空类也可以。

### **使用应用程序配置**

开发人员通常更喜欢能够将配置具体化。其他人喜欢将代码与配置完全分开。为了满足这些需求，Dekorate 使应用程序配置成为可能。

目前，该功能仅适用于 Spring Boot。对于 Spring Boot，应用程序通过`application.properties`或`application.yml`进行配置。de Korea 将读取这两个文件，并处理所有带有`dekorate`前缀的属性。

使用`application.properties`添加`foo=bar`标签:

```
dekorate.kubernetes.label[0].key=foo
dekorate.kubernetes.label[0].value=bar

```

或者使用`application.yaml`:

```
dekorate:
  kubernetes:
    label:
    - key: foo
      value: bar

```

支持属性的完整列表可在:[https://github.com/snowdrop/dekorate/blob/master/config.md](https://github.com/snowdrop/dekorate/blob/master/config.md)找到

### **同时使用**

如果您想通过标准的应用程序配置机制使用注释和覆盖值，这也是可能的。

#### 测试生成的清单

为了测试应用程序和生成的清单，de Korea 提供了 junit5 扩展，这使得:

*   构建应用程序容器。
*   将应用程序部署到目标集群。
*   等到应用程序准备好。
*   在测试代码中注入应用程序元数据。

```
import io.dekorate.deps.kubernetes.client.KubernetesClient;
import io.dekorate.deps.kubernetes.api.model.Pod;
import io.dekorate.testing.annotation.Inject;
import io.dekorate.testing.annotation.Named;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertNotNull;

@KubernetesIntegrationTest
public class SpringBootOnKubernetesIT {

  @Inject
  private KubernetesClient client;

  @Inject
  Pod pod;

  @Test
  public void testApplication() throws Exception {
    assertNotNull(client);
    // test the application....
  }

```

Dekorate 提供了许多以高级集成测试为特色的例子。更多详情，请查看:[https://github.com/snowdrop/dekorate/tree/master/examples](https://github.com/snowdrop/dekorate/tree/master/examples)。

#### 构建和部署

Dekorate 专注于清单生成和维护。构建容器映像并部署它们并不是它的主要焦点。然而，为了使开发人员的生活更容易，它确实提供了钩子，用于编译阶段的构建、推进到注册表和部署。

### **Docker 构建**

对于 [Docker](https://docker.io) 构建，假设`Dockerfile`在模块根中，`docker`二进制在路径中，Dekorate 可以与以下标志一起使用:

构建容器图像:

```
mvn clean package -Ddekorate.build=true

```

在实际的 maven 构建之后， [Dekorate](https://github.com/dekorateio/dekorate) 将使用`docker`二进制文件来执行 docker 构建。

推送容器图像:

```
mvn clean package -Ddekorate.push=true

```

在实际的 maven 构建之后，将立即执行`docker`推送。

部署生成的清单:

```
mvn clean package -Ddekorate.deploy=true

```

构建完成后，生成的清单将被部署到目标集群。此操作假设路径中存在`kubectl`或`oc`，并且存在指向目标集群的有效`~/.kube/config`文件。

#### **S2i 构建**

对于 [Openshift](https://openshift.com) ，处理容器构建更加简单，因为不需要`Dockerfile`。当`openshfit-annotations`或 [Openshift](https://openshift.com) 启动器之一与`-Ddekorate.build=true`一起使用时，de Korea 将生成一个`BuildConfig`和所需的`ImageStream`资源，用于执行 s2i 二进制编译。该操作要求路径中存在`oc`二进制文件。

#### 其他功能

仅仅一篇博客文章甚至无法列举所有可用的特性。不过值得一提的是，它确实提供了与流行运营商(例如[普罗米修斯](https://prometheus.io)、[耶格](https://www.jaegertracing.io)、[组件](https://github.com/snowdrop/component-operator))和扩展的集成，并且这个列表正在快速增长。

### 框架集成

de Korea 确实提供了与 Spring Boot 等应用框架的集成。不过也有基于 Dekorate 为 Kubernetes 提供原生资源生成的框架，比如 [Quarkus](https://quarkus.io) 。

Quarkus 为 Kubernetes 提供了自己的扩展。该扩展不使用注释，而是与其余可用的扩展进行交互，以尽可能多地检索应用程序的信息(例如，服务器端口、健康检查等)。一旦收集了所有需要的信息，它就使用 de Korea 定制生成的清单。您可以在 Quarkus Kubernetes extension 找到更多信息。

### 技术预览

作为 Red Hat OpenShift 应用程序运行时( [RHOAR)](https://developers.redhat.com/products/rhoar/overview) )的一部分，Dekorate 的一个精简功能集也可以作为“技术预览”使用。

*   技术预览版包括下列模块:[https://github.com/snowdrop/dekorate](https://github.com/snowdrop/dekorate)。
*   技术预览发布神器可以在:[https://maven.repository.redhat.com/ga/io/dekorate](https://maven.repository.redhat.com/ga/io/dekorate)找到。

### 结论

de Korea 是一个相当新的项目，它负责 Kubernetes 和 Red Hat Openshift 清单管理，适用于所有 JVM 语言和所有构建工具。它只需要尽可能少的努力就可以开始，并且它提供了两种类型的配置:使用注释和使用应用程序配置。它提供了广泛的集成模块与框架，扩展和运营商和它的快速增长。

尽情享受吧！

*Last updated: January 19, 2022*