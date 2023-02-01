# 在 Open Liberty 中使用 Red Hat OpenShift 的内置 OAuth 服务器作为身份验证提供者

> 原文：<https://developers.redhat.com/blog/2020/01/28/use-red-hat-openshifts-built-in-oauth-server-as-an-authentication-provider-in-open-liberty>

在[Open Liberty](https://openliberty.io/about/)20.0.0.1，你可以配置社交登录功能，使用[红帽 OpenShift](http://developers.redhat.com/openshift/) 的 OAuth 服务器进行认证。此外，还有一个新的微文件指标来衡量 CPU 时间、内存堆和响应时间。这个版本还通过 Liberty 注释缓存和更新的 JavaServer Face 提供了更快的应用程序启动。

## 使用 20.0.0.1 运行您的应用

如果你用的是 [Maven](https://openliberty.io/guides/maven-intro.html) ，这里是坐标:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.1</version>
    <type>zip</type>
</dependency>
```

如果您使用的是 [Gradle](https://openliberty.io/guides/gradle-intro.html) :

```
dependencies {
libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.1,)'
}
```

或者，如果您正在使用 [Docker](https://openliberty.io/guides/containerize.html) :

```
FROM open-liberty
```

## 通过 Red Hat OpenShift 使用 Liberty 社交登录

社交登录功能`socialLogin-1.0`现在可以配置为使用 OpenShift 的内置 OAuth 服务器和 OAuth 代理 sidecar 作为认证提供者。社交登录功能有几个预配置的提供商(如谷歌、GitHub 和脸书)，但您也可以配置其他提供商(如 Instagram)。OpenShift 的 OAuth 服务器和 OAuth 代理 sidecar 现在也可以配置为额外的提供者。第一个是标准的 OAuth 授权代码流，其中访问 Liberty 中运行的应用程序的 web 浏览器被重定向到 OpenShift OAuth 服务器进行身份验证。第二个从 OpenShift OAuth 代理边车接受一个入站令牌，或者从 OpenShift API 调用获得一个入站令牌。第二种方法需要较少的特定于集群的配置。

大多数使用这个功能的人会在一个 pod 中运行 Liberty。但是，在授权代码流中，Liberty 可以在 OpenShift 集群之外运行。在任一模式下，都可以创建一个可选的 JWT 来传播到下游服务。

使用 OpenShift 作为提供者与其他 OAuth 提供者略有不同，因为它需要一个服务帐户令牌来获取关于 OAuth 令牌的信息。一旦从 OpenShift 获得了客户机 ID、秘密和令牌，就可以配置 Liberty 了。

要使您的服务器能够使用 OpenShift OAuth 服务器，请将其添加到`server.xml`文件中，如下所示:

```
    <server description="social">

      <!-- Enable features -->
      <featureManager>
        <feature>appSecurity-3.0</feature>
        <feature>socialLogin-1.0</feature>
      </featureManager>

    <logging traceSpecification="com.ibm.ws.security.*=all=enabled" maxFiles="8" maxFileSize="200"/>

    <httpEndpoint id="defaultHttpEndpoint" host="*" httpPort="8941" httpsPort="8946" > <tcpOptions soReuseAddr="true" /> </httpEndpoint>

      <!-- specify your clientId, clientSecret and userApiToken as liberty variables or environment variables -->
      <oauth2Login id="openshiftLogin" scope="user:full" clientId="${myclientId}" clientSecret="${myclientSecret}" authorizationEndpoint="https://oauth-openshift.apps.papains.os.fyre.ibm.com/oauth/authorize" tokenEndpoint="https://oauth-openshift.apps.papains.os.fyre.ibm.com/oauth/token" userNameAttribute="username" groupNameAttribute="groups" userApiToken="${serviceAccountToken}" userApiType="kube" userApi="https://api.papains.os.fyre.ibm.com:6443/apis/authentication.k8s.io/v1/tokenreviews"> 
      </oauth2Login>

      <keyStore id="defaultKeyStore" password="keyspass" />

      <!-- more application config would go here -->

    </server>
```

在 sidecar 场景中，配置更改为接受来自 sidecar 的入站令牌。要设置您的服务器使用 OAuth 代理边车:

```
      <!-- specify your userApiToken as a liberty variable or environment variable -->
      <!-- note that no clientId or clientSecret are needed --> 
    <oauth2Login id="openshiftLogin" scope="user:full" userNameAttribute="username" groupNameAttribute="groups" userApiToken="${serviceAccountToken}" userApiType="kube" accessTokenHeaderName="X-Forwarded-Access-Token" accessTokenRequired="true" userApi="https://kubernetes.default.svc/apis/authentication.k8s.io/v1/tokenreviews"> 
    </oauth2Login>
```

使用 HTTPS 通信需要两件事情之一。你可以给你的服务器一个由知名认证机构签名的密钥，Liberty 可以自动信任它，或者你可以将服务器的公钥添加到 Liberty 信任存储中。OpenShift 默认不附带 CA 签名的密钥，所以需要添加 Red Hat OpenShift OAuth 服务器的公钥。最方便的方法是在`server.env`中指定一个环境变量，如下所示:

```
    # server.env

    # OAuth sidecar scenario: causes the Kubernetes default certificate that is pre-installed in pods to be added to Liberty trust store.
      cert_defaultKeyStore=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

    # OAuth server scenario: causes the public keys from /tmp/trustedcert.pem (obtained seperetly) to be added to Liberty trust store.
      cert_defaultKeyStore=/tmp/trustedcert.pem
```

## 监控进程 CPU 时间(微文件度量 2.0)

一个新的指标，`processCpuTime,`,它返回运行 JVM 的进程所使用的 CPU 时间。MicroProfile Metrics 特性提供了监控应用程序的信息，例如使用的 CPU 时间、内存堆和 servlet 响应时间。

新的`processCpuTime`指标通过 Grafana 提供了云平台上更准确的 CPU 负载百分比。以前，CPU 负载百分比以指标`processCpuLoad`显示。但是，负载百分比是使用分配给部署的核心总数计算的。如果部署的内核数量有限，当达到最大内核数量时，`processCpuLoad`会在 Grafana 上显示一个平台。例如，在一个分配了 32 个内核但限制为 4 个内核的部署中，`processCpuLoad`图显示当所有 4 个内核都被使用时，平台为 12.5%。

新指标`processCpuTime,`可以在 Grafana 上操作，以创建更准确的 CPU 使用情况表示。PromQL 查询`rate(processCpuTime)[1m]`显示了一分钟内 CPU 时间的平均增长率。将这个结果除以 CPU 核心的总数，我们可以看到一个更准确的 CPU 使用百分比，其中考虑了约束条件。

新的`processCpuTime`指标显示在具有 MicroProfile Metrics 2.0 和 2.2 特性的`/metrics`端点上。在[仪表板](https://github.com/OpenLiberty/open-liberty-operator/tree/master/deploy/dashboards/metrics)上，可以使用以下 PromQL 查询创建一个新面板:

`(rate(base:cpu_process_cpu_time[1m])/1e9) / base:cpu_available_processors{app=~[[app]]}`

[查看完整的仪表板。](https://github.com/OpenLiberty/open-liberty-operator/tree/master/deploy/dashboards/metrics)

## 使用 Liberty 注释缓存更快地启动应用程序

由于向核心类和注释扫描功能添加了注释缓存，应用程序启动时间现在更快了。根据应用程序的特性，启动时间会减少 10%到 50%以上。有许多 jar 文件的应用程序，或者使用 CDI 或 JAX-RS 函数的应用程序，可以看到最好的改进，如图 1 所示。

[![Graph showing the startup time boosts broken out by context](img/82ea95bf5e09c0d94cc05e38d99703a1.png "20001annocache")](/sites/default/files/blog/2020/01/20001annocache.png)Figure 1: Annotation caching boosts startup times.">

**注意:**好消息！默认情况下，注记缓存处于启用状态。

注记缓存数据存储在服务器的工作区中。当执行干净的服务器启动时(使用`--clean`选项启动服务器)，应用程序类数据的缓存被清除。在正常操作中，不需要清除缓存数据，因为缓存会自动为更改的应用程序类重新生成缓存数据。

在容器环境中，为了使注释缓存有效，在创建容器映像时，服务器映像必须经过*预热*。可以通过在容器构建期间启动和停止服务器来预热服务器。预热图像会将注释扫描移动到容器构建中，这意味着您可以获得容器部署的最佳启动。在基本的`open-liberty` docker 映像中使用`configure.sh`文件会导致服务器在容器构建期间启动和停止。

## JavaServer Faces 2.3 中的错误修复

JavaServer Faces 2.3 包含一个加载 Apache MyFaces 2.3.6 中的错误修复的新特性。`jsf-2.3`特性提取 Apache MyFaces 实现，并将其集成到 Liberty 运行时中。

Apache MyFaces 2.3.6 版本包含错误修复。查看[发行说明了解更多信息。](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=10600&version=12346352)

要使用 JSF 2.3，启用`jsf-2.3`特性来利用最新的 Apache MyFaces 2.3.x 版本。有关 JavaServer 特性的更多信息，请查看 Apache 网站。

## 现在尝试在红帽运行时打开自由 20.0.0.1

Open Liberty 是 Red Hat Runtimes 产品的一部分。如果你是 Red Hat Runtimes 的用户，你现在可以尝试 Open Liberty。

要了解更多关于将 Open Liberty 应用部署到 OpenShift 的信息，请查看我们的 [Open Liberty 指南:将微服务部署到 OpenShift](https://openliberty.io/guides/cloud-openshift.html) 。

*Last updated: June 29, 2020*