# Eclipse XML 编辑器中的 Apache Camel URI 完成

> 原文：<https://developers.redhat.com/blog/2018/01/31/apache-camel-uri-completion-eclipse-xml-editor>

[Apache Camel](http://camel.apache.org/) 使您能够用各种特定于领域的语言定义路由和中介规则，包括基于 Java 的 [Fluent API](http://camel.apache.org/dsl.html) 、 [Spring](http://camel.apache.org/spring.html) 或 [Blueprint](http://camel.apache.org/using-osgi-blueprint-with-camel.html) [XML 配置](http://camel.apache.org/xml-configuration.html)文件，以及 [Scala DSL](http://camel.apache.org/scala-dsl.html) 。Apache Camel 使用 [URIs](http://camel.apache.org/uris.html) 直接与任何种类的[传输](http://camel.apache.org/transport.html)或消息模型一起工作，例如 [HTTP](http://camel.apache.org/http.html) 、 [ActiveMQ](http://camel.apache.org/activemq.html) 、 [JMS](http://camel.apache.org/jms.html) 、 [JBI](http://camel.apache.org/jbi.html) 、SCA、 [MINA](http://camel.apache.org/mina.html) 或 [CXF](http://camel.apache.org/cxf.html) ，以及可插入的[组件](http://camel.apache.org/components.html)和[数据格式](http://camel.apache.org/data-format.html)Apache Camel 是一个小型库，具有最小的[依赖性](http://camel.apache.org/what-are-the-dependencies.html)，可以轻松嵌入到任何 Java 应用程序中。

## 在 Eclipse XML 编辑器中完成 Apache Camel URI

感谢[这个旨在提供 Apache Camel 语言服务器协议的新社区项目](https://github.com/lhein/camel-language-server)。Eclipse XML Editor 提议完成 Apache Camel URIs。

![](img/aad8681b0d9fd6b0b38fca7126c80289.png)

它也兼容[熔丝工具](https://tools.jboss.org/features/fusetools.html)的 Camel Route 编辑器的 source 选项卡。

![](img/e852ff4a190813d13bb013fbe42b12c0.png)

查看这个页面关于如何安装插件——不完全是琐碎的，因为它仍然是一个非常早期的项目——并有利于在您的 IDE 中完成骆驼 URI！

## 下一步是什么？

### 主要 ide 中的可用性

技术选择是使用[语言服务器协议](https://github.com/Microsoft/language-server-protocol)。这意味着 Apache Camel 语言服务器协议实现的改进将使所有受支持的客户端受益。一些 ide 和编辑器可以从中受益。这里可以看到潜力榜[。关于](https://microsoft.github.io/language-server-protocol/implementors/tools/) [VS 代码、](https://code.visualstudio.com/)的工作已经开始，我们期望它也能用于 [Eclipse Che](https://www.eclipse.org/che/) 和 [IntelliJ IDEA](https://www.jetbrains.com/idea/) 。欢迎更多的客户对任何其他 IDEs 的贡献。

### 将工作与 Camel IDEA 插件合并

IntelliJ IDEA 有一个现有的 Camel 支持插件。不幸的是，IntelliJ IDEA 是少数默认情况下不支持语言服务器协议的主流 ide 之一。我希望有一天他们会支持它。目前可以用一个[第三方插件](https://plugins.jetbrains.com/plugin/10209-lsp-support)。我开始调查合并两个 Camel 项目的开发工作。非常欢迎帮助。

*Last updated: July 27, 2022*