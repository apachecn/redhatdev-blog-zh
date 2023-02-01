# 使用 Kubernetes 配置映射来定义您的 Quarkus 应用程序的属性

> 原文：<https://developers.redhat.com/blog/2020/01/23/using-kubernetes-configmaps-to-define-your-quarkus-applications-properties>

因此，您编写了 Quarkus 应用程序，现在您想将它部署到 Kubernetes 集群。好消息:将 Quarkus 应用程序部署到 Kubernetes 集群很容易。但是，在此之前，您需要理顺应用程序的属性。毕竟，您的应用程序可能需要连接数据库、调用其他服务等等。这些设置已经在您的`application.properties`文件中进行了定义，但是这些值与您本地环境的值相匹配，并且一旦部署到您的集群上，这些设置将不起作用。

那么，如何轻松解决这个问题呢？让我们看一个例子。

## 创建示例 Quarkus 应用程序

不要用复杂的例子，让我们用一个简单的用例来很好地解释这个概念。首先创建一个新的 Quarkus 应用程序:

```
$ mvn io.quarkus:quarkus-maven-plugin:1.1.1.Final:create
```

创建新应用程序时，您可以保留所有默认值。在这个例子中，应用程序被命名为`hello-app`。现在，打开`HelloResource.java`文件并重构它，如下所示:

```
@Path("/hello")
public class HelloResource {

 @ConfigProperty(name = "greeting.message")
 String message;

 @GET
 @Produces(MediaType.TEXT_PLAIN)
 public String hello() {
  return "hello " + message;
 }
}

```

在您的`application.properties`文件中，现在添加`greeting.message=localhost`。`@ConfigProperty`注释不在本文的讨论范围内，但是在这里我们可以看到使用这个注释在代码中注入属性是多么容易。

现在，让我们启动我们的应用程序，看看它是否如预期的那样工作:

 ````
$ mvn compile quarkus:dev
```

浏览到`http://localhost:8080/hello`，它应该输出`hello localhost`。Quarkus 应用程序到此结束。已经准备好了。

## 将应用程序部署到 Kubernetes 集群

这里的想法是将这个应用程序部署到我们的 Kubernetes 集群，并用一个可以在集群上工作的值替换我们的`greeting`属性的值。重要的是要知道`application.properties`的所有属性都是公开的，因此可以用环境变量覆盖。约定是将属性的名称转换为大写，并将每个点(`.`)替换为下划线(`_`)。例如，我们的`greeting.message`将变成`GREETING_MESSAGE`。

此时，我们几乎已经准备好将我们的应用程序部署到 Kubernetes，但是我们还需要做三件事情:

1.  创建应用程序的 Docker 映像，并将其推送到集群可以访问的存储库中。
2.  定义一个配置映射资源。
3.  为我们的应用程序生成 Kubernetes 资源。

要创建 Docker 映像，只需执行以下命令:

```
$ docker build -f src/main/docker/Dockerfile.jvm -t quarkus/hello-app .
```

确保设置正确的 Docker 用户名，并推送至图像注册表，如`docker-hub`或`quay`。如果您不能推送图像，您可以使用`sebi2706/hello-app:latest`。

接下来，创建文件`config-hello.yml`:

```
apiVersion: v1
data:
    greeting: "Kubernetes"
kind: ConfigMap
metadata:
    name: hello-config

```

确保您已连接到群集，并应用此文件:

```
$ kubectl apply -f config-hello.yml
```

Quarkus 附带了一个有用的扩展`quarkus-kubernetes`，可以为您生成 Kubernetes 资源。你甚至可以通过提供额外的属性来调整生成的资源——更多细节，请查看这个[指南](https://quarkus.io/guides/kubernetes)。

安装完扩展后，将这些属性添加到我们的`application.properties`文件中，以便它为我们的容器规范生成额外的配置参数:

```
kubernetes.group=yourDockerUsername
kubernetes.env-vars[0].name=GREETING_MESSAGE
kubernetes.env-vars[0].value=greeting
kubernetes.env-vars[0].configmap=hello-config
```

运行`mvn package`并在`target/kubernetes`中查看生成的资源。有趣的部分在`spec.containers.env`:

```
- name: "GREETING_MESSAGE"
  valueFrom:
  configMapKeyRef:
    key: "greeting"
    name: "hello-config"
```

在这里，我们看到如何使用来自 ConfigMap 的值将环境变量传递给容器。现在，只需应用资源:

```
$ kubectl apply -f target/kubernetes/kubernetes.yml
```

公开您的服务:

```
kubectl expose deployment hello --type=NodePort
```

然后，浏览到公共网址或做一个卷曲。例如，使用 Minikube:

```
$ curl $(minikube service hello-app --url)/hello
```

这个命令应该输出:`hello Kubernetes`。

## 结论

现在你知道了如何结合环境变量和你的 Quarkus 的`application.properties`来使用配置映射。正如我们在介绍中所说的，当定义一个 DB 连接的 URL(比如`QUARKUS_DATASOURCE_URL`)或者使用`quarkus-rest-client` ( `ORG_SEBI_OTHERSERVICE_MP_REST_URL`)时，这种技术特别有用。

*Last updated: June 29, 2020*`