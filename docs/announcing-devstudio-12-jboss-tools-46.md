# 宣布为 Eclipse Photon 发布 Red Hat Developer Studio 12.0.0.GA 和 JBoss Tools 4.6.0.Final

> 原文：<https://developers.redhat.com/blog/2018/07/18/announcing-devstudio-12-jboss-tools-46>

桌面 IDE 用户注意:[Red Hat Developer Studio 12.0](https://developers.redhat.com/products/devstudio/download/)和 Eclipse Photon 的社区版、 [JBoss Tools 4.6.0](http://tools.jboss.org/downloads/jbosstools/photon/4.6.0.Final.html) 现已推出。您可以下载一个捆绑的安装程序 Developer Studio，它会安装 Eclipse 4.8 以及所有已经配置好的 JBoss 工具。或者，如果您已经安装了 Eclipse 4.8 (Photon)，可以下载 JBoss 工具包。本文重点介绍了 JBoss 工具和 Eclipse Photon 中的一些新特性，涵盖了 WildFly、Spring Boot、Camel、Maven 和许多与 Java 相关的改进，包括对 Java 10 的全面支持。

Developer Studio / JBoss Tools 提供了一个桌面 IDE，其中包含了一系列涵盖多种编程模型和框架的工具。如果您正在进行容器/云开发，那么可以使用 Red Hat OpenShift、Kubernetes、Red Hat Container Development Kit 和 Red Hat OpenShift 应用程序运行时的集成功能。对于集成项目，有涵盖 Camel 和 Red Hat Fuse 的工具，可用于本地和云部署。

## 装置

### Red Hat Developer Studio -完全安装

下载[红帽开发者工作室安装程序](https://www.jboss.org/products/devstudio.html)。下载是一个单独的可执行 JAR 文件。你需要安装一个 JDK。然后，像这样运行它:

```
java -jar jboss-devstudio-<installername>.jar
```

### 将 JBoss 工具添加到现有的 Eclipse 4.8 环境中

JBoss Tools 也称为自带 Eclipse (BYOE)选项，它至少需要 Eclipse 4.8 (Photon)，但我们建议使用最新的 [Eclipse 4.8 Photon JEE 捆绑包](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/photonr)，因为从那时起，您就可以预装大多数依赖项。

一旦你安装了 Eclipse，你可以在 *JBoss Tools* 下的 *Eclipse Marketplace* 或者 *Red Hat Developer Studio* 中找到我们。

或者，对于 JBoss 工具，您也可以直接使用我们的更新站点:

```
http://download.jboss.org/jbosstools/photon/stable/updates/
```

## 什么是新的？

我们这次发布的主要焦点是 Java 10 的采用、基于容器的开发的改进和错误修复。Eclipse Photon 本身有很多新的很酷的东西，但是让我强调一下 Eclipse Photon 和 JBoss Tools 插件中我认为值得一提的几个更新。

### OpenShift 3

#### 服务器适配器的增强 Spring Boot 支持

OpenShift 服务器适配器已经支持 Spring Boot 运行时。然而，它有一个主要的限制:只有对于主项目，文件和资源才在本地工作站和远程 pod 之间同步。如果您的 Spring Boot 应用程序具有存在于本地工作区中的依赖项，则不会处理对这些依赖项之一的文件或资源的任何更改。幸运的是，这个新版本解决了这个问题！

### 服务器工具

#### Wildfly 13 服务器适配器

一个服务器适配器已被添加到 Wildfly 13 中。它增加了对 Servlet 4.0 的支持。

### 骆驼和保险丝工具

#### 骆驼休息 DSL 从 WSDL 向导

有一个新的*“骆驼休息 DSL 来自 WSDL”*向导。该向导包装了现在包含在 Fuse 7 发行版中的 [wsdl2rest 工具](https://github.com/jboss-fuse/wsdl2rest)，该工具为基于 SOAP(JAX-WS)的 web 服务获取一个 wsdl 文件，并生成 CXF 生成代码和 Camel rest DSL 路由的组合，使其可以使用 REST 操作进行访问。

首先，您需要在您的工作区中有一个现有的 Fuse 集成项目，并访问 SOAP 服务的 WSDL。然后使用*文件→新建→其他…* ，选择*红帽保险丝→WSDL*向导骆驼休息 DSL。

在向导的第一页，选择您的 WSDL 和 Fuse 集成项目，在其中生成 Java 代码和 Camel 配置。

![](img/6871bfdad4c237e2d74f6a37a294ec75.png)

在第二个页面上，您可以为生成的类定制 Java 文件夹路径，为生成的 Camel 文件定制文件夹，另外还可以定制 SOAP 服务地址和目的地 REST 服务地址。

![](img/4434c015c9161604982c2a3be7a3b9dd.png)

点击 *Finish* ，新的 Camel 配置和相关的 Java 代码就会在您的项目中生成。该向导确定您的项目是基于 Blueprint、Spring 还是 Spring Boot 的，并且它创建相应的工件，而不需要任何额外的输入。当向导完成时，您可以在 Fuse Tooling Route 编辑器中打开您的新 Camel 文件，以查看它创建了什么。

![](img/7f3aa81f48721c33a0c56cbb6e0f8d4a.png)

这为我们带来了另一个新功能，Fuse Tooling Route 编辑器中的 REST 选项卡。

#### Camel 编辑器休息标签

Fuse Tooling Route Editor 提供了一个新的 *REST* 选项卡。对于此版本，此选项卡的内容是只读的，包括以下信息:

*   REST 配置元素的详细信息，包括组件(jetty、netty、servlet 等)。)，上下文路径，端口，绑定方式(JSON，XML 等。)，还有主持人。只有一个 REST 配置元素。
*   收集 REST 操作的 REST 元素列表。一个配置可以有多个 REST 元素。每个 REST 元素都有一个关联的属性页，该属性页显示附加的详细信息，比如路径和它所消费或产生的数据。

![](img/ca459827df5b83108844d0cf3e1f30ff.png)

*   所选 REST 元素的 REST 操作列表。每个操作都有一个关联的属性页，提供诸如 URI 和输出类型等详细信息。

![](img/5ae39543ac9d08d6482144b8042051dc.png)

对于此版本，其余选项卡是只读的。如果您想要编辑 REST DSL，请使用路由编辑器的 Source 选项卡。当您在 Source 选项卡中进行更改并保存它们时，REST 选项卡会刷新以显示您的更新。

#### 用 XML DSL 完成骆驼 URI

正如这里宣布的那样，通过在 IDE 中安装对 Apache Camel 的语言支持，已经可以在 Camel Route 编辑器的 source 选项卡中使用 XML DSL 完成 Camel URI。

这个特性现在默认安装了 Fuse 工具！

![](img/8ea58b066cf72f9d402971c49f1f8544.png)

### 专家

#### Maven 支持更新到 M2E 1.9.1

Maven 支持基于 Eclipse M2E 1.9.1，具有以下特性:

##### 高级类路径隔离

多亏了 Eclipse Photon，有了两个新的不同的类路径，主类路径和测试类路径。主类现在将不再看到测试类和依赖项

##### 嵌入式 Maven 运行时

嵌入式 Maven 运行时现在基于 Apache Maven 3.5.3。

##### 原型目录管理

现在可以禁用原型目录。

##### Java 9/10 支持

对 Java 9/10 的支持得到了改进:修复了错误，更好地处理了模块路径。

### java 开发工具 _jdt)

#### 支持 Java 10

##### 将项目合规性和 JRE 更改为 10 的快速修复功能

快速修复**改变项目符合性和 JRE 到 10** 被提供来快速改变当前项目以兼容 Java 10。

![quickfix change compliance 10](img/3fab21d0305defa58d5495151495e405.png)

#### Java 编辑器

##### 将@NonNullByDefault 添加到包中的快速修复功能

提供了一个新的快速修复方法，用于修复在启用包警告上缺少“@NonNullByDefault”批注时报告的问题。如果包已经有了一个`package-info.java`，可以从编辑器中调用快速修复:

![](img/8c1588157df0277d3a812124b9e78ed5.png)

否则，必须从 problems 视图调用快速修复，并将创建一个带有所需注释的`package-info.java`:

![](img/45cdba7dc53772c2a4e8656aae970103.png)

当从 problems 视图调用时，快速修复的两种变体可以同时修复多个包的问题。

##### 导航到“switch”语句

您现在可以在 case 或 default 关键字上使用 **Ctrl+click** 或 **Open Declaration (F3)** 来快速导航到 switch 语句的开头。

![](img/86235fa8952cc499664323840f365dfa.png)

##### 粘贴到字符串文字时转义非 ASCII 字符

**Java >编辑器>在粘贴到字符串文字时键入>转义文本**首选项现在有了一个子选项**对非 ASCII 字符使用 Unicode 转义语法**:

![](img/9f7350de3ee52fd90e8bf7cac2b9c9a3.png)

启用后，在粘贴到字符串中时，可见 ASCII 范围之外的字符将被 unicode 转义序列替换:

![](img/df6134c06d54a972bb77aff165d6c6b9.png)

##### 改进了黑暗主题中的 Java 语法着色

为了提高深色主题的可读性，粗体样式的使用已经减少，一些过于接近的颜色已经被修改。

![](img/032ecfc02076d7d344837b87d478aeae.png)

##### 改进了黑暗主题中代码元素信息的链接着色

代码元素信息控件中链接的颜色现在考虑了来自**颜色&字体**首选项页面的**超链接文本颜色**和**活动超链接文本颜色**的颜色设置。黑暗主题中的可读性因此提高了很多。

之前:

![](img/a53506a7c93a1f8678cd0c14e9c422b8.png)

之后:

![](img/994345f0edfb8df835a8946cbeea2943.png)

##### 改进了黑暗主题中快速轮廓中继承成员的着色

Eclipse 默认的深色主题现在包括 JDT 的**快速大纲**中继承成员的样式。这大大提高了深色主题的可读性。可以通过**颜色和字体**首选项页面上的 **Java >继承成员**颜色定义来配置颜色。

之前:

![](img/095b1895318b284b3d6d574ddf48c378.png)

之后:

![](img/1f231fa7c8ed07ef0d78c973c9fbb40c.png)

#### Java 视图和对话框

##### 测试源

在 **Java 构建路径**项目设置中，现在有一个属性**包含测试源**来配置一个源文件夹包含测试源。(注意:测试源必须有自己的输出文件夹)。类似地，对于项目和库，有一个属性**只对测试源**可见。这个设置也适用于类路径容器，如果其中一个容器设置为 **Yes** ，这个值将用于所有包含的库和项目。

![](img/7317fb2aa3d2eb9a753b43e330c75b69.png)

在构建路径设置、包资源管理器和其他位置中，测试源文件夹和依赖项以深色图标显示。这可以在**首选项> Java >外观**中禁用:

![](img/708f53e5d2a2a90f09ab2f265e774c4e.png)

被引用的项目可以包含测试源，并且本身具有测试依赖项。通常，当编译测试源时，构建路径上项目中的测试代码将是可见的。由于这并不总是令人满意的，可以通过将项目可用的不带测试代码的新构建路径属性**设置为 **Yes** 来进行更改。**

![](img/b5e5386cf13cffef837569741d779629.png)

这样配置的构建路径条目在项目名称后有一个装饰【无测试代码】，可以在**首选项>常规>外观>标签装饰**中禁用:

![](img/dc7c9bf012ec7abe040933bdc36af19f.png)

对于每个项目，编译现在分两个阶段完成:首先是所有的主源代码(在构建路径上看不到任何测试代码)，然后是所有的测试源代码。

![](img/f186b2fca1c6db545df5f5d76ccb5788.png)

因此，如果项目是一个模块化的 Java 9 项目，像 JUnit 这样的测试依赖项就不能在`module-info.java`中被引用，因为它们在编译时是不可见的。Maven 使用的解决方案是相同的:当测试依赖项被放到类路径中时，被编译的模块将被自动配置为在编译测试源期间读取未命名的模块，因此测试依赖项将是可见的。

当然，代码完成不会在主要源代码中建议测试代码:

![](img/6f25768f6b9b40219df788952a8d12ea.png)

现在有两个动态 Java 工作集 **Java 主源**和 **Java 测试源**，包含根据**包含测试源**属性的值分组的源文件夹。例如，这可以用来从 problems 视图中删除测试源中的警告:

![](img/943798e77c6a1bb5feb8581cd744fa72.png)

为了实现这一点，创建一个新的过滤器来显示对 **Java 主源**工作集的警告，并使用**工作区上的所有错误**过滤器来选择它:

![](img/016d3983ebb4ce6cda75718084895430.png)

还有专门的过滤器，可以从 Java 搜索结果中快速删除主代码或测试代码中的匹配项:

![](img/e21594fce07951f8f48a05200e276060.png)

类似地，有一个过滤器从**调用层次结构**中移除测试代码:

![](img/89793de99730cc6aa1b9bb1ad99ab036.png)

用于移除测试代码的另一个过滤器存在于**快速类型层次结构**中:

![](img/334597da9622ac0bf53efa47fc697dda.png)

测试源文件夹将在 **New JUnit Test Case** 向导中预先选择

![](img/afcd52d1c7a06d231481118433ecf590.png)

在运行和调试配置中，**类路径**选项卡(或使用 Java 9 启动时的**依赖关系**选项卡)包含一个新选项**排除测试代码**，当从未标记为包含测试源的源文件夹启动 Java 应用程序时，会自动预先选择该选项:

![](img/99fa8c0eb2abbf85861598257d205bc4.png)

当使用 Java 9 启动并且未选择该选项时，将自动添加命令行选项，以便具有非空类路径的模块可以读取未命名的模块。这些命令行选项是可以使用新的**覆盖依赖关系**按钮覆盖的一部分。

##### 在包资源管理器中按字母顺序对库条目排序

库的内容按照类路径的顺序显示。这使得很难通过名称找到特定的库，尤其是当项目有许多依赖项时。在 **Java >外观**首选项页面的**包浏览器**中设置首选项【按字母顺序排序库条目】时，库条目现在可以按字母顺序排序:

![](img/3ffe720a0f322c8027de4d53ea899734.png)

![](img/7568b1b2936d7324330f98cf3a7d4656.png)

该首选项的默认设置是**关闭**。

##### 生成对话框使用动词而不是 OK

Java 工具的 **Generate…** 对话框已经被修改为使用动词而不是 OK。

#### Java 编译器

##### 模块声明搜索中的正则表达式选项

这是一个**实验性的**支持，用于在搜索模块声明时允许在搜索字段中使用正则表达式。这可以被认为是 API 变化的包装。

要从 **Java Search** 下的搜索字段调用正则表达式搜索，请以“/r”开始表达式，即斜杠“/”、字母“r”和空格“”(非制表符)，后跟一个正则表达式，示例如下:

![](img/600b4f4f2b9a712e1a9b9da1f04fdf94.png)

在上面的例子中，所有跟在“/r”后面的字符组成了一个 Java 正则表达式，用来表示一个模块名，这个模块名以零个或多个“n”开头，后跟一个字符串。ver”后面再跟零个或多个任意字符。

另一个例子是搜索所有以`java.x`开头的模块，后面是零个或多个字符，这由正则表达式`/r java\.x.*`给出。认为这是一个“普通”字符，而不是特殊的正则表达式]。

另一个例子是搜索所有以 j 开头、后跟零个或多个字符并以。xml 在 regex 语言中翻译成`/r j.*\.xml`。请注意这里的第一个“.”是特殊的正则表达式字符，而第二个“.”被转义以表示这是一个正常字符。

**注意**:你应该只对模块的**声明**搜索使用它，因为它不是为模块引用实现的。结合正则表达式选择**所有出现**将默认只查找与正则表达式匹配的**声明**，忽略引用。

#### 每个模组的非空订购项目错误

如果一个模块用`@NonNullByDefault`标注，编译器会将其解释为该模块中所有类型的全局缺省值:

```
@org.eclipse.jdt.annotation.NonNullByDefault module my.nullsafe.mod { ...
```

但是，请注意，这需要用 target `ElementType.MODULE`声明一个注释类型，或者根本不用显式 target 声明。bundle `org.eclipse.jdt.annotation`的 2.2.0 和更高版本使用后一种策略，因此支持模块范围的非空缺省值。

##### @NonNullByDefault 改进

当使用基于注释的空分析时，现在有更多的方法来定义哪些未注释的位置被隐式地假定为注释为`@NonNull`:

*   `@NonNullByDefault`如果主空性注释是声明注释，也可以使用基于枚举`DefaultLocation`的注释(以前这仅支持`TYPE_USE`注释)。
*   已经实现了对针对参数的`@NonNullByDefault`注释的支持。
*   多个不同的`@NonNullByDefault`注释(尤其是具有不同默认值的)可能被放置在同一个目标上，在这种情况下，受影响位置的集合被合并。
*   使用元注释`@TypeQualifierDefault`而不是基于`DefaultLocation`的规范的注释现在也被理解了，例如`@org.springframework.lang.NonNullApi`。

bundle `org.eclipse.jdt.annotation`的 2.2.0 版本包含一个注释类型`NonNullByDefault`，可以应用于参数和模块声明(除了以前允许的目标之外)。

##### 测试源

现在支持在测试源上运行 Java 注释处理器。为这些生成的文件的输出文件夹可以在 **Java 编译器>注释处理**的项目属性中配置为**生成的测试源目录**。

![](img/2416e3a2445c74866d8802fa085685fb.png)

##### 添加了“编译器兼容性与使用的 JRE 不匹配”的新首选项

与已用 JRE 不匹配的新首选项**编译器被添加到**编译器首选项构建页面**。**

此首选项指示当项目使用的 JRE 与选定的编译器兼容级别不匹配时所报告问题的严重性。(如使用 JRE 1.8 作为 JRE 系统库的项目，编译器兼容设置为 1.7)。

默认情况下，此首选项的值为 WARNING。

如果正在使用的 JRE 是 9 或更高版本，并且选择了 **- release** 选项，即使编译器兼容性与正在使用的 JRE 不匹配，该选项也将被忽略。

该首选项可以如下所示进行设置:

![](img/171c752c25d837ee2f03f4a1a23065fc.png)

#### Java 格式化程序

##### 新建格式化程序配置文件页面

格式化程序配置文件首选项页面( **Java >代码样式>格式化程序>编辑…** )有了新的外观，这使得设置格式化 Java 代码的首选项更加容易。所有首选项都显示在一个可展开的树中，而不是多个选项卡。

![](img/cd4c6ff72aa41f72d9e4a797dbed229b.png)

您可以使用**过滤**来仅显示名称与特定短语匹配的设置。按值过滤也是可能的(在值过滤器前加一个波浪号)。

![](img/423a48306f844643da617a4fc210a9dd.png)

大多数部分的标题中都有一个 **Modify all** 按钮，你只需点击一下就可以将它们的所有参数设置为相同的值。

![](img/4be76ff6e06bbfb7809bc65fb209e2df.png)

一些偏好设置有更方便的控制。例如，可以使用箭头按钮轻松修改数值。包装策略设置由简单的工具栏控制，因此您可以一次查看和比较多个策略。

![](img/7efecbc31655859855acdd9adfcaaf70.png)

在预览面板中，您现在可以使用自己的代码来立即查看修改后的设置将如何影响它。您还可以查看标准预览样本的原始形式，并对其进行临时修改。

![](img/294dbaf2e3b2e7d72129c0237ec724c3.png)

##### 格式化程序:在列中对齐 Javadoc 标记

格式化程序现在可以用新的方式对齐 Javadoc 标签中的名称和/或描述。在**注释> Javadoc** 下，可以选择格式化程序配置文件编辑器。

![](img/be88aa1da6627252d60191c4a7a212b4.png)

例如， **Align 描述，通过类型**设置分组，现在用于内置的 Eclipse 概要文件中。

![](img/357ce6bc099963edaefdec7518fd5ee8.png)

之前被称为**缩进 Javadoc 标签**的设置现在被称为**将描述与标签宽度**对齐。与 **@param 标签**相关的两个设置的标签也进行了更改，以更好地描述它们的功能。

##### Java 代码格式化程序首选项现在为黑暗主题设计了样式

格式化程序首选项树样式已修复，可在黑暗主题中正常工作。

##### 新的清理动作“删除多余的修改器”

新的清理动作“移除多余的修饰符”移除类型、方法和字段上不必要的修饰符。删除了以下修饰符:

*   接口字段声明:`public`、`static`、`final`
*   接口方法声明:`public`，`abstract`
*   嵌套接口:`static`
*   最终类中的方法声明:`final`

清理动作可以在**不必要的代码**页面上配置为保存动作。

![](img/35e0b4249794ee4f5b1c6b5a188f37cc.png)

#### 调试

##### Java 启动配置的启动配置原型

Java 启动配置现在可以基于原型。

![](img/e8df20209a403a332bf60c0aae86deb9.png)

原型在其关联的 Java 启动配置中植入属性，并在“原型”选项卡中指定设置。

![](img/a10bff7f89e6c37e30b94a0a594a7db6.png)

一旦创建了 Java 启动配置，您就可以覆盖原型中的任何初始设置。您还可以使用原型中的设置来重置 Java 启动配置的设置。Java 启动配置维护一个到其原型的链接，但它是一个完整的独立启动配置，可以被启动、导出、共享等。

![](img/d88da29c4377c956ff924dac2f1934e9.png)

##### 高级源代码查找实现

更精确的**高级的**源代码查找实现，在调试运行时动态加载类的应用程序时特别有用。

新的`org.eclipse.jdt.launching.workspaceProjectDescribers`扩展点可用于启用具有非默认布局的项目的高级源代码查找，如 PDE 插件项目。

新的`org.eclipse.jdt.launching.sourceContainerResolvers`可以用来从远程工件仓库下载源代码 jar 文件，比如 Maven Central 或 Eclipse P2。

高级源代码查找仅影响调试启动，并且可以使用 **Java >调试>使用高级源代码查找(JRE 1.5 及更高版本)**首选项选项来启用或禁用:

![](img/aaae463fa6addc70e84d4aad7dc7cfd9.png)

##### 调试器侦听线程名称的更改

如果线程名在被调试的 JVM 中被更改，调试视图现在会自动更新线程名。如上所述，这显示了 worker 实例的实时信息。

从技术上讲，Java 调试器会自动在 JVM 中添加一个新的(用户不可见的)断点，并在断点命中时通知客户端(如 Debug view)。如果出于某种原因不希望出现这种行为，产品所有者可以通过产品定制禁用它。

属性值为:**org . eclipse . JDT . debug . ui/org . eclipse . JDT . debug . ui . javadebug . listenonthreadnamechanges = false**

##### 为方法退出和异常断点显示的值

当一个**方法退出断点**被命中时，返回的值现在显示在变量视图中。

![](img/300dbdea68ebae337bc439f0b6ca3c25.png)

类似地，当一个**异常断点**被命中时，被抛出的异常被显示。

![](img/e10d1655199ec9b42a08c9e9d4bd35de.png)

##### 显示视图重命名为调试外壳

**显示视图**已被重命名为**调试外壳**，以更好地匹配该视图的功能和目的。此外，在新打开的调试 Shell 中会显示一个 java 注释，解释何时以及如何使用它。

![](img/279b63f956a7d03a8cef184bb85190af.png)

### 还有更多…

您可以在本页的[中找到更多值得注意的更新。](http://tools.jboss.org/documentation/whatsnew/jbosstools/4.6.0.Final.html)

## 下一步是什么？

现在 JBoss Tools 4.6.0 和 Red Hat Developer Studio 12.0 已经出来了，我们已经在为 Eclipse 2018-09 的下一个版本工作了。

尽情享受吧！

*Last updated: March 19, 2020*