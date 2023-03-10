# 开放自由 20.0.0.7 中灵活的单点登录认证等

> 原文：<https://developers.redhat.com/blog/2020/07/17/flexible-single-sign-on-authentication-and-more-in-open-liberty-20-0-0-7>

当使用信任关联拦截器(TAI)或简单且受保护的 GSS-API 协商机制(SPNEGO)身份验证时，开放自由 20.0.0.7 允许您禁用返回轻量级第三方身份验证(LTPA)cookie 进行身份验证的默认设置。使用 JWT 的单点登录(SSO)功能时，您也可以禁用 JSON Web Token(JWT)cookie。在本文中，我们将介绍这些改进以及新的[Open Liberty](https://openliberty.io/about)20.0.0.7 版本中的更多改进:

*   [如何使用 Open Liberty 20.0.0.7 运行您的应用程序](#run)
*   [如何禁用 TAI 或 SPNEGO 的 LTPA cookie](#LTPA-cookie)
*   [如何禁用 JWT 单点登录 cookie](#JWT-cookie)
*   [开放自由 20.0.0.7 修复了重大漏洞](#significant-bug-fixes)

## 如何使用 Open Liberty 20.0.0.7 运行您的应用

如果你正在使用 [Maven](https://openliberty.io/guides/maven-intro.html) ，这里是坐标:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.7</version>
    <type>zip</type>
</dependency>

```

对于 [Gradle](https://openliberty.io/guides/gradle-intro.html) ，您可以使用:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.7,)'
}

```

如果你正在使用一个`docker`容器，它是:

```
FROM open-liberty

```

另一个选择是从 [Open Liberty 下载页面](https://openliberty.io/downloads/)下载 ZIP 文件形式的内核包。然后你可以使用 [`featureUtility`](https://openliberty.io/blog/2020/06/05/graphql-open-liberty-20006.html#MVN) 命令将你需要的特性添加到内核中。

## 如何禁用 TAI 或 SPNEGO 的 LTPA cookie

LTPA cookie 包含带有用户身份和过期信息的加密身份验证令牌，您可以使用它进行单点登录(SSO)身份验证。在 Open Liberty 20.0.0.7，您可以决定在使用 TAI 或 SPNEGO 进行身份验证时是否要接收 LTPA cookie。

当客户机(如浏览器)通过 Open Liberty 服务器认证时，默认响应是在 HTTP servlet 中接收 SSO LTPA cookie。当客户端访问 Open Liberty 服务器中共享相同 LTPA 配置的另一个受保护资源时，使用 SSO LTPA cookie 的身份验证优先于任何其他身份验证机制。如果您使用另一种身份验证机制，使用 SSO LTPA cookie 优先进行身份验证可能会导致意想不到的结果。现在，您可以通过在使用 TAI 和 SPNEGO 身份验证时禁止创建 LTPA cookie 来解决这个问题。

下面是如何在`server.xml`中禁用 TAI 的 LTPA cookie:

```
<server>
  <featureManager>
    <feature>appSecurity-2.0</feature>
  </featureManager>
  <trustAssociation id="sample" disableLtpaCookie="true" />
</server>

```

以下是如何禁用 SPNEGO 的 LTPA cookie(也在`server.xml`中):

```
<server>
  <featureManager>
    <feature>spnego-1.0</feature>
  </featureManager>
  <spnego id="sample" disableLtpaCookie="true" />
</server>

```

## 如何禁用 JWT SSO cookie

当一个客户端(比如一个浏览器)通过 JSON Web Tokens SSO 特性(`jwtSso-1.0`)被 Open Liberty 服务器认证时，默认响应是 HTTP servlet 中的一个 JWT SSO cookie。当客户端访问同一个 Open Liberty 服务器或不同服务器中的另一个受保护资源时，使用 JWT cookie 的身份验证优先于任何其他身份验证机制。如果使用另一种身份验证机制，使用 JWT cookie 优先进行身份验证可能会导致意想不到的结果。现在，您可以通过在使用 JWT SSO 功能时禁用 JWT cookie 来解决此问题。

下面是如何在`server.xml`中禁用 JWT SSO 的 JWT cookie:

```
<server>
  <featureManager>
    <feature>jwtSso-1.0</feature>
  </featureManager>
  <jwtSso id="sample" disableJwtCookie="true" />
</server>

```

## 开放自由 20.0.0.7 中的错误修复

在这一节中，我们将描述我们在 20.0.0.7 版本中已经解决的几个问题。参见[修复开放自由 20.0.0.7](https://github.com/OpenLiberty/open-liberty/issues?q=label%3Arelease%3A20007+label%3A%22release+bug%22+)中的错误，查看完整的评论。

### 修正了 JAX RS 2.1 中的两个错误

**问题 8048: NullPointerException** :如果您在 JAX-RS 响应中写多部分表单数据时看到了一个`NullPointerException`，好消息是我们已经修复了它。详见问题 [8048](https://github.com/OpenLiberty/open-liberty/issues/8048) 。

**问题 12715: ContextResolver** :用户需要一种方法来通过用户的安全角色限制 JSON 字段序列化。他们能够通过使用`ContextResolver`指定 JSON-B 可见性策略并注入`SecurityContext`来使其工作。不幸的是，注射到`ContextResolver`里的药物不起作用。我们用问题 [12715](https://github.com/OpenLiberty/open-liberty/issues/12715) 解决了这个问题，这是一个非常酷的用例。

### 对 HTTP/2 的改进

**问题 12599: HTTP/2 连接**:一位用户报告了一个场景，当客户端没有正常终止 HTTP/2 连接时，会导致 CPU 消耗过多。我们已经解决了问题 [12599](https://github.com/OpenLiberty/open-liberty/issues/12599) 的 CPU 消耗。

**问题 12399: HTTP/2 流量控制**:在特定情况下，Open Liberty 更新其 HTTP/2 读取窗口的速度不够快，导致流量控制窗口停止。我们已经通过问题 [12399 改进了流量控制行为。](https://github.com/OpenLiberty/open-liberty/issues/12399)

### 解决微概要文件容错 2.1 中缺失的依赖关系

在此版本之前，在`maven`或`gradle`版本中添加对[微文件容错](https://developers.redhat.com/cheat-sheets/microprofile-fault-tolerance) 2.1 特性的依赖不会自动添加对微文件容错 2.1 API 的相应依赖。在某些情况下，当编写或编译使用 API 的应用程序时，用户会收到错误消息:

```
The import org.eclipse.microprofile.faulttolerance cannot be resolved
```

我们已经解决了问题 [12567](https://github.com/OpenLiberty/open-liberty/issues/12567) 中缺失的依赖项。

## 尝试在红帽运行时打开自由 20.0.0.7

开放自由是红帽运行时的一部分。如果你是 Red Hat Runtimes 的订户，你现在就可以尝试这个新的 Open Liberty 版本。

要了解更多关于将 Open Liberty 应用部署到 [Red Hat OpenShift](https://developers.redhat.com/openshift) 的信息，请参阅我们的 [*Open Liberty 指南:将微服务部署到 OpenShift*](https://openliberty.io/guides/cloud-openshift.html) 。

*Last updated: July 15, 2020*