# 将 Java 应用程序迁移到 Quarkus:经验教训

> 原文：<https://developers.redhat.com/blog/2019/04/12/migrating-java-applications-to-quarkus-lessons-learned>

在公开发布后的几天内将应用程序从一个基础良好的框架迁移到一个全新的框架听起来很疯狂，对吗？在这样做之前，我问了自己几个问题，比如:我为什么要这样做？这个新框架稳定吗？会有什么收获？对我来说，其中最重要的是:为什么？

为了回答这个问题，我开始思考我的应用程序的性能——在这个例子中，是引导时间——并问自己是否对我的应用程序启动所用的实际时间感到满意。答案是否定的。现在，这是使用微服务(主要是在无服务器架构上)时要考虑的最重要的指标之一。

本文的目标是为现有 Java EE 应用程序向 Quarkus 的基本迁移提供一个参考点。出于这个原因，我将省去几行文章，不再介绍 Quarkus，而主要关注迁移部分。如果你不知道 Quarkus 是什么，那么我推荐你阅读这篇文章，并访问 [Quarkus 主页](https://quarkus.io)。

在本文中，我将尝试说明所有的更改，或者至少是最重要的更改，我必须对现有的应用程序进行这些更改，才能使它在 Quarkus 上运行良好。

## 一点背景

使用的应用程序是一个带有几个插件的 [Telegram API + Bot](https://github.com/rebase-it/rebot) ,我一直将它作为一个游乐场，在那里我可以尝试新事物、框架等。当创建这个应用程序时，我们决定创建一个 Telegram Java EE API，这样我们就可以在任何实现所有 EE 7 标准的应用服务器或框架中运行它。主要原因是能够使用 CDI、JPA 和 Rest 等很酷的东西。它是为野生蜂群设计的，现在叫做[荆棘尾巴](https://thorntail.io/)。自那以后，框架没有变化。Quarkus 一经发布，我就决定进一步了解它，第一印象非常惊人。它支持我最初拥有的所有依赖项，最重要的是，支持用 GraalVM 创建原生容器映像的能力。所以，我想，为什么不试试呢？

当然，像这样的迁移并不容易。

## 第一步

当我开始迁移应用程序时，我将工作分成了更小的部分，例如:

*   属国
*   分而治之(逐个迁移模块)
*   代码更改
*   依赖性审查

## 属国

可以想象，我做的第一件事是完全删除 Thorntail 依赖项——在我的例子中，从父 pom 中删除 BOM，从每个模块中删除相关的依赖项。我改了这个:

> ```
> <dependency>
>     <groupId>io.thorntail</groupId>
>     <artifactId>bom</artifactId>
>     <version>${version.io.thorntail}</version>
>     <scope>import</scope>
>     <type>pom</type>
> </dependency>
> ```

收件人:

> ```
> <dependency>
>     <groupId>io.quarkus</groupId>
>     <artifactId>quarkus-bom</artifactId>
>     <version>${io.quarkus.version}</version>
>     <type>pom</type>
>     <scope>import</scope>
> </dependency>
> ```

接下来，我还必须删除主要用于 CDI 目的的 **javaee-api** 依赖项，并用 **quarkus-arc** 替换它，它是提供依赖项注入的核心库之一。因此，我将它从父 pom 中移除，并对每个 *pom.xml:* 进行了一次大替换

> ```
> <dependency>
> - <groupId>javax</groupId>
> - <artifactId>javaee-api</artifactId>
> + <groupId>io.quarkus</groupId>
> + <artifactId>quarkus-arc</artifactId>
> + <scope>provided</scope>
> </dependency>
> ```

下面你可以找到一些我必须更改的依赖项:

*   io . thorn tail:jaxrs-> io . quar kus:quar kus-rest easy
*   io . thorn tail:JPA-> io . quar kus:quar kus-hibernate-ORM
*   数据验证-> io . quar kus:quar kus-hibernate-validator。
*   com . h2 database-> io . quar kus:quar kus-jdbc-H2(quar kus 已经有一些 JDBC 扩展，H2、MariaDB、PostgreSQL 对于甲骨文来说，这里有一个很好的起点。

在项目层次结构中，[这个](https://github.com/rebase-it/rebot/tree/master/rebot-telegram)是产生 runnable jar 的模块，在这里我们需要取出 [Thorntail maven 插件](https://github.com/rebase-it/rebot/blob/0.x/rebot-telegram/pom.xml#L144-L161)并放入 Quarkus maven 插件。这是非常重要的一步；没有它，你将无法使用夸库斯。

请注意，在处理多模块项目时，您需要注意 Quarkus 依赖关系。例如，如果您的一些模块公开了一个 Rest 端点，您将需要使用 **quarkus-resteasy** 扩展。但是，请注意，为了启用这个扩展，还需要在 runnable 模块上声明这个依赖关系。

示例:

> ```
> <dependency>
>     <groupId>io.quarkus</groupId>
>     <artifactId>quarkus-resteasy</artifactId>
>     <scope>provided</scope>
> </dependency>
> 
> ```

为了确保扩展按预期安装，您可以随时验证日志。Quarkus 让您知道安装了哪些扩展，当应用程序准备好时，会打印出类似的行:

> *INFO [io.quarkus]已安装功能:[agoral，cdi，hibernate-orm，jdbc-h2，narayana-jta，resteasy，scheduler]*

现在是乐趣真正开始的时候，有很多问题，缺少依赖等。第一次执行 **mvn 编译 quarkus:dev** 或者 **mvn 清理包**的时候，不要慌；会发生很多问题，记住，分而治之！

## 代码更改

首先，我承认我有一点害怕，所以这是一个合适的时间来喘口气，喝杯咖啡，然后把事情做完！

我用来简化事情的方法是首先在较小的模块上进行构建，然后继续完成第一个目标:成功的构建！此时，我从主 pom.xml 中删除了所有依赖项，以便只构建 CoreAPI 和 runnable jar。

检查完依赖项后，我的 IDE 开始抱怨不满意的依赖项。在这种情况下，存在与 EJB 注释相关的问题。正如你在这里看到的，我在应用程序启动期间使用@Startup、@PostConstruct 和@PreDestroy 注释来执行任务。解决这个问题的一个简单方法是删除 EJB 注释，继续使用后置构造，并添加一个 CDI 注释(例如，在类级别上应用程序)。但是 Quarkus 提供了一种奇特的方式来控制应用程序的生命周期，在[这个](https://github.com/rebase-it/rebot/blob/master/rebot-telegram/src/main/java/it/rebase/rebot/Startup.java#L61-L74)例子中可以观察到启动和关闭事件。有关该特定功能的更多详细信息，请参考 Quarkus [文档](https://quarkus.io/guides/application-lifecycle-events-guide)。

最简单的部分之一是核心 API ，以及一些配置 CDI 的基本变化。在以前的版本中，我使用服务提供者实现，并通过服务提供者机制发现新的提供者。使用 Quarkus，我可以完全删除服务提供者文件，并用 [beans.xml](https://github.com/rebase-it/rebot/blob/master/rebot-telegram-api/rebot-telegram-api/src/main/resources/META-INF/beans.xml) 文件替换它。在我的例子中，所有使用 CDI 的模块都必须添加 beans.xml 文件，这样 Quarkus 才能正确地发现 CDI beans。一旦完成，我就可以向前迈进，尝试构建 CoreAPI。如果您面临一个问题，就像下面的例子，您可能想看的第一个地方是问题中列出的依赖项是否有 beans.xml。

我必须解决的一个有趣的问题是 CDI 循环依赖，我注入 ClassA 和 ClassB，但是 ClassB 已经在注入 ClassA 了。我不知道索恩泰尔为什么会接受，但夸库斯非常严格。

至此，我成功构建了 CoreAPI 它需要一些[系统属性](https://github.com/rebase-it/rebot/blob/master/rebot-telegram-api/rebot-telegram-api/src/main/java/it/rebase/rebot/telegram/api/UpdatesReceiver.java#L63)来正确配置应用程序。如果未设置所需的属性，应用程序将无法启动。以前，API 只需要系统属性；在 Quarkus 中，这种配置通过读取[*micro profile-config . properties*](https://github.com/rebase-it/rebot/blob/master/rebot-telegram/src/main/resources/META-INF/microprofile-config.properties)得到了改进。一旦应用程序属性的配置完成，我继续插件。

我转移到 Quarkus 的第一批插件是最简单的，不需要持久性或缓存层，例如，[这个简单的 ping 插件](https://github.com/rebase-it/rebot/tree/master/rebot-plugins/rebot-ping-plugin)。

另一个注意点是私有成员的使用；如果您得到类似下面的消息，您将不得不更改访问修饰符。或者，如果您真的认为访问修饰符需要是私有的，您可能希望使用 package-private 来代替:

> 在应用程序 beans 中发现私有成员的不推荐用法(改为使用 package-private)

哇，现在我已经用 Quarkus 上的第一个插件运行了我的应用程序:

> INFO [it.reb.reb.Startup] (main)应用程序启动...
> INFO[io . quar kus](main)quar kus 0 . 11 . 0 开始于 0.359s .
> INFO[io . quar kus](main)已安装特性:[cdi]

因为我的应用程序的大多数日志记录都在精细日志级别之下，所以我想配置日志记录，只打印属于我的应用程序的调试消息。为了实现这一点，我必须在**micro profile-config . properties**上添加一些日志配置，就像下面的例子:

> ```
> # DEBUG console logging
> quarkus.log.console.enable=true
> quarkus.log.console.format=%d{HH:mm:ss} %-5p [%c] %s%e%n
> quarkus.log.console.level=TRACE
> 
> # TRACE file logging
> quarkus.log.file.enable=true
> quarkus.log.file.path=/tmp/quarkus.log
> quarkus.log.file.level=TRACE
> quarkus.log.file.format=%d{HH:mm:ss} %-5p [%c{2.}]] (%t) %s%e%n
> 
> # custom loggers
> quarkus.log.category."it.rebase".level=TRACE
> ```

### 调度程序

我使用的一个插件是 EJB 计时器。然而，为了利用 Quarkus 提供的一切，我用 quarkus-scheduler 扩展(Quartz under the hood)替换了 EJB 计时器，它也可以处理注释。而且，说实话，它的用法很简单，那就是:

> ```
> @Scheduled(every = "1800s", delay = 30)
> ```

及其依赖性:

> ```
> <dependency>
>     <groupId>io.quarkus</groupId>
>     <artifactId>quarkus-scheduler</artifactId>
>     <scope>provided</scope>
> </dependency>
> ```

### Infinispan 缓存

我的应用程序使用 Infinispan 嵌入式缓存，我使用特定的限定符和生成器来配置不同目的的不同缓存，并且我可以根据我的插件需求注入特定的缓存。当我构建缓存模块时，我遇到了以下问题:

> [错误]:生成步骤 io . quar kus . arc . deployment . arcannotationprocessor # Build 引发了异常:javax . enterprise . inject . SPI . definition 异常:侦听器没有绑定:org . infini span . jcache . annotation . cacheputinterceptor

已通过更新 Infinispan 依赖关系修复此问题:

*   将版本从 9.1.1.Final 提升到 9.4.9.Final
*   将相关性从 org . infini span:infini span-embedded 更新为 org . infini span:infini span-CDI-embedded

从这一点来看，我有一些 CDI 问题，比如:

*   org . infinispan . manager . embeddedcachemanager # defaultCacheContainer 类型的未满足的相关性
*   org . infinispan . CDI . embedded . infinispanExtension embedded # infinispanExtension 类型的未满足的依赖关系
*   org.infinispan.Cache 类型`java.lang.string`和限定符[@Default]的未满足依赖关系

基本上，这个问题告诉我们，上面的方法没有生产者。为了解决这个问题，我必须手动创建生产者来满足缺失的依赖关系；首先，我修改了默认缓存，删除了旧的缓存配置，如下所示:

> ```
> @Produces
> @ConfigureCache("default-cache")
> @DefaultCache
> public Configuration specialCacheCfg(InjectionPoint injectionPoint) {
> 
> ...
> 
> }
> ```

并添加了新的缓存配置:

> ```
> private DefaultCacheManager defaultCacheManager;
> 
> @Produces
> @DefaultCache
> public Cache<String, String> returnDefaultCacheStringObject() {
>     return defaultCacheContainer().getCache();
> }
> 
> @Produces
> public Configuration defaultCacheProducer() {
>     log.info("Configuring default-cache...");
>     return new ConfigurationBuilder()
>             .indexing()
>             .autoConfig(true)
>             .memory()
>             .size(1000)
>             .build();
> }
> 
> @Produces
> public EmbeddedCacheManager defaultCacheContainer() {
>     if (null == defaultCacheManager) {
>         GlobalConfiguration g = new GlobalConfigurationBuilder()
>                 .nonClusteredDefault()
>                 .defaultCacheName("default-cache")
>                 .globalJmxStatistics()
>                 .allowDuplicateDomains(false)
>                 .build();
>         defaultCacheManager = new DefaultCacheManager(g, defaultCacheProducer());
>     }
>     return defaultCacheManager;
> }
> 
> @Produces
> public InfinispanExtensionEmbedded defaultInfinispanExtensionEmbedded() {
>     return new InfinispanExtensionEmbedded();
> }
> ```

我必须更新的另一点是[事件监听器。](http://infinispan.org/docs/stable/user_guide/user_guide.html#listeners_and_notifications)以下是修改前的一个例子:

> ```
> @Listener
> public class KarmaEventListener {
> ...
>     @CacheEntryCreated
>     public void entryCreated(@Observes CacheEntryCreatedEvent event) {
>     ...
>     }
> }
> ```

### 持久模块

除了前面在 Thorntail 上描述的依赖关系更改之外，要使用数据库，您必须在 **resources/** 路径下创建一个名为 **project-defaults.yaml** 的文件，并在其中声明数据库信息以及 persistence.xml 文件。有了 Quarkus，这可以通过在相同的路径上提供一个名为 **application.properties** 的文件来完成，如这里的[所述](https://quarkus.io/guides/hibernate-orm-guide)。或者，您可以在运行二进制文件时通过命令行使用系统属性提供所有数据库设置。因此，这两个旧文件被 application.properties 替换了；这里有一个例子:

> ```
> quarkus.datasource.url=jdbc:h2:file:/opt/h2/database.db;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
> quarkus.datasource.driver=org.h2.Driver
> quarkus.datasource.username=rebot
> quarkus.datasource.password=rebot
> quarkus.datasource.max-size=8
> quarkus.datasource.min-size=2
> quarkus.hibernate-orm.database.generation=update
> quarkus.hibernate-orm.log.sql=false
> ```

另一个小变化是对 EntityManager 注入的调整，必须从:

> ```
> @PersistenceContext(unitName = "rebotPU")
> private EntityManager em;
> ```

到

> ```
> @Inject
> EntityManager em;
> ```

## 依赖性审查

我完全移除了 resteasy 依赖项，转而使用 quarkus-resteasy。移除 resteasy 后，新的问题开始出现:

> 原因:Java . lang . classnotfoundexception:org . glassfish . jersey . client . jerseyclientbuilder

为了解决这个问题，必须在使用**javax . ws . RS . client . client builder**的模块上添加三个新的依赖项。新的依赖项包括:

*   核心:jersey-client
*   org.glassfish.jersey.inject:新泽西-hk2
*   org . glassfish . jersey . media:jersey-media-JSON-Jackson

进一步研究这一点或放弃使用 ClientBuilder 肯定是值得的。

> WARN:rest easy 002145:NoClassDefFoundError:无法从 jar 加载内置提供程序 org . JBoss . rest easy . plugins . providers . data source provider:file:/home/spol ti/. m2/repository/org/JBoss/rest easy/rest easy-core/4 . 0 . 0 . 0 . 0 . beta 8/rest easy-core-4 . 0 . 0 . 0 . beta 8 . jar！/META-INF/services/javax . ws . RS . ext . providers
> Java . lang . noclassdeffound error:javax/activation/data source

通过在因该问题而失败的模块上添加 **quarkus-hibernate-orm** 修复了这种情况。

我想我已经涵盖了我在将应用程序迁移到 Quarkus 的过程中遇到的最重要的几点。这是第一步；不幸的是，我无法构建本机二进制文件，因为 Infinispan 嵌入式缓存依赖项还没有准备好。毫无疑问，下一步将是寻找它的替代品，这将是一篇新文章的主题。

*Last updated: January 5, 2022*