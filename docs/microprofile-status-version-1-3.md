# 版本 1.3 中的微配置文件状态

> 原文：<https://developers.redhat.com/blog/2018/05/07/microprofile-status-version-1-3>

将近两年前启动的 Eclipse MicroProfile 项目进展迅速，发布了四个版本和八个子版本，每个版本至少有两个实现。因为它是一个快速移动的目标，这篇文章试图给出一个 9 月 30 日发布的 MicroProfile 1.3 的概述，并帮助你开始使用这个规范。

## 什么是 MicroProfile？

在你开始阅读这篇文章之前，你可能已经知道什么是 MicroProfile 了。但是为了让事情更清楚，我将引用下面的 microprofile.io 网站定义:

“微文件是一个基准平台定义，它为微服务架构优化企业 Java，并提供跨多个微文件运行时的应用程序可移植性。最初计划的基线是 JAX-RS CDI JSON-P，目的是让社区在微文件定义和路线图中发挥积极作用

换句话说，MicroProfile 是一个利用 JAX-RS、CDI 和 JSON-P(以及未来可能的其他规范)来轻松生产微服务的平台。

在这三个众所周知的 Java EE(或 Jakarta EE)规范之上，MicroProfile 添加了自己的层，为微服务(但也为任何 Java EE 应用程序)提供增值功能。

MicroProfile 是一个 Eclipse Foundation 项目，使它完全与供应商无关。

## 支持 MicroProfile 的供应商和项目

MicroProfile 有着巨大的发展势头，并被供应商广泛采用。实现它的主要产品和项目有:

*   被红帽子包围的野花
*   IBM 的 Open Liberty
*   阿帕契·汤姆
*   帕亚拉微型公司
*   Apache Hammock
*   库穆卢泽

该列表并不详尽，并且随着每个微文件版本的增加而增加。

## 包括什么？

在 1.3 版本中，微文件规范包括以下内容:

*   配置 1.2(从上一版本更新)
*   容错 1.0
*   运行状况检查 1.0
*   指标 1.1(从上一版本更新)
*   JWT 认证 1.0
*   微文件 OpenAPI 1.0(新规范)
*   MicroProfile OpenTracing 1.0(新规范)
*   MicroProfile Rest 客户端 1.0(新规范)

让我们快速浏览一下每一项。

### 伞形规格

微文件伞状规范为微文件定义了 JDK、CDI、JAX-RS 和 JSON-P 的最低版本。现在，这些版本在 Java EE 7 上是一致的，但是如果实现者愿意，他们可以选择使用更高的版本。

该规范可能会被错误地认为是 Maven POM 文件中的物料清单。事实上，还不止于此。社区成员就三个要点达成了一致，这使得该规范更加出色:

1.  doc 工具的一致性:虽然规范没有强制 subspecs 这样做，但是每个 MicroProfile 贡献者都同意使用 Asciidoc 格式和 Asciidoctor 工具链来生成文档。
2.  TCKs 中工具的一致性:技术兼容性工具包是规范中非常重要的一部分；它帮助实现者根据测试集合来验证他们的代码，设计一个“活的”契约来遵守规范。正如对文档所做的那样，达成了一项协议，使用 TestNG 和 Arquillian 为所有的微文件规范生成 tck。
3.  以 CDI 为中心的方法:同样，这不是一个书面规则，而是一个隐含的指导方针，以确保 CDI 编程模型存在于所有微文件子空间中，从而为微文件用户提供一致的用户体验。

第 1 点和第 2 点对于使贡献者和实现者的生活更容易是很重要的。作为一名贡献者，当从一个规范切换到另一个规范时，我不必学习另一种编写文档和测试的方法。这种一致性是授权来自社区的贡献的一种方式。

第三点是让采用变得更容易的一种方式。使用一致的方式来消费特性和开发使得学习曲线变得更短，并且允许团队专注于他们的业务部分，而不是技术部分。此外，作为 CDI 规范负责人，我只能批准对 MicroProfile 采用 CDI 编程模型。

微轮廓伞规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-bom/releases/download/1.3/microprofile-spec-1.3.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-bom) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile</groupId>
    <artifactId>microprofile</artifactId>
    <version>1.3</version>
    <type>pom</type>
    <scope>provided</scope>
</dependency>
```

### 微配置文件配置规范 1.2

所有其他规范都使用微配置文件配置来提供从各种来源读取配置信息的一致方式。虽然它提供了 CDI 支持，但该规范允许在不使用 CDI 的情况下访问配置。它的灵感来自于 Apache DeltaSpike 和 Apache Tamaya 等项目。

微配置文件配置规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-config/releases/download/1.2/microprofile-config-spec-1.2.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-config) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.config</groupId>
    <artifactId>microprofile-config-api</artifactId>
    <version>1.2</version>
</dependency>
```

### 微文件容错 1.0

该规范允许开发人员通过断路器和隔板模式、异步操作以及重试或回退策略为微服务添加容错机制。1.0 版通过提供拦截器绑定来将这些策略应用到所选的方法或类，从而完全面向 CDI。

微轮廓容错规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-fault-tolerance/releases/download/1.0/microprofile-fault-tolerance-spec-1.0.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-fault-tolerance) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.fault-tolerance</groupId>
    <artifactId>microprofile-fault-tolerance-api</artifactId>
    <version>1.0</version>
</dependency>
```

### 微配置文件运行状况检查 1.0

该规范提供了一种从另一台机器探测计算节点状态的方法。该规范不是为人类操作员设计的，而是为云基础设施中的自动化系统设计的，以决定一个给定的计算节点是否应该被丢弃或被另一个节点替换。

MicroProfile 健康检查规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-health/files/1312316/microprofile-health-spec-1.0.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-health) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.health</groupId>
    <artifactId>microprofile-health-api</artifactId>
    <version>1.0</version>
</dependency>
```

### 微轮廓度量 1.1

该规范为使用 MicroProfile 开发的微服务提供了一种统一的方式，将监控数据暴露给管理代理。与其他规范一样，它并不局限于 MicroProfile，也可以用于任何 Java 开发来公开遥测数据。

MicroProfile Metrics 规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-metrics/releases/download/1.1/metrics_spec-1-1.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-metrics) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.metrics</groupId>
    <artifactId>microprofile-metrics-api</artifactId>
    <version>1.1</version>
</dependency>
```

### 微配置文件 JWT 验证 1.0

这个微文件子空间为开发的微服务增加了安全性。通过使用 JSON Web 令牌(JWT)传播，它有助于使用 RESTful 方法跨不同的服务保存安全信息。

微配置文件 JWT 认证规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-jwt-auth/files/1305001/microprofile-jwt-auth-spec-1.0.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-jwt-auth) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.jwt</groupId>
    <artifactId>microprofile-jwt-auth-api</artifactId>
    <version>1.0</version>
</dependency>
```

### MicroProfile OpenAPI 1.0

这个 MicroProfile 规范旨在为 OpenAPI v3 规范提供一个统一的 Java API，所有应用程序开发人员都可以使用它来公开他们的 API 文档。

OpenAPI 规范(OAS)为 RESTful APIs 定义了一个标准的、与语言无关的接口，允许人类和计算机在不访问源代码或文档或通过网络流量检查的情况下发现和理解服务的功能。

MicroProfile OpenAPI 规范的链接:

*   规格文档的 [PDF](http://download.eclipse.org/microprofile/microprofile-open-api-1.0/microprofile-openapi-spec.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-open-api) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.openapi</groupId>
    <artifactId>microprofile-openapi-api</artifactId>
    <version>1.0</version>
</dependency>
```

### MicroProfile OpenTracing 1.0

该微文件规范定义了在 JAX 遥感应用程序中访问符合 OpenTracing 的跟踪对象的行为和 API。这些行为指定传入和传出请求如何自动创建 OpenTracing 范围。API 定义了如何显式访问已配置的跟踪对象。

MicroProfile OpenTracing 规范的链接:

*   规格文档的 [PDF](https://github.com/eclipse/microprofile-opentracing/releases/download/1.0/microprofile-opentracing.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-opentracing) 上的源代码

该规范的 Maven 工件:

```
<dependency>
    <groupId>org.eclipse.microprofile.opentracing</groupId>
    <artifactId>microprofile-opentracing-api</artifactId>
    <version>1.0</version>
</dependency>
```

### MicroProfile Rest 客户端 1.0

这个 MicroProfile 规范提供了一种类型安全的方法来通过 HTTP 调用 RESTful 服务。MicroProfile Rest 客户端规范尽可能地尝试使用 JAX-RS 2.0 API 来实现一致性和更容易的重用。

MicroProfile Rest 客户端规范的链接:

*   规格文档的 [PDF](http://download.eclipse.org/microprofile/microprofile-rest-client-1.0/microprofile-rest-client.pdf) 文件
*   [GitHub](https://github.com/eclipse/microprofile-rest-client) 上的源代码

该规范的 Maven 工件:

```
<dependency>
 <groupId>org.eclipse.microprofile.rest.client</groupId>
 <artifactId>microprofile-rest-client-api</artifactId>
 <version>1.0</version>
</dependency>
```

### 结论

如果你想了解更多关于 MicroProfile 的知识，请关注一下 [microprofile.io](http://microprofile.io) 网站或者 Eclipse 网站的 [MicroProfile 版块](https://projects.eclipse.org/projects/technology.microprofile)。你可能还想在推特上关注[@ micropofileio](https://twitter.com/microprofileio)。

如果您想为该规范或其中一个实现做出贡献， [microprofile.io](http://microprofile.io) 是查找信息和开始工作的最佳地方。订阅[微档案谷歌群](https://groups.google.com/forum/#!forum/microprofile)可能是你的第一步。

*Last updated: October 17, 2018*