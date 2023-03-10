# 使用 Open Liberty 20.0.0.10 的自动证书管理环境支持 2.0 安装签名证书

> 原文：<https://developers.redhat.com/blog/2020/10/07/install-a-signed-certificate-with-open-liberty-20-0-0-10s-automatic-certificate-management-environment-support-2-0>

Red Hat Runtimes 现在支持新的[Open Liberty 20.0.0.10](https://github.com/OpenLiberty/open-liberty/releases)Java 运行时。开放自由 20.0.0.10 支持[自动证书管理环境](https://tools.ietf.org/html/rfc8555) (ACME)协议，该协议自动获取由认证机构(CA)签署的证书。开放自由 20.0.0.10 版本也包括许多错误修复。

Open Liberty 是一个快速、轻量级的 [Java](https://developers.redhat.com/topics/enterprise-java) 运行时，用于构建云原生应用和[微服务](https://developers.redhat.com/topics/microservices/)。它与 MicroProfile 和 Jakarta EE 兼容，使您能够根据需要包含尽可能多或尽可能少的自由来支持您的应用程序。每四周发布一次，零迁移，更容易保持最新，避免技术债务。

在本文中，我将介绍新的 ACME CA Support 2.0 ( `acmeCA-2.0`)特性，包括如何使用它来安装 CA 签名的证书。访问 Open Liberty 的 GitHub 库，查看本次发布的[修复错误列表](https://github.com/OpenLiberty/open-liberty/issues?q=label%3Arelease%3A200010+label%3A%22release+bug%22+)。

## 使用 Open Liberty 20.0.0.10 运行您的应用

使用以下坐标安装带 [Maven](https://openliberty.io//guides/maven-intro.html) 的开放自由 20.0.0.10:

```
<dependency>
    <groupId>io.openliberty</groupId>
    <artifactId>openliberty-runtime</artifactId>
    <version>20.0.0.10</version>
    <type>zip</type>
</dependency>

```

对于[梯度](https://openliberty.io//guides/gradle-intro.html)，使用:

```
dependencies {
    libertyRuntime group: 'io.openliberty', name: 'openliberty-runtime', version: '[20.0.0.10,)'
}

```

如果您使用 docker，它是:

```
FROM open-liberty
```

## 使用 ACME CA 2.0 安装签名证书

默认情况下，Open Liberty 为传输安全(SSL/TLS)支持提供自签名证书。自签名证书让您可以立即建立传输安全性，但是大多数浏览器会将该证书标记为不安全。因此，访问您网站的用户将会收到警告或错误消息。

拥有一个 CA 签名的证书可以解决这个问题，但是获得一个证书的成本很高。在某些情况下，签名证书在开发和测试期间可能不可用。由公共认证机构签署的证书，比如[让我们加密](https://letsencrypt.org/)，提供了一个中间地带。

通过开放自由 20.0.0.10，我们增加了对 ACME 协议的支持，这使得认证机构和您的 web 服务器之间的交互自动化。您可以使用新的 ACME CA 2.0 特性来安装 CA 签名的证书，以改进测试和用户体验。

## 在 server.xml 中添加 acmeCA-2.0 特性

在您的`server.xml`中，只需提供使用 ACME 2.0 协议的认证机构的目录 URI，以及您的 Open Liberty 服务器的域名。ACME 提供者在端口 80 上回叫以验证域所有权。一旦验证了所有权，CA 就会颁发证书。在启动时，Open Liberty 服务器使用提供的 CA 目录 URI 来请求证书。CA 签名的证书安装在密钥库中，充当默认证书。

要在您的 Open Liberty 20.0.0.10 安装中包含 ACME CA 2.0 功能，请按如下方式更新您的`server.xml`:

```
<featureManager>
    <feature>acmeCA-2.0</feature>
</featureManager>

<acmeCA directoryURI="https://acme.host.com/directory" >
    <domain>theDomainThatIOwn.com</domain>
    <accountContact>mailto:my_email_addr@theDomainThatIOwn.com</accountContact>
</acmeCA>

<httpEndpoint host="*" httpPort="80" httpsPort="443" id="defaultHttpEndpoint"/>
<keyStore password="password_for_keystore" id="defaultKeyStore"/>

```

请参见 [ACME 规范(RFC8555)](https://tools.ietf.org/html/rfc8555) 和 [ACME 维基百科页面](https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment)了解 ACME 协议的高级视图。关于 ACME 协议及其工作原理的更多信息，请参见[的“让我们加密”主页](https://letsencrypt.org/how-it-works/)。

## 尝试在红帽运行时打开自由 20.0.0.10

Open Liberty 是 Red Hat Runtimes 的一部分，可供 [Red Hat Runtimes 的订户](https://access.redhat.com/products/red-hat-runtimes)使用。要了解更多关于将 Open Liberty 应用部署到 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 的信息，请参见我们的 Open Liberty 指南，*将微服务部署到 Open shift。开放自由 20.0.0.10 可以通过 Maven、Gradle、docker 获得，也可以下载存档。*

*Last updated: October 6, 2020*