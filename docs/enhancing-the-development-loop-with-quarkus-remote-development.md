# 使用 quartus 远程开发增强开发循环

> 原文：<https://developers.redhat.com/blog/2021/02/11/enhancing-the-development-loop-with-quarkus-remote-development>

[Kubernetes](https://developers.redhat.com/topics/kubernetes) 是[云原生微服务](https://developers.redhat.com/topics/microservices)和[无服务器架构](https://developers.redhat.com/topics/serverless-architecture)的既定基础层。通过自动化应用程序的部署、扩展和管理，Kubernetes 在内环开发(本地编码、构建、运行和测试应用程序)和外环开发(集成测试、持续部署和安全性)方面改变了[开发人员的日常工作流程](https://developers.redhat.com/blog/2020/06/16/enterprise-kubernetes-development-with-odo-the-cli-tool-for-developers/)。使用 Kubernetes 的开发人员还必须为容器化、在 pods 中调试代码以及自动化测试用例做好计划。

在本文中，您将看到如何使用 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 远程开发来增强 Kubernetes 上的开发循环。我们将建立一个新的 Quarkus 项目，然后在远程 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 集群上对其进行配置，就像您在本地开发环境中一样。

## 步骤 1:创建一个新的 Quarkus 项目

我们将使用 Maven 插件通过以下命令搭建一个新项目:

```
$ mvn io.quarkus:quarkus-maven-plugin:2.1.0.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=quarkus-remote \
    -DprojectVersion=1.0.0-SNAPSHOT \
    -DclassName="org.acme.GreeterResource" \
    -Dextensions="openshift"
```

该命令生成一个包含新 Quarkus 项目的`quarkus-remote`目录。当你在`src/main/java/org/acme`中打开`GreeterResource.java`类文件时，你会看到一个简单的 RESTful API:

```
@Path("/hello")
public class GreeterResource {

   @GET
   @Produces(MediaType.TEXT_PLAIN)
   public String hello() {
       return "Hello RESTEasy";
   }
}
```

## 步骤 2:在您的本地环境中进行现场编码

Quarkus 自带内置开发模式，支持后台编译热部署。在更改了正在运行的应用程序中的代码、资源或配置后，只需刷新 web 浏览器或调用项目的 RESTful API，更改就会自动生效。要在本地运行您的应用程序，请在项目的主目录中执行以下命令:

```
$ mvn quarkus:dev
```

您将看到实时编码已被激活:

```
INFO  [io.quarkus] (Quarkus Main Thread) Profile dev activated. Live Coding activated.
```

用一个`curl`命令调用应用程序的 RESTful API 端点:

```
$ curl http:/localhost:8080/hello
```

输出应该是:

```
Hello RESTEasy
```

返回到`src/main/java/org/acme`中的`GreeterResource.java`类文件。改变`hello()`方法中的代码:

```
return "Hello RESTEasy from Local";
```

保存文件，然后使用相同的`curl`命令重新调用端点。新的输出应该是:

```
Hello RESTEasy from Local
```

调用 RESTful API 会触发对工作区的扫描。如果检测到任何变化，编译 Java 文件，重新部署应用程序，重新部署的应用程序服务于您的请求。如果您检查正在运行的 Quarkus 运行时中的日志，您应该会看到检测到的源文件:

```
INFO  [io.qua.dep.dev.RuntimeUpdatesProcessor] (vert.x-worker-thread-4) Changed source files detected, recompiling [/Users/danieloh/Downloads/quarkus-remote/src/main/java/me/daniel/GreeterResource.java]

```

当你准备好停止应用程序时，点击 **CTRL+C** 。

## 步骤 3:构建和部署可变应用程序

如果您想将内循环开发周期扩展到 Kubernetes 或 OpenShift 等远程容器环境，该怎么办？您可以在远程开发模式下配置您的应用程序，以使对本地文件的更改在远程容器环境中立即可见。

要进行远程开发，您需要使用`mutable-jar`格式构建一个可变的应用程序。然后可以使用 OpenShift 扩展和 Maven 插件将应用程序部署到远程 OpenShift 集群。将以下配置附加到 Quarkus 项目的`application.properties`文件中:

```
# Mutable Jar configurations
quarkus.package.type=mutable-jar
quarkus.live-reload.password=changeit

# OpenShift Extension Configurations
quarkus.container-image.build=true
quarkus.kubernetes-client.trust-certs=true
quarkus.kubernetes.deployment-target=openshift
quarkus.openshift.route.expose=true
quarkus.openshift.env.vars.quarkus-launch-devmode=true
```

请注意，您可以随意更改密码。它用于保护远程端和本地端之间的通信。

### 在 OpenShift 集群中打开可变应用程序

要登录到 OpenShift 集群，您必须安装`oc`命令行界面并使用`oc`登录。[CLI](https://docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/getting-started-cli.html#cli-getting-started)的安装选项会因您的操作系统而异。

假设您已经安装了`oc`，在您的 Quarkus 项目主目录中执行以下命令:

```
$ oc new-project quarkus-remote
$ mvn clean package -DskipTests -Dquarkus.kubernetes.deploy=true
```

该命令在远程 OpenShift 集群中创建新项目。可变 JAR 将被打包并部署到 OpenShift。输出应以`BUILD SUCCESS`结束。

### 在 OpenShift 控制台中检查应用程序

到目前为止，一切顺利。现在，让我们转到 OpenShift 集群中的开发人员控制台，然后浏览拓扑视图。您将看到您的 Quarkus 应用程序已经部署。点击**查看日志**查看可变应用程序是如何部署的，如图 1 所示。

[![Figure 1\. Topology View](img/a0fa90d6994fd49038b20fc42c0b428a.png "Figure 1\. Topology View")](/sites/default/files/blog/2021/01/Screen-Shot-2021-01-20-at-10.45.44-AM.png)Figure 1\. Topology View

Figure 1: The new, mutable application in the Topology view.

您应该在 pod 日志中看到输出“`Profile dev activated. Live Coding activated`”，如图 2 所示。

[![Figure 2\. Pod Logs](img/b3b5b0b1008b9d70a306785f1cdd06a6.png "Figure 2\. Pod Logs")](/sites/default/files/blog/2021/01/Screen-Shot-2021-01-20-at-10.45.55-AM.png)Figure 2\. Pod Logs

Figure 2: The pod logs show that live coding has been activated.

### 检查应用程序的 RESTful API

返回拓扑视图，访问应用程序的 RESTful API。单击图 3 中突出显示的**打开 URL** 图标。

[![Figure 3\. Open URL](img/88fc2e36b96d740b81f515a8abc72553.png "Figure 3\. Open URL")](/sites/default/files/blog/2021/01/Screen-Shot-2021-01-20-at-10.46.03-AM.png)Figure 3\. Open URL

Figure 3: Open the RESTful API URL

在应用程序的路由 URL 上追加`/hello`。当您检查它时，您应该在您的本地环境中看到相同的输出:

[![Figure 4\. Access the REST API](img/086fe97ef9a7603faf16a41c6b190e2d.png "Figure 4\. Access the REST API")](/sites/default/files/blog/2021/01/Screen-Shot-2021-01-20-at-10.46.14-AM.png)Figure 4\. Access the REST API

Figure 4\. Access the REST API

## 步骤 4:在远程开发模式下运行应用程序

最后一步是在 OpenShift 上将本地代理连接到远程主机。首先，将`quarkus.live-reload.url`配置附加到您的`application.properties`文件中。请注意，您将需要删除或注释 OpenShift 扩展配置，以便它们不会在您更改代码时触发[源到映像构建](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html#builds-strategy-s2i-build_understanding-image-builds):

```
# Mutable Jar configurations
quarkus.package.type=mutable-jar
quarkus.live-reload.password=changeit
quarkus.live-reload.url=http://YOUR_APP_ROUTE_URL

# OpenShift Extension Configurations
# quarkus.container-image.build=true
# quarkus.kubernetes-client.trust-certs=true
# quarkus.kubernetes.deployment-target=openshift
# quarkus.openshift.expose=true
# quarkus.openshift.env-vars.quarkus-launch-devmode.value=true
```

使用`remote-dev`命令执行远程开发模式:

```
$ mvn quarkus:remote-dev
```

输出应以`Connected to remote server`结束。现在，您可以在运行应用程序的相同环境中进行开发，并访问相同的服务。

接下来，返回到`src/main/java/org/acme`中的`GreeterResource.java`类文件，然后修改`hello()`方法中的代码:

```
return "Hello RESTEasy from OpenShift";
```

保存文件，然后刷新浏览器。输出应该如图 5 所示。

[![Figure 5\. Reinvoke the RESTful API](img/3db8c08e0810152199dde6eb186ecb4d.png "Figure 5\. Reinvoke the RESTful API")](/sites/default/files/blog/2021/01/Screen-Shot-2021-01-20-at-10.46.18-AM.png)Figure 5\. Reinvoke the RESTful API

Figure 5\. Reinvoke the RESTful API

每次刷新浏览器时，您应该会看到本地更改在远程应用程序中立即可见。基于 HTTP 的长轮询传输通过 HTTP 调用同步本地工作区和远程应用程序。下面是本地 Quarkus 运行时的日志输出示例:

```
INFO  [io.qua.dep.dev.RuntimeUpdatesProcessor] (Remote dev client thread) Changed source files detected, recompiling [/Users/danieloh/Downloads/quarkus-remote/src/main/java/me/daniel/GreeterResource.java]
...

INFO  [io.qua.ver.htt.dep.dev.HttpRemoteDevClient] (Remote dev client thread) Sending dev/app/me/daniel/GreeterResource.class
INFO  [io.qua.ver.htt.dep.dev.HttpRemoteDevClient] (Remote dev client thread) Sending quarkus-run.jar
...
```

厉害！在实现应用程序的新业务需求时，您应该已经准备好享受您的内循环开发体验了。

**注意**:在生产中使用 Quarkus 的远程开发模式可能会导致正在运行的应用程序发生意外的功能变化。只有在应用程序处于开发阶段时，才应该使用远程开发。

## 结论

您可以使用 Quarkus 来增强开发循环，方法是将本地机器上的实时编码特性连接到一个远程容器环境，比如 OpenShift。能够在云原生 Java 运行时中进行远程开发简化了开发工作流——从编写代码到快速构建、运行、调试和部署微服务。有关 Quarkus 如何通过统一配置、零配置和实时编码以及轻松注入扩展来实现云原生应用程序，从而优化开发效率的更多信息，请参见 [Quarkus 指南](https://quarkus.io/guides/)。

*Last updated: October 7, 2022*