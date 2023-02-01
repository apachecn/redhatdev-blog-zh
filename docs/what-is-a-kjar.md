# 什么是 KJAR？

> 原文：<https://developers.redhat.com/blog/2018/03/14/what-is-a-kjar>

从版本 6 开始，Red Hat JBoss BPM Suite 和 [Red Hat Decision Manager](https://developers.redhat.com/products/red-hat-decision-manager/overview/) (以前的 Red Hat JBoss BRMS)都使用一个称为“KJAR”的工件包，或知识工件。这是什么文件类型？它与标准 JAR 文件的区别是什么？

# 基本概要

简而言之，一个 [KJAR](https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/administration_and_configuration_guide/chap_business_central_configuration#sect_deployment_descriptors) 是一个标准的 JAR 文件，其中包含了一些额外的文件。KJAR 保持与 JAR 文件相同的`.jar`扩展名，因为它的基本文件结构与 JAR 相同。

# 为什么不同？

JAR(“Java archive”)文件意味着运行任何种类的 Java 代码，几乎无限的资源和资产都可能作为包的一部分。引用 JAR 标准，“JAR 文件本质上是一个包含可选 META-INF 目录的 zip 文件。”相比之下，KJAR(“知识工件”)专门针对那些倾向于用 XML 或纯文本表示的规则和流程。知识工件中可能包含也可能不包含任何其他资源或资产。KJAR 的内容仍然要编译成 Java 字节码，但是对规则和过程的特殊关注允许它们配置和优化内容，同时仍然符合标准的 JAR 文件结构。

# 有哪些区别？

一个 KJAR 必须至少包含文件`META-INF/kmodule.xml`。这个文件用于描述 KJAR 结构并定义一些 KIE 特有的工件。这个文件的 v6 规范可以在[这里](https://github.com/kiegroup/droolsjbpm-knowledge/blob/6.5.x/kie-api/src/main/resources/org/kie/api/kmodule.xsd)找到，v6 的例子可以在[这里](https://github.com/kiegroup/drools/blob/6.5.x/drools-compiler/src/test/resources/META-INF/kmodule.xml)找到。

以下项目在广口瓶和 KJARs 之间也有所不同:

*   **基于 Maven 的**
    *   **JAR:** 不需要根据任何特定的文件夹结构来构建，只要在顶层有一个`META-INF`目录
    *   **KJAR:** 符合 [Maven 标准目录布局](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)
*   **额外文件**
    *   **JAR:** 不需要存在特定的文件
    *   **KJAR:** 必须包含一个`META-INF/kmodule.xml`文件，该文件又必须包含至少一个格式正确的`<kmodule>` XML 标签
*   **预编译资产缓存**
    *   JAR: 任何要预编译成字节码的代码都是生成 JAR 的构建服务的责任。
    *   **KJAR:** 在构建期间使用 [KIE Maven 插件](https://mvnrepository.com/artifact/org.kie/kie-maven-plugin)，它会自动预编译一些规则，并将资产处理到缓存中(`META-INF/<kie-base-name>/kbase.cache`)。(此缓存并不完整，但可以提高加载规则/流程时的性能。)如果没有创建缓存，所有资产将在运行时构建。
*   **部署描述符(从 6.1 版开始)**
    *   JAR: 没有指定具体的配置或部署类型，尽管各种其他框架或标准(如 Java EE)可能会创建它们自己的标准。
    *   **KJAR:** 如果文件`META-INF/kie-deployment-descriptor.xml`存在，它将自动用于确定执行规则和/或流程的各种属性，例如运行时策略、事件监听器、工作项处理程序等等。 [²](#fn2)

# 什么保持不变？

几乎所有关于 KJAR 的东西都是针对 JAR 文件的，所以你可以相信 KJAR 和 JAR 文件几乎完全一样。你仍然可以使用一个`MANIFEST.MF`文件来定义包信息，你仍然可以使用一个`beans.xml`文件来让 [CDI](https://github.com/cdi-spec/cdi-spec.org/blob/master/_faq/intro/4-what-is-beans-xml-and-why-do-i-need-it.asciidoc) 选择类，你仍然可以使用像`logback.xml`或`log4j.xml`这样的流行文件来与流行的 Java 日志框架一起使用。

# KJAR 是如何制作的？

构建 KJAR 和构建标准 JAR 的主要区别在于，KJAR 在其`pom.xml`中有一个`<packaging>kjar</packaging>`条目，并且在同一个文件中还包含了`kie-maven-plugin`所必需的`<plugin>`。 [³](#fn3)

# 如果我生成一个罐子而不是一个 KJAR，会发生什么？

官方的说法是，使用 KIE Maven 插件可以确保工件资源得到验证和预编译，所以建议一直使用该插件。然而，如果 KJAR 中的规则/过程是有效的，那么不管它们是从 JAR 还是从 KJAR 运行，都不可能有任何执行问题。没有 KIE Maven 插件 [⁴](#fn4) 就不会创建`kbase.cache`，所以当用户试图从 JAR 而不是 KJAR 运行规则/流程时，可能会遇到更差的性能。

# 脚注:

[1) JAR 文件规范-https://docs . Oracle . com/javase/7/docs/technotes/guides/JAR/JAR . html](https://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html)

[2)部署描述符-https://access . red hat . com/documentation/en-us/red _ hat _ JBoss _ BPM _ suite/6.4/html/administration _ and _ configuration _ guide/chap _ business _ central _ configuration # IDM 139940729511264](https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/administration_and_configuration_guide/chap_business_central_configuration#idm139940729511264)

如何从 Maven 命令为 BRMS/BPM Suite 6 构建一个 kjar？——https://access.redhat.com/solutions/892893

[4)Drools 6 中是否可以直接加载 kbase.cache 文件来节省加载 KieBase 的时间？——https://access.redhat.com/solutions/1297523](https://access.redhat.com/solutions/1297523)

*Last updated: January 13, 2022*