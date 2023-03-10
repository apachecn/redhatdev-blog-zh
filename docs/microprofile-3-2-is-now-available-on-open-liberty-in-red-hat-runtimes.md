# MicroProfile 3.2 现在可以在 Red Hat 运行时的 Open Liberty 上获得

> 原文：<https://developers.redhat.com/blog/2019/12/11/microprofile-3-2-is-now-available-on-open-liberty-in-red-hat-runtimes>

[Open Liberty](https://openliberty.io/about/)19.0.0.12 提供对 [MicroProfile 3.2](https://microprofile.io/) 的支持，允许用户提供自己的健康检查程序，并使用指标轻松监控微服务应用。此外，更新允许通过环境变量使用 JDK 的默认信任库或证书来建立信任。

在开放自由 19.0.0.12 中:

*   [MicroProfile 3.2 支持](#mp32)
*   [提供您自己的健康检查程序(MicroProfile 健康检查 2.1)](#hc21)
*   [利用指标轻松监控微服务应用(MicroProfile Metrics 2.2)](#hm22)
*   [Jaeger 支持在 MicroProfile 中打开追踪](#jmo)
*   [可信证书增强功能(传输安全 1.0)](#ssl)
*   [自由读者角色支持](#rrs)

## 使用 19.0.0.12 运行您的应用

如果你用的是 [Maven](https://openliberty.io/guides/maven-intro.html) ，这里是坐标:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>19.0.0.12</version>
    <type>zip</type>
</dependency>
```

或者对于 [Gradle](https://openliberty.io/guides/gradle-intro.html) :

```
dependencies {
libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[19.0.0.12,)'
}
```

或者，如果您正在使用 [Docker](https://openliberty.io/guides/containerize.html) :

```
FROM open-liberty
```

## MicroProfile 3.2 支持

使用`server.xml`中的这个便利特性，将整个 MicroProfile 3.2 添加到您的应用程序中:

```
<featureManager>
  <feature>microProfile-3.2</feature>
<featureManager>
```

`microProfile-3.2`特性在您的应用程序中自动包含以下特性:JAX-RS 2.1、CDI 2.0、JSON-P 1.1、JSON-B 1.0、微文件配置 1.3、微文件容错 2.0、微文件健康检查 2.1、微文件 JWT 1.1、微文件度量 2.2、微文件 OpenAPI 1.1、微文件 OpenTracing 1.3 和微文件 Rest 客户端 1.3。

微配置文件健康检查和度量功能包含更新。

### 提供您自己的健康检查程序(MicroProfile 健康检查 2.1)

MicroProfile Health Check 2.1 使您能够提供由 Liberty 调用的您自己的健康检查过程，以验证您的微服务的健康状况:

```
HealthCheckResponse.up("myCheck");
```

在以前的版本中，为了定义一个简单的成功/失败的命名健康检查响应，应用程序级代码总是需要使用 HealthCheckResponse API 中的几个静态方法来检索用于构造健康检查响应的 HealthCheckResponseBuilder。

在 OpenLiberty 的`mpHealth-2.1`特性中，您现在可以使用来自标准 Java APIs 的方便而简单的方法，在您的应用程序中构造 UP/DOWN 命名的健康检查响应，例如`HealthCheckResponse.named(“myCheck”).up().build();`

要使其工作，请在`server.xml`文件中包含以下行:

```
<feature>mpHealth-2.1</feature>
```

应用程序应该通过使用`@Liveness`或`@Readiness`注释实现 health check 接口来提供健康检查过程，Liberty 分别使用这些注释来验证应用程序的活性或就绪性。在`call()`方法中添加健康检查的逻辑，并通过从 API 使用简单的`up()`和`down()`方法构造返回 HealthCheckResponse 对象。要查看每个健康检查的状态，请访问`+http://:/health/live+`或`+http://:/health/ready+`端点。

```
**Liveness Check**
@Liveness
@ApplicationScoped
public class AppLiveCheck implements HealthCheck {
...
    @Override
     public HealthCheckResponse call() {
       ...
       HealthCheckResponse.up("myCheck");
       ...
     }
}
```

有关更多信息:

*   [MicroProfile Health Check 2.1 发布页面](https://github.com/eclipse/microprofile-health/releases/tag/2.1)
*   [Javadocs](http://download.eclipse.org/microprofile/microprofile-health-2.1/apidocs/)
*   [规范文件](https://download.eclipse.org/microprofile/microprofile-health-2.1/microprofile-health-spec.html)

### 利用指标轻松监控微服务应用(MicroProfile Metrics 2.2)

MicroProfile Metrics 2.2 使开发人员能够在他们的(微服务)应用程序中使用指标，以便于他们的运营团队进行监控。

之前，MetadataBuilder API 有`reusable()`和`notReusable()`方法将可重用字段设置为`true`或`false`。MetadataBuilder API 已经过更改，为可重用属性包含了一个新的 setter 方法。实现这一更改是为了让 MetadataBuilder API 遵循 Builder 模式。

要启用`server.xml`文件中的功能:

```
<feature>mpMetrics-2.2</feature>
```

该示例显示了如何使用 MetadataBuilder API 设置可重用字段:

```
MetadataBuilder mdb = Metadata.builder();
```

```
mdb = mdb.withName("metricName").withType(MetricType.COUNTER)
.reusable(resolveIsReusable());
```

有关更多信息:

*   微配置文件指标的更改
*   微服务可观察性度量

## 添加了 Jaeger 对跟踪的支持(MicroProfile OpenTracing 1.3)

Open Liberty 在 MicroProfile OpenTracing 中增加了对 Jaeger 的支持。使用 Zipkin 作为跟踪后端，有一个[样本跟踪器可用](https://github.com/WASdev/sample.opentracing.zipkintracer)。随着 Jaeger 支持的加入，开发人员也可以使用 Jaeger 作为跟踪后端。

可以从 [Maven 资源库](https://mvnrepository.com/artifact/io.jaegertracing/jaeger-client/0.34.0)下载 Jaeger 客户端 0.34.0 版库及其依赖项。

在 server.xml 中:

```
<library id="jaegerLib" apiTypeVisibility="+third-party" >
        <file name="<path>/jaegerLib_0.34/gson-2.8.2.jar" />
        <file name="<path>/jaegerLib_0.34/jaeger-client-0.34.0.jar" />
        <file name="<path>/jaegerLib_0.34/jaeger-core-0.34.0.jar" />
        <file name="<path>/jaegerLib_0.34/jaeger-thrift-0.34.0.jar" />
        <file name="<path>/jaegerLib_0.34/jaeger-tracerresolver-0.34.0.jar" />
        <file name="<path>/jaegerLib_0.34/libthrift-0.12.0.jar" />
        <file name="<path>/jaegerLib_0.34/slf4j-api-1.7.25.jar" />
        <file name="<path>/jaegerLib_0.34/slf4j-jdk14-1.7.25.jar" />
        <file name="<path>/jaegerLib_0.34/opentracing-util-0.31.0.jar" />
        <file name="<path>/jaegerLib_0.34/opentracing-noop-0.31.0.jar" />
    </library>
```

定义您的应用:

```
<webApplication location="yourapp.war" contextRoot="/yourapp">
        <!-- enable visibility to third party apis -->
        <classloader commonLibraryRef="jaegerLib" apiTypeVisibility="+third-party" />
  </webApplication>
```

你可以通过查看 [jaeger-client-java readme](https://github.com/jaegertracing/jaeger-client-java/blob/10c641f8df6316f1eac4d5b1715513275bcd724e/jaeger-core/README.md) 找到更多关于使用环境变量设置 Jaeger 设置的信息。

对于`JAEGER_PASSWORD`环境变量，可以使用 securityUtility 命令对密码进行编码。根据 Jaeger 的采样设置`JAEGER_SAMPLER_TYPE`和`JAEGER_SAMPLER_PARAM`，Jaeger 可能不会报告应用程序创建的每个跨度。

## 可信证书增强(传输安全 1.0)

Open Liberty 现在提供了新的选项来帮助建立 TLS 连接的信任。现在提供了一种使用 JDK 的默认信任库进行信任的简单方法，以及一种通过环境变量将建立信任所需的证书传递给信任库的方法。

### 使用 JDK 的默认信任库建立信任

默认情况下，JDK 默认信任库是`cacerts`文件。默认的信任库可以由`javax.net.ssl.truststore`系统属性或`jssecacerts`文件设置，如果用户配置了一个的话。为了让 Open Liberty 使用配置为 JDK 默认信任库的内容，需要在`ssl`元素上将`useDefaultCerts`属性设置为`true`。默认设置为`false`。例如:

```
<ssl id="defaultSSLConfig" keyStoreRef="defaultKeyStore" trustStoreRef="defaultTrustStore" trustDefaultCerts="true" />
<keyStore id="defaultKeyStore" location="key.p12" type="PKCS12" password=<your password>  />
<keyStore id="defaultTrustStore" location="trust.p12" type="PKCS12" password=<your password> />
```

当`trustDefaultCerts`设置为`true`时，服务器将尝试与已配置的信任库建立信任，在本例中首先是`defaultTrustStore`。如果没有通过配置的信任库建立信任，那么它将尝试使用 JDK 的默认信任库来建立信任。

### 通过环境变量提供证书以建立信任

Open Liberty 将从环境变量中读取一个证书，并将其添加到密钥库或信任库中，以便可以用于信任。证书将被添加到密钥库或信任库的运行时副本中，而不会存储到文件系统中。如果密钥库配置包括设置为`true`的`readOnly`属性，则不会包括该证书。

环境变量键必须采用格式`cert_ + keystore id`。例如:

```
<keyStore id="myKeyStore" location="myKey.p12" type="PKCS12" password=<your password> />
```

环境变量的键应该是`cert_myKeyStore`(区分大小写)。

环境变量的值可以是基本 64 位格式的证书，也可以是包含基本 64 位编码证书或 DER 编码证书的文件的路径。如果直接在环境变量上使用基本 64 位编码证书，它必须包含`-----BEGIN CERTIFICATE-----`和`-----END CERTIFICATE-----`标记。例如:

```
cert_myKeyStore="-----BEGIN CERTIFICATE-----
....
-----END CERTIFICATE-----"
```

文件的环境变量类似于:

`cert_myKeyStore=/Users/me/servercert.crt`

任何不以`-----BEGIN CERTIFICATE-----`标签开头的值都将被视为文件。

[#rrs]

## 自由读者角色支持(应用安全 2.0 和应用安全 3.0)

reader 角色是一个管理角色，允许对选择的管理 REST APIs 以及管理中心 UI ( `adminCenter-1.0`)进行只读访问。

在此版本之前，管理员管理角色是 Open Liberty 中唯一的管理角色，它提供读写访问权限。新的读者管理角色提供了向用户和组分配只读角色的能力。这将允许这些用户和组监视服务器，而不授予这些用户以任何方式修改服务器的能力。

使用新的读者管理角色与使用管理员管理角色几乎相同。在`server.xml`中，包括`appSecurity-2.0`或`appSecurity-3.0`特性，并添加新的`reader-role`配置元素，该元素指定应被授予读者管理角色的组、用户和/或组或用户的访问 ID。

```
<server>
    <featureManager>
        <feature>appSecurity-3.0</feature>
    </featureManager>

    <reader-role>
        <group>group</group>
        <group-access-id>group:realmName/groupUniqueId</group-access-id>
        <user>user</user>
        <user-access-id>user:realmName/userUniqueId</user-access-id>
    </reader-role>
</server>
```

## 现在尝试在红帽运行时打开自由 19.0.0.12

Open Liberty 是 Red Hat Runtimes 产品的一部分。如果你是 Red Hat Runtimes 的用户，你现在可以尝试 Open Liberty。

要了解更多关于将 Open Liberty 应用部署到 OpenShift 的信息，请查看我们的 [Open Liberty 指南:将微服务部署到 OpenShift](https://openliberty.io/guides/cloud-openshift.html) 。

*Last updated: July 1, 2020*