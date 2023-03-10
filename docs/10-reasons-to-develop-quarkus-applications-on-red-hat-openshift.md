# 在 Red Hat OpenShift 上开发 Quarkus 应用程序的 10 个理由

> 原文：<https://developers.redhat.com/blog/2021/01/15/10-reasons-to-develop-quarkus-applications-on-red-hat-openshift>

将 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 与 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 结合起来，为创建可伸缩、快速、轻量级的应用程序提供了一个理想的环境。Quarkus 通过工具、预建集成、应用服务等显著提高了开发人员的工作效率。本文给出了应该在 OpenShift 上开发 Quarkus 应用程序的 10 个理由。

## 原因 1:单步 OpenShift 部署

部署 Quarkus 应用程序不一定要成为 OpenShift 专家。Quarkus OpenShift 扩展自动生成 OpenShift 资源，易于上手。该扩展提供了多种部署选项，包括[吊臂](https://github.com/GoogleContainerTools/jib)、[对接器](https://quarkus.io/guides/container-image#docker)和源到图像( [S2i](https://docs.openshift.com/enterprise/3.0/creating_images/s2i.html#:~:text=Source%2Dto%2DImage%20(S2I,ease%20of%20use%20for%20developers.) )。它还创建了一个 [DeploymentConfig](https://docs.openshift.com/container-platform/4.6/applications/deployments/what-deployments-are.html) ，每当在[图像流](https://docs.openshift.com/container-platform/4.6/rest_api/image_apis/imagestream-image-openshift-io-v1.html)中检测到更改时，它就会触发自动重新部署。

下面是一个在 OpenShift 上部署 Quarkus 的简单示例:

```
//deploy JVM-based app on OpenShift
//add OpenShift extension
mvn quarkus:add-extension -Dextensions="openshift"

//application.properties
quarkus.s2i.base-jvm-image=
quarkus.openshift.expose=true

mvn clean package -Dquarkus.kubernetes.deploy=true

```

**了解更多** : [在 OpenShift 上部署夸库](https://quarkus.io/guides/deploying-to-openshift)。

## 原因 2:一步式无服务器功能部署

Quarkus 应用程序，尤其是那些编译成本机代码的应用程序，由于它们的小尺寸和快速启动时间，是无服务器应用程序的理想选择。Quarkus OpenShift 扩展也使得部署和扩展无服务器服务变得容易。作为开发人员，您不需要担心服务器供应或维护底层基础设施。您只需编写代码并将其打包到容器中进行部署。

以下是一个无服务器功能部署的示例:

```
//deploy serverless knative app on OpenShift
//add OpenShift extension
mvn quarkus:add-extension -Dextensions="openshift"

//application.propertiesquarkus.s2i.base-jvm-image=
quarkus.kubernetes.deployment-target=knative
mvn clean package -Dquarkus.kubernetes.deploy=true

```

**了解更多** : [使用 Knative via OpenShift 无服务器](https://quarkus.io/guides/deploying-to-openshift#knative-openshift-serverless)。

## 原因 3:现场编码

传统的 Java 开发工作流程是生产力的主要消耗。完成一个周期中的每个迭代可能需要几分钟。Quarkus 实时编码特性解决了这个问题。图 1 展示了在开发模式下运行时的示例工作流。

[![](img/da9d341eb82ee2f43d7be758d7e73236.png "screenshot-docs.google.com-2021.01.08-14_18_48")](/sites/default/files/blog/2021/01/screenshot-docs.google.com-2021.01.08-14_18_48.png)

Figure 1: The write and refresh development cycle in Quarkus.

以下是在开发模式下运行 Quarkus 的命令:

```
mvn compile quarkus:dev

```

给定这个命令，Quarkus 检查是否有任何应用程序源文件发生了更改。如果有，Quarkus 透明地编译更改的文件并重新部署应用程序。

**了解更多** : [用夸库斯进行现场编码](https://quarkus.io/vision/developer-joy#live-coding)。

## 原因 4:远程开发和调试

您还可以在开发模式下，在集群化的 OpenShift 或 Kubernetes 环境中远程进行实时编码。您在本地所做的任何更改都将在集群环境中立即可见。远程开发和调试允许您在运行应用程序的同一环境中创建应用程序。关键是构建一个可变的应用程序:

```
//application.properties
quarkus.package.type=mutable-jar
quarkus.live-reload.password=abc123
quarkus.kubernetes.env.vars.QUARKUS_LAUNCH_DEVMODE=true

//Deploy
mvn clean install -Dquarkus.kubernetes.deploy=true

//Start
mvnw quarkus:remote-dev -Dquarkus.live-reload.url=

```

**了解更多** : [在远程开发模式下构建 Quarkus 应用](https://quarkus.io/guides/maven-tooling#remote-development-mode)。

## 原因 5:访问 OpenShift 配置图和机密

Quarkus 包含一个`kubernetes-config`扩展，允许您使用 Kubernetes `ConfigMap` s 和 secrets 作为配置源。您甚至不必将它们安装到运行 Quarkus 应用程序的 pod 中。相反，您的 Quarkus 应用程序使用 Kubernetes 客户端直接从 Kubernetes API 服务器读取`ConfigMap`和秘密:

```
//application.properties
quarkus.kubernetes-config.enabled=true
quarkus.kubernetes-config.secrets.enabled=true
quarkus.kubernetes-config.config-maps=

mvn clean package -Dquarkus.kubernetes.deploy=true

```

**了解更多**:[quar kus kubernetes-config 扩展](https://quarkus.io/guides/kubernetes-config)。

## 原因 6:健康终点

Quarkus 使用[微概要健康规范](https://download.eclipse.org/microprofile/microprofile-health-2.1/microprofile-health-spec.html)(通过 SmallRye 扩展)来提供关于应用程序状态的信息，比如可用性和状态。这些信息在云环境中非常有用，在云环境中，自动化流程必须频繁地决定是丢弃还是重启应用程序。默认情况下，大多数 Quarkus 客户端扩展都启用了内置健康状态:

```
//add Kubernetes-config extension
mvn quarkus:add-extension -Dextensions="smallrye-health"

//validate health extension
mvnw compile quarkus:dev
curl http://localhost:8080/health/live

//org.acme.microprofile.health.SimpleHealthCheck class
package org.acme.microprofile.health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import org.eclipse.microprofile.health.Liveness;
import javax.enterprise.context.ApplicationScoped;

@Liveness
@ApplicationScoped

public class SimpleHealthCheck implements HealthCheck {

@Override
public HealthCheckResponse call() {
return HealthCheckResponse.up("Simple health check");
}
}

```

**使用 Quarkus 中的微概要健康规范了解更多** : [。](https://quarkus.io/guides/microprofile-health)

## 原因 7:应用度量支持

Quarkus 使用微米扩展来支持捕获运行时和应用程序指标。这些指标提供了对应用程序内部发生的事情的洞察。您还可以使用支持分析和可视化的工具，如 [Prometheus](https://prometheus.io/) 和 [Grafana](https://grafana.com/) 来格式化千分尺延伸度量:

```
//add Kubernetes-config extension
mvn quarkus:add-extension -Dextensions="micrometer"

//validate health extension
mvnw compile quarkus:dev

//Code snippet to discover, count, store, and record prime numbers
@Path("/")
public class PrimeNumberResource {

private final LongAccumulator highestPrime = new LongAccumulator(Long::max, 0);

private final MeterRegistry registry;

PrimeNumberResource(MeterRegistry registry) {
this.registry = registry;
// Create a gauge to obtain the highest observed prime number
registry.gauge("prime.number.max", this,PrimeNumberResource::highestObservedPrimeNumber);}

// Return the highest observed prime value
long highestObservedPrimeNumber() {
return highestPrime.get();}
}

```

**了解更多**:[quar kus](https://quarkus.io/guides/micrometer#support-for-the-microprofile-metrics-api)中的 MicroProfile Metrics API。

## 原因 8:跟踪支持

Quarkus 使用 [MicroProfile OpenTracing](https://download.eclipse.org/microprofile/microprofile-3.3/microprofile-spec-3.3.html#mp-opentracing) 规范(通过 SmallRye 扩展)为交互式 web 应用程序提供跨服务的分布式跟踪。SmallRye 扩展包括默认的 [Jaeger](https://www.jaegertracing.io/) tracer，用于监控和排除分布式系统中的事务:

```
//add Kubernetes-config extension
mvn quarkus:add-extension -Dextensions="smallrye-opentracing"

//validate health extension
mvnw compile quarkus:dev

//REST endpoints are automatically traced. Here's how to trace additional methods
import javax.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.opentracing.Traced;

@Traced
@ApplicationScoped
public class FrancophoneService {
public String bonjour() {
return "bonjour";}
}

```

**了解更多** : [在 Quarkus 应用中使用 open tracing](https://quarkus.io/guides/opentracing)。

## 原因 9:开发人员工具

你可能是为了性能而来，但你会为了开发人员的生产力而留下来。开发者工具使得在 OpenShift 上开发和部署 Quarkus 应用程序变得更加容易。这里有几个例子:

*   **IDE 支持** : Quarkus 利用 [Quarkus 语言服务器](https://github.com/redhat-developer/quarkus-ls)支持你喜欢的 IDE，包括 [VSCode](https://developers.redhat.com/blog/category/vs-code/) ，Eclipse，IntelliJ，还有 [more](https://quarkus.io/blog/march-of-ides/) 。
*   构建工具 : Quarkus 也支持 [Maven](https://quarkus.io/guides/maven-tooling) 和 [Gradle](https://quarkus.io/guides/gradle-tooling) 构建工具。
*   **Codestarts** :扩展 [codestarts](https://quarkus.io/blog/extension-codestarts-a-new-way-to-learn-and-discover-quarkus/) 包括代码示例和文档，让刚接触 Quarkus 的开发者更容易创建应用程序。

## 原因 10:与 Spring APIs 的兼容性

Spring 是开发人员的主流 Java 框架，但是 Spring 应用程序不是为像 OpenShift 这样的云原生环境设计的。另一方面，Quarkus 是为云而创建和优化的。因此，Quarkus 可以[将云资源效率降低 64%](https://www.redhat.com/en/resources/mi-quarkus-lab-validation-idc-analyst-paper) 。

如果您需要云原生的效率，但更喜欢坚持使用您知道的框架，Quarkus 提供了一个 Spring 兼容层。这意味着您可以使用您熟悉的 Spring APIs 创建应用程序，包括数据、web、配置、安全性、依赖注入等等。这里有一个 Quarkus 的 Spring web 开发的例子:

```
//Spring Web example
import java.util.List;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/person")
public class PersonController {
@GetMapping(path = "/greet/{id}", produces = "text/plain")

public String greetPerson(@PathVariable(name = "id") long id) {
String name="";
return name;
}

@GetMapping(produces = "application/json")
public Iterable findAll() {
return personRepository.findAll();
}

```

**了解更多**:针对 Spring 开发者的[夸库](https://quarkus.io/blog/quarkus-for-spring-developers/)。

## 开始使用 quartus

我希望开发人员工具、预构建集成和应用程序服务的可用性能够激发您在 OpenShift 上开发您的第一个 Quarkus 应用程序。这些附加资源将帮助您开始:

*   **互动教程**:Quarkus 主页包括许多[互动教程](https://learn.openshift.com/developing-with-quarkus/)，带你在预先配置的 OpenShift 环境中构建 quar kus 应用。
*   **生成一个 Quarkus 项目** : Quarkus 项目初始化器使得为 Quarkus 的[社区](http://code.quarkus.io)和[红帽](https://code.quarkus.redhat.com/)构建选择扩展和生成示例应用变得容易。
*   **OpenShift access**: Red Hat provides several options for accessing an OpenShift environment, including the [developer sandbox](https://developers.redhat.com/developer-sandbox) shown in Figure 2.
    [![The sandbox include quick starts, samples, and a variety of deployment options.](img/9dcf5d85bdcdf53f17d5443738c52b26.png "screenshot-console-openshift-console.apps.sandbox.x8i5.p1.openshiftapps.com-2021.01.08-16_41_51")](/sites/default/files/blog/2021/01/screenshot-console-openshift-console.apps_.sandbox.x8i5.p1.openshiftapps.com-2021.01.08-16_41_51.png)

    Figure 2: The OpenShift developer sandbox.

    [了解更多关于在您的计算机、数据中心等使用 Red Hat OpenShift 4 集群的可能性](https://www.openshift.com/try?extIdCarryOver=true&sc_cid=701f2000001OH74AAG)。