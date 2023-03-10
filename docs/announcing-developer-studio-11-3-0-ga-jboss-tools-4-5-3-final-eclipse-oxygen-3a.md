# 宣布 Developer Studio 11.3.0.GA，JBoss Tools 4 . 5 . 3 for Eclipse oxygen . 3a

> 原文：<https://developers.redhat.com/blog/2018/04/24/announcing-developer-studio-11-3-0-ga-jboss-tools-4-5-3-final-eclipse-oxygen-3a>

Eclipse Oxygen.3a 的 [JBoss Tools 4.5.3](http://tools.jboss.org/downloads/jbosstools/oxygen/4.5.3.Final.html) 和 [JBoss Developer Studio 11.3](http://tools.jboss.org/downloads/devstudio/oxygen/11.3.0.GA.html) 社区版在这里等你。看看吧！

## 装置

JBoss Developer Studio 的安装程序中预捆绑了所有东西。只需从我们的 [JBoss 产品页面](https://www.jboss.org/products/devstudio.html)下载并运行它，如下所示:

```
java -jar jboss-devstudio-<installername>.jar
```

JBoss Tools 或自带 Eclipse(BYOE)JBoss Developer Studio 需要更多:

这个版本至少需要 Eclipse 4.7 (Oxygen)，但是我们建议使用最新的 [Eclipse 4.7.3a Oxygen JEE 包](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/oxygen3a)，因为这样你就可以预装大多数依赖项。

一旦你安装了 Eclipse，你可以在 Eclipse Marketplace 的“JBoss Tools”或者“Red Hat JBoss Developer Studio”下找到我们。

对于 JBoss 工具，你也可以直接使用我们的更新站点。

```
http://download.jboss.org/jbosstools/oxygen/stable/updates/
```

## 什么是新的？

我们这次发布的主要焦点是 Java10 的采用，对基于容器的开发的改进和错误修复。Eclipse Oxygen 本身有很多新的很酷的东西，但是让我强调一下 Eclipse Oxygen 和 JBoss 工具插件中我认为值得一提的几个更新。

### OpenShift 3

#### CDK 和 Minishift 服务器适配器改善了开发人员体验

##### MINISHIFT_HOME 设置

当同时使用 CDK 和上游 Minishift 时，建议通过 MINISHIFT_HOME 变量来区分环境。以前可以使用该参数，但需要两步过程:

*   首先创建服务器适配器(通过向导)
*   然后在服务器适配器编辑器中更改 MINISHIFT_HOME

现在可以从服务器适配器向导中设置该参数。现在，当您创建服务器适配器时，一切都已正确设置。

让我们看一个 CDK 服务器适配器的例子。

从**服务器**视图中，选择新的服务器菜单项，并在过滤器中输入 cdk:

![](img/e9cf57d28e49509e2a9aa5aa362345ed.png)

选择 Red Hat 容器开发工具包 3.2+

![](img/d2806d5880ec0e5d2c737ae676af039d.png)

点击**下一个**按钮:

![](img/b236e11493eaf3722184e59e31585ddf.png)

MINISHIFT_HOME 参数可以在这里设置，默认为。

##### CDK 和 Minishift 服务器适配器运行时下载

当同时使用 CDK 和上游 Minishift 时，您需要事先下载 CDK 或 Minishift 二进制文件。现在可以在创建服务器适配器时将运行时下载到特定的文件夹中。

让我们看一个 CDK 服务器适配器的例子。

从**服务器**视图中，选择新的服务器菜单项，并在过滤器中输入 cdk:

![](img/63873449bc768cd2d0ed00c552d0c836.png)

选择 Red Hat 容器开发工具包 3.2+

![](img/5ab2eebeeb24ca2a1fcb9db9fca2e8e4.png)

点击**下一个**按钮:

![](img/91e1f0f5b0f049ff9f816d463e71e8c6.png)

为了下载运行时，单击**下载并安装运行时…** 链接:

![](img/84d8721a1101a1efb768644fa2fa5c0b.png)

选择要下载的运行时版本

![](img/eec23a80b8e0435a10c2e3de6d99b3d7.png)

点击**下一个**按钮:

![](img/7a01e5f4e59a0ca2f3e543ba30f2ba52.png)

您需要一个帐户来下载 CDK。如果您已经配置了凭据，请选择要使用的凭据。如果没有，请点击**添加**按钮来添加您的凭证。

![](img/95dc4dcfcba0afab5dfcfc4738f5a195.png)

点击**下一个**按钮。您的凭据将得到验证，成功后，您必须接受许可协议:

接受许可协议并点击**下一步**按钮:

![](img/da68c1df67e728101607dbb99d7cb68d.png)

您可以选择要安装运行时的文件夹。设置完成后，点击**完成**按钮:

将开始下载运行时，您应该会在服务器适配器向导上看到进度:

![](img/ee55874d3f1b477af7216b101ca0f785.png)

下载完成后，您会注意到 **Minishift Binary** 和 **Username** 字段已被填充:

![](img/1714c8327e31d028661200e82f2f7dc0.png)

点击**完成**按钮创建服务器适配器。

请注意，如果这是你第一次安装 CDK，你必须执行初始化。在**服务器**视图中，右键单击服务器，选择**设置 CDK** 菜单项:

![](img/bdd51956de38535297b0a4301399d793.png)

![](img/ff257383a6f95cb9aa23e1c0bc07bb95.png)

请注意，如果在用户批准后检测到 MINISHIFT_HOME 环境未初始化，当您启动 cdk 服务器适配器时，也会自动运行 **setup-cdk** 命令。

#### Minishift 服务器适配器

添加了新的服务器适配器来支持上游微移位。虽然服务器适配器本身的功能有限，但它能够通过其 minishift 二进制文件启动和停止 Minishift 虚拟机。在 Servers 视图中，单击 **New** ，然后键入 minishift，这将显示一个设置和/或启动 Minishift 服务器适配器的命令。

![](img/8e7b09ac22721e8aa6e5f4d636b51c70.png)

您所要做的就是设置 minishift 二进制文件的位置、虚拟化管理程序的类型和可选的 Minishift 配置文件名称。

![](img/bb6923808642abe633185950fe1d0b48.png)

完成后，将创建一个新的 Minishift 服务器适配器，并在 Servers 视图中可见。

![](img/79a6f65e1e585c7ac872dddb99c75d79.png)

一旦服务器启动，Docker 和 OpenShift 连接应该出现在各自的视图中，允许用户快速创建一个新的 Openshift 应用程序，并开始在一个高度可复制的环境中开发他们的 AwesomeApp。

![](img/14165e27ac46a8fbec0afe18f37fa84d.png)

![](img/ea0beef4f98b0b300579ea375892885e.png)

在其他服务/组件需要或使用该凭证域的情况下，凭证框架仍然支持 JBoss.org 凭证。

### 保险丝工具

#### 融合视角下的新捷径

Fuse Integration 透视图中现在提供了 Java、启动和调试透视图以及基本导航操作的快捷方式。

结果是工具栏中的一组按钮:

![](img/d9741b7f15b6ce46f4e5b6199e972db9.png)

所有相关的键盘快捷键也是可用的，比如 Ctrl+Shift+T 来打开一个 Java 类型。

#### 性能改进:加载 Camel 端点的高级选项卡

Camel 端点的 Properties 视图中的“Advanced”选项卡的加载时间大大缩短了。

![](img/5feb57c8cc93a279d850608f958357ef.png)

以前，对于有很多参数的 Camel 组件，加载 Advanced 选项卡需要几秒钟。例如，对于文件组件，需要大约 3.5 秒。现在需要大约 350 毫秒。装载时间减少了十分之一。(参见这篇关于响应时间的有趣的[文章)](https://www.nngroup.com/articles/response-times-3-important-limits/)

如果您注意到其他地方表现缓慢，您可以使用 [Fuse Tooling issue tracker](https://issues.jboss.org/browse/FUSETOOLS) 提交报告。Fuse 工具团队非常感谢您的帮助。您的反馈有助于我们的开发优先级，并改善 Fuse 工具的用户体验。

#### 显示与提议的 Camel 版本相对应的保险丝版本

当您创建一个新项目时，您从列表中选择 Camel 版本。现在，Camel 版本列表包括 Fuse 版本，以帮助您选择与您的生产版本相对应的版本。

![](img/9e2dcc791717a1c014b4e9b98fe71aea.png)

#### 更新组件及其定义之间相似 id 的验证

从 Camel 2.20 开始，您可以对组件名及其定义使用类似的 id，除非提供了特定的属性“registerEndpointIdsFromRoute”。验证过程检查 Camel 版本和“registerEndpointIdsFromRoute”属性的值。

例如:

```
<from id="timer" uri="timer:timerName"/>
```

#### 改进了全局 Bean 上工厂方法的方法选择指南

当在全局 bean 上选择工厂方法时，在用户界面中提出了许多可能性。现在，全局 bean 的工厂方法列表仅限于那些与 bean 的全局定义类型(bean 或 bean 工厂)的约束相匹配的方法。

#### 自定义图表中的 EIP 标签

编辑器视图的 Fuse 工具首选项页面包括一个新的“首选标签”选项。

![](img/2f6224ffbfd4983c9aca37403379028e.png)

使用此选项可以定义在编辑器的设计视图中显示的 EIP 组件(终结点除外)的标签。

![](img/8b8871986209735f13ec3d883ecbaa4b.png)

#### Fuse Ignite 技术扩展模板

“使用骆驼路线的自定义步骤”的现有模板已更新，可用于 Fuse 7 技术预览版 4。

添加了两个新模板:-使用 Java Bean 的自定义步骤-自定义连接器

![](img/886f56130e01ebc6c695d13a05040b08.png)

#### 改进了创建 Fuse 集成项目的向导

创建向导为目标部署环境提供了更好的指导:

![](img/c9af24a05d1d0592eddf32a72ba2c497.png)

有更多的地方可供选择模板，它们现在根据目标环境进行筛选:

![](img/11158a733b8bbe0d48dd39caedf2ad17.png)

它还指出其他地方为高级用户找到不同的例子(见前面截图底部的链接)。

#### Camel Rest DSL 编辑器(技术预览)

Camel 提供了一个 Rest DSL 来帮助通过 Rest 端点进行集成。Fuse Tooling 现在提供了一个只读模式的新标签来可视化定义的 Rest 端点。

![](img/f49453d4cab2d94cc3fafe4de73ef3cd.png)

它当前在技术预览中，需要在窗口→首选项→融合工具→编辑器→启用只读技术预览 REST DSL 选项卡中激活。

工作仍在进行中，非常欢迎对这一新功能的反馈，您可以评论[这部 JIRA 史诗](https://issues.jboss.org/browse/FUSETOOLS-1287)。

#### 推土机升级和迁移

从 Camel < 2.20 to Camel > 2.20 升级时，Dozer 依赖项已升级到不向后兼容的版本。如果您在 Fuse Tooling 中打开基于 Dozer 的数据转换，它将建议迁移用于转换的文件(技术上更改名称空间)。它允许继续使用数据转换编辑器，并且在大多数情况下，数据转换在 Camel > 2.20 的运行时工作。

### 休眠工具

#### Hibernate 运行时提供程序更新

对可用的 Hibernate 运行时提供程序进行了大量的添加和更新。

##### 新的 Hibernate 5.3 运行时提供程序

随着 Hibernate 5.3 流中测试版的发布，是时候发布相应的 Hibernate 5.3 运行时提供程序了。这个运行时提供程序包含 Hibernate 核心版本 5.3.0.Beta2 和 Hibernate 工具版本 5.3.0.Beta1。

![](img/69980cadd216945eba6029a51b4bde68.png)

##### 其他运行时提供程序更新

Hibernate 5.0 运行时提供程序现在合并了 Hibernate 核心版本 5.0.12.Final 和 Hibernate 工具版本 5.0.6.Final。

Hibernate 5.1 运行时提供程序现在合并了 Hibernate 核心版本 5.1.12.Final 和 Hibernate 工具版本 5.1.7.Final。

Hibernate 5.2 运行时提供程序现在合并了 Hibernate 核心版本 5.2.15.Final 和 Hibernate 工具版本 5.2.10.Final。

### java 开发工具 _jdt)

#### 支持 Java 10

最大的部分是对局部变量类型推断的支持。

##### 添加 Java 10 JRE

认识 Java 10 启动的基本必要性

![j10](img/362ff4ec81a0853367199a45eef6342f.png)

以及编译器兼容性选项 10

![](img/08bf8f69c69eec2444139d1745a7c3e5.png)

##### JEP 286 var -编译

支持编译变量，如下所示

![](img/636424aaf88112afaa24c2720ac41416.png)

按预期标记编译器错误，如下所示

![](img/a6b9eaf191868329c1744ddee3a97ac0.png)

允许在 var 位置完成

![](img/f0f4f433b7236d7b236535e90300f974.png)

不允许在 var 的地方完成

![](img/8de3c1500242d70120fc2c9f7dfc72e1.png)

悬停以显示 javadoc

![](img/b0885f9e5b49ea96f1697d127d0e2cd7.png)

使用快速助手将 var 转换为适当的类型

![](img/b946b35069a5ac5b432ee6f6744dfa95.png)

使用快速助手将类型转换为变量

![](img/e2b82b4d25c4c82b32bfc8abb53117fb.png)

### 一般

### 凭证框架

#### 日落 jboss.org 证书

`Download Runtimes`和`CDK Server Adapter`使用凭证框架来管理凭证。但是，不能再使用 JBoss.org 凭据，因为这些组件使用的基础服务不支持这些凭据。

### 还有更多…

您可以在本页的[中找到更多值得注意的更新。](http://192.168.99.100:4242/documentation/whatsnew/jbosstools/4.5.3.Final.html)

随着 JBoss Tools 4.5.3 和 Developer Studio 11.3 的发布，我们已经在为 Eclipse Photon 的下一个版本工作了。

*Last updated: September 9, 2022*