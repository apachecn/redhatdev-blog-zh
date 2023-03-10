# 在 Red Hat OpenShift 中向微服务添加密钥库和信任库

> 原文：<https://developers.redhat.com/blog/2020/06/05/adding-keystores-and-truststores-to-microservices-in-red-hat-openshift>

您可能不需要在同一个集群中的微服务之间进行基于安全套接字层(SSL)的通信，但是如果您想要连接到远程 web 服务或消息代理，这通常是必需的。在您将公开 web 服务或其他端点的情况下，您可能还必须在部署在 [Red Hat OpenShift](https://developers.redhat.com/openshift) 上的微服务中使用自定义密钥库，以便外部客户端只与特定的信任库连接。

在本文中，我将向您展示如何为使用 [Spring Boot](https://developers.redhat.com/topics/spring-boot/) 构建的基于 [Java](https://developers.redhat.com/topics/enterprise-java/) 的微服务配置密钥库和信任库。我使用了来自[红帽 Fuse](https://developers.redhat.com/products/fuse/overview) 的 Apache Camel 和 CXF 库来开发微服务。我使用了源到图像(S2I)部署，并在 Red Hat OpenShift 4.3 中测试了这些示例。

## 关于示例应用程序

第一个示例应用程序是部署在 OpenShift 4.3 中的基于 REST 的 web 服务，它通过 SSL 进行通信。第二个示例应用程序是一个与远程安全 web 服务连接的[客户端。我在 GitHub 上托管了这两个应用程序和本文的所有示例文件。](https://github.com/1984shekhar/camel-client-ssl)

我们的任务是修改微服务的`deployment-config`,以便我们可以使用密钥库或信任库挂载卷。密钥库用于后端服务，信任库用于客户端。对于双向 SSL 通信，我们可能希望同时使用这两种机制。图 1 显示了配置了信任库的客户端应用程序。

[![](img/550bc2db4d993abcd036ffeffe96d417.png "VolumeMount and Volume Configurations")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-07-23-05-30.png)VolumeMount and Volume Configurations

Figure 1\. The client configured with a truststore.

我们将介绍在 OpenShift 4.3 中保护和部署基于 REST 的 web 服务的过程。请注意，这些指令既适用于新的 OpenShift 项目，也适用于现有项目。我将使用我刚刚介绍的示例应用程序。

## 保护和部署基于 REST 的 web 服务

要在 OpenShift 4.3 中保护和部署基于 REST 的 web 服务，首先要创建密钥库和信任库。然后将它们添加到您的项目的秘密(`rest-keystore`)，如图所示:

```
$ keytool -genkey -alias mydomain -keyalg RSA -keystore keystore.jks -keysize 2048
$ keytool -export -alias mydomain -file mydomain.crt -keystore keystore.jks
$ keytool -import -trustcacerts -alias mydomain -file mydomain.crt -keystore clientkeystore.jks
$ oc create secret generic rest-keystore --from-file=keystore.jks

```

接下来，将 [SSL 配置](https://github.com/1984shekhar/restsslopenshift/blob/master/src/main/resources/application.properties#L2-L6)添加到您的`src/main/resources/application.properties`文件中(点击 config 链接了解更多细节):

```
server.port=8080
server.ssl.key-password=password
server.ssl.key-store=/mnt/secrets/keystore.jks
server.ssl.key-store-provider=SUN
server.ssl.key-store-type=JKS

```

**注意**:keystore 的路径是`server.ssl.key-store`。稍后，我们将修改 Spring Boot 微服务的`deployment-config`,用这个密钥库挂载一个卷。

### 定义 web 服务

接下来，在`src/main/resources/endpoint.xml`中，您将[定义基于 CXF 的 JAX-RS web 服务](https://github.com/1984shekhar/restsslopenshift/blob/master/src/main/resources/endpoint.xml#L38)。注意被配置为记录请求和响应的入站和出站拦截器。

**注意**:从客户端外部调用`[HelloServiceImpl.java](https://github.com/1984shekhar/restsslopenshift/blob/master/src/main/java/io/fabric8/quickstarts/cxf/jaxrs/HelloServiceImpl.java#L21)`中描述的操作。

这个示例微服务是从 [SampleRestApplication](https://github.com/1984shekhar/restsslopenshift/blob/master/src/main/java/io/fabric8/quickstarts/cxf/jaxrs/SampleRestApplication.java#L24-L25) 启动的。注意注释`SpringBootApplication`和`ImportResource`。`SpringBootApplication`注释与用`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`注释声明类是一样的。`ImportResource`注释从资源的`endpoint.xml`导入 bean 定义。

### 部署 web 服务

在 OpenShift 4.3 GUI 中执行以下操作:

1.  选择**开发者**视角。
2.  从目录 **- >** **中选择**添加** **- >** **搜索**，寻找 **CXF JAX-RS 带弹簧**，如图 2 所示。在本例中，我们使用 S2I 在 OpenShift 中部署 web 服务和客户端应用程序。[![A screenshot showing the Developer perspective -&gt; Add -&gt; From Catalog -&gt; Search -&gt; CXF JAX-RS with Spring.](img/12ad5dfd2c48b59ff3f4f4b5044fbede.png "Search 'CXF JAX-RS' from Catalog")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-05-22-56-49.png)从目录中搜索

    图 2。在开发者目录中搜索共享应用程序、服务和 S2I 影像构建器。** 

接下来，执行以下操作:

3.  选择 **CXF JAX-RS 带弹簧**模板，点击**实例化模板**。
4.  设置 **Git 存储库 URL** 并分支它。
5.  记下版本和**服务名**。其他条目将保持不变。
6.  点击页面底部的**创建**，如图 3 所示。

[![](img/5b48fccbb0df719de3c5aa71ee6e0ba0.png "Instantiate Template")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-05-22-45-55.png)Instantiate Template

Figure 3\. Instantiate your template.

7.  等待几分钟，让 OpenShift 创建`build-config`、`deployment-config`和(最后)窗格。

**注意**:在某些情况下，OpenShift 可能没有您需要的模板。参见 [Red Hat Fuse 文档](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.5/html-single/fuse_on_openshift_guide/index)将模板添加或更新至最新版本。

### 使用密钥库挂载卷

现在，您将使用密钥库挂载一个卷。为此，向您的`deployment-config` : [volumeMounts](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L114-L117) 和 [keystore.jks](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L66-L71) 添加两个条目。

将`volumeMounts`加到`deployment-config`上就创建了挂载路径。一旦完成，您就可以使用您的`rest-keystore`秘密将`keystore.jks`条目添加到挂载路径中。

#### 添加密钥库的两种方法

有两种方法可以将这些条目添加到您的`deployment-config`。您的第一个选择是编辑`deployment-config`文件并手动添加条目，如图 4 所示。

[![](img/0e02ffffac928bcd076ccb9e301ce597.png "Edit Deployment-Config")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-07-20-50-04.png)Edit Deployment-Config

Figure 4\. Edit the deployment-config file.

您的第二个选择是将`rest-keystore`秘密添加到您的项目工作量中，如图 5 所示。

[![](img/805e665837c2afc04aa49bdeb8cde570.png "Add secret as workload")](/sites/default/files/blog/2020/05/Screenshot-from-2020-05-07-20-51-19.png)Add secret as workload

Figure 5\. Add the rest-keystore secret to the project workload.

如果选择第二个选项，保存工作后检查`deployment-config`。您会发现一些条目显示密钥库是从`rest-keystore` secret 卷挂载的。您仍然需要编辑配置，使其与[卷装载](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L114-L117)和 [keystore.jks](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L66-L71) 完全匹配。尽管如此，我更喜欢使用秘密，因为它导致更少的错别字和 YAML 格式的问题。

**注意**:对于[就绪](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L81-L85)和[活跃](https://github.com/1984shekhar/restsslopenshift/blob/master/deploymentconfig.yaml#L93-L97)探测，我必须将`scheme`设置为 HTTPS。

### 访问微服务

完成这些配置后，您应该会发现您有一个运行良好的 pod:

```
$ oc get pods
s2i-fuse74-spring-boot-cxf-jaxrs-8-dq4zd 1/1 Running 0 3d7h

```

要从外部客户端访问这个微服务，您需要创建一个 OpenShift 路由，终止于`passthrough`:

```
$ oc get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
ssl-cxf-jaxrs ClusterIP None  9413/TCP 4d6h

$ oc create route passthrough --service ssl-cxf-jaxrs

$ oc get route
NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
ssl-cxf-jaxrs ssl-cxf-jaxrs-restproject.apps.openshift4b.lab.upshift.rdu2.redhat.com ssl-cxf-jaxrs passthrough None
ssl-cxf-jaxrs-route ssl-cxf-jaxrs-route-restproject.apps.openshift4b.lab.upshift.rdu2.redhat.com ssl-cxf-jaxrs None

```

最后进入路线`ssl-cxf-jaxrs`，这是一条`passthrough`。

至此，您已经为 web 服务配置了 SSL，如下所示:

```
$ curl -k https://ssl-cxf-jaxrs-restproject.apps.openshift4b.lab.upshift.rdu2.redhat.com/services/helloservice/sayHello/FIS 
Hello FIS, Welcome to CXF RS Spring Boot World!!!

```

接下来，我将快速介绍客户机应用程序，您可以设置它来连接到您的单向 SSL web 服务应用程序。

## 基于 REST 的安全 web 服务的客户端

我的客户端应用程序示例是一个基于 [Apache Camel SSL 的微服务](https://github.com/1984shekhar/camel-client-ssl) ( `camel-client-ssl`)。我在 OpenShift 中部署了微服务，使用了我刚才为 web 服务演示的相同的 S2I 方法。

对于这个例子，我还编写了一个 web 服务的[特殊版本](https://github.com/1984shekhar/Fuse7_examples/blob/master/jaxrs-ssl-example-springboot-openshift/spring-boot-cxf-jaxrs-xml-tls)，您可以在本地笔记本电脑或虚拟机(VM)上独立运行。要运行这个示例，您只需要确保您的 OpenShift 集群可以从您的虚拟机或笔记本上访问到。第一次运行应用程序时，使用`pom.xml`中的`mvn spring-boot:run`。

### 示例设置

我不会介绍配置客户机与您的 web 服务交互的整个过程。我只指出几个重要的细节。

*   首先，请注意位于 Spring Boot 的微服务的`[application.properties](https://github.com/1984shekhar/camel-client-ssl/blob/master/src/main/resources/application.properties#L11-L15)`。您将使用这个文件来定义您想要连接的服务的 IP 和端口，以及信任库的详细信息。
*   将`[camel-http4](https://github.com/1984shekhar/camel-client-ssl/blob/master/src/main/resources/spring/camel-context.xml#L9-L11)`组件与您的信任库配置和`NoopHostnameVerifier`一起使用，这样`hostname`就不会被验证。(我使用了自签名的密钥库和信任库。)
*   客户端是一条骆驼路线，有一个用于[生产者或 HTTP 客户端](https://github.com/1984shekhar/camel-client-ssl/blob/master/src/main/resources/spring/camel-context.xml#L23)的`[camel-http](https://github.com/1984shekhar/camel-client-ssl/blob/master/src/main/resources/spring/camel-context.xml#L16)`端点。
*   为了让示例正常工作，您必须使用 S2I 部署客户端微服务。在您的 OpenShift 开发者目录中搜索 Spring Boot 的 **Camel XML DSL。用指向`[camel-http](https://github.com/1984shekhar/camel-client-ssl/blob/master/src/main/resources/spring/camel-context.xml#L16)`端点的 Git URL 实例化模板。**
*   一旦部署了客户端微服务，就按照上一节所述修改`deployment-config`:
    *   在您的`deployment-config`中添加一个 [`volumeMounts`](https://github.com/1984shekhar/camel-client-ssl/blob/master/deploymentconfig.yaml#L115-L118) 条目来创建卷挂载路径。
    *   向卷挂载路径添加一个 [`keystore.jks`](https://github.com/1984shekhar/camel-client-ssl/blob/master/deploymentconfig.yaml#L66-L72) 条目。我推荐使用`rest-keystore`秘密，而不是直接修改`deployment-config`文件。

就是这样！通过这些步骤，您应该能够使用信任库从您的客户端应用程序连接到外部 SSL web 服务或 HTTPS 端点。

## 结论

我描述了为远程 web 服务或 HTTP 端点配置密钥库的步骤，以及如何为 web 服务客户机、HTTP 客户机或消息客户机配置信任库。我希望这些说明有助于您使用证书创建微服务，或者创建一个 HTTP 或消息客户端来连接外部 HTTPS REST 端点或消息代理。我还提供了将现有应用程序迁移和部署到 OpenShift 4.3 的说明。

*Last updated: April 1, 2022*