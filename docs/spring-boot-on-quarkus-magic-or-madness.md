# 夸尔库斯上的 Spring Boot:魔法还是疯狂？

> 原文：<https://developers.redhat.com/blog/2021/02/09/spring-boot-on-quarkus-magic-or-madness>

想了解更多关于使用 Quarkus 开发应用程序的信息吗？下载我们为 Spring 开发者准备的免费电子书[*Quarkus*](https://red.ht/quarkus-spring-devs)，帮助熟悉 Spring 的 Java 开发者快速轻松地过渡。

[Quarkus](https://quarkus.io) 是一个为 OpenJDK HotSpot(或 zSeries 上的 OpenJ9)和 GraalVM 定制的 [Java](https://developers.redhat.com/topics/enterprise-java) 栈，由优化的 Java 库和标准精心制作而成。对于构建高可伸缩性的应用程序来说，它是一个很好的选择，同时比其他 Java 框架使用更少的 CPU 和内存资源。这些应用可以是传统的网络应用、无服务器应用，甚至是作为服务的[功能](https://quarkus.io/guides/funqy)。

有许多记录在案的组织将其应用程序迁移到 Quarkus 的实例。在这篇文章中，让我们来看看这样一条从 [Spring Boot](https://spring.io/projects/spring-boot) 到夸尔库斯的迁徙之路，这部分是魔法，部分是疯狂！神奇的是，在不修改一行代码的情况下，挥挥手就能完成移植。疯狂将试图找出它是如何做到的。

## 应用程序

该应用程序是一个简单的“待办事项”任务管理系统。用户可以输入待办事项，然后在完成后检查它们。这些条目存储在一个 [PostgreSQL](https://www.postgresql.org/) 数据库中。所有应用程序的源代码都可以在[这里](https://github.com/edeandrea/todo-spring-quarkus)找到。有一个版本使用 [Gradle](https://gradle.org/) 而不是 [Maven](https://maven.apache.org/) 作为`gradle`分支上的构建工具。

### 启动数据库

该应用程序需要一个 PostgreSQL 数据库，所以我们要做的第一件事是使用 Docker 或 [Podman](https://podman.io/) 在本地启动一个实例:

```
docker run --ulimit memlock=-1:-1 -it --rm=true --memory-swappiness=0 --name tododb -e POSTGRES_USER=todo -e POSTGRES_PASSWORD=todo -e POSTGRES_DB=tododb -p 5432:5432 postgres:13

```

端口 5432 上的 PostgreSQL 11.5 实例现在应该正在运行。应该创建用户`todo`可以使用密码`todo`访问的`tododb`模式。

### 运行应用程序

通过发出命令`./mvnw clean spring-boot:run`运行应用程序。如果你想用 Gradle 代替 Maven，首先切换到`gradle`分支(`git checkout gradle`)并运行命令`./gradlew clean bootRun`。

你应该看到标准的 Spring Boot 横幅:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

INFO 70823 --- [  restartedMain] i.q.todospringquarkus.TodoApplication    : Started TodoApplication in 4.392 seconds (JVM running for 5.116)

```

记下启动时间。我们稍后将再次讨论这个问题。

应用程序运行后，在您喜欢的浏览器中导航到 [http://localhost:8080](http://localhost:8080) 。您应该会看到如图 1 所示的主应用程序屏幕。

[![MainApplicationScreen](img/3fc63ebd29e07380d4c3c43f8a4c174c.png "Main Application Screen")](/sites/default/files/blog/2021/01/spring-todo-1.png)Figure 1 - Main Application Screen

Figure 1: Initial application screen.

稍微摆弄一下这个应用程序。在文本框中键入新的待办事项，然后按 Enter 键。该待办事项将显示在列表中，如图 2 所示。

[![TodoAdded](img/19b4c25b556250cfb6acbd69625ac536.png "Todo Added")](/sites/default/files/blog/2021/01/spring-todo-2.png)Figure 2 - Todo Added

Figure 2: Add a new todo.

1.  单击待办事项旁边的空圆圈以完成它，或取消选中它以将其标记为未完成。
2.  点击 **X** 删除待办事项。
3.  页面底部的 **OpenAPI** 链接将打开应用的 OpenAPI 3.0 规范。
4.  **Swagger UI** 链接打开嵌入式 [Swagger UI](https://swagger.io/tools/swagger-ui/) ，可以用来直接执行一些 [RESTful 端点](https://en.wikipedia.org/wiki/Representational_state_transfer)。
5.  **普罗米修斯指标**链接指向[普罗米修斯指标端点](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics-export-prometheus)，它会被[普罗米修斯](https://prometheus.io/)间歇地刮擦。
6.  **健康检查**链接打开 Spring Boot 曝光的[内置健康检查](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health)。

继续玩一会儿，看看它是如何工作的。做完后别忘了回来！完成后，使用键盘上的`CTRL-C`停止应用程序。

### 检查内部

该应用程序是一个全功能的 Spring Boot 应用程序，使用以下功能:

*   [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html) 构建休息层:
    *   打开`[src/main/java/io/quarkus/todospringquarkus/TodoController.java](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/java/io/quarkus/todospringquarkus/TodoController.java)`找到 Spring MVC RESTful 控制器，暴露用户界面可用的各种端点。
*   [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html) 用于定义关系实体以及存储和检索它们:
    *   打开`[src/main/java/io/quarkus/todospringquarkus/TodoEntity.java](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/java/io/quarkus/todospringquarkus/TodoEntity.java)`找到 [Java Persistence API (JPA)](https://www.oracle.com/java/technologies/persistence-jsp.html) 实体，代表存储 todos 的关系表。
    *   打开`[src/main/java/io/quarkus/todospringquarkus/TodoRepository.java](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/java/io/quarkus/todospringquarkus/TodoRepository.java)` 找到 [Spring Data JPA Repository](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories) ，显示 TodoEntity 的所有创建、读取、更新和删除操作。
*   [Spring Boot 致动器](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)用于提供操作能力，包括健康检查和指标收集。
*   [SpringDoc OpenAPI 3](https://springdoc.org/) 用于生成和公开 RESTful API 信息以及嵌入式 Swagger UI 端点。
*   [Prometheus 测微计注册表](https://micrometer.io/docs/registry/prometheus)用于向 Prometheus 展示指标。
*   打开`[src/main/resources/META-INF/resources](https://github.com/edeandrea/todo-spring-quarkus/tree/main/src/main/resources/META-INF/resources)`找到使用的用户界面组件。

#### 配置

打开`[src/main/resources/application.properties](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/resources/application.properties)`找到应用程序配置:

```
spring.jpa.hibernate.ddl-auto=create-drop
spring.datasource.url=jdbc:postgresql://localhost:5432/tododb
spring.datasource.username=todo
spring.datasource.password=todo

springdoc.api-docs.path=/openapi
springdoc.swagger-ui.path=/swagger-ui

management.endpoints.web.exposure.include=prometheus,health

```

打开`[src/main/resources/import.sql](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/resources/import.sql)`找到一些 SQL，这些 SQL 会用一组初始数据预先填充数据库表:

```
INSERT INTO todo(id, title, completed) VALUES (0, 'My first todo', 'true');

```

对于一个功能齐全的应用程序来说，这里没有太多的代码，只是“Hello World！”

## 魔力

这种迁移的一个硬性要求是:应用程序的源代码不能以任何方式修改。

你准备好变魔术了吗？返回命令行并运行命令`./.mvnw clean spring-boot:run`。如果你想用 Gradle 代替 Maven，首先切换到`gradle`分支(`git checkout gradle`)并运行命令`./.gradlew clean bootRun`。

你首先会注意到的是 Quarkus 的横幅和创业信息，而不是 Spring Boot 的:

```
__  ____  __  _____   ___  __ ____  ______
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/
INFO  [io.quarkus] (Quarkus Main Thread) todo-spring-quarkus 0.0.1-SNAPSHOT on JVM (powered by Quarkus 2.0.0.Final) started in 2.748s. Listening on: http://localhost:8080
INFO  [io.quarkus] (Quarkus Main Thread) Profile dev activated. Live Coding activated.
INFO  [io.quarkus] (Quarkus Main Thread) Installed features: [agroal, cdi, hibernate-orm, hibernate-orm-panache, jdbc-postgresql, kubernetes, micrometer, narayana-jta, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-openapi, spring-data-jpa, spring-di, spring-web, swagger-ui]

```

等等，刚刚发生了什么？该应用程序现在是 Quarkus 应用程序，而不再是 Spring Boot 应用程序？太疯狂了！

想要证据吗？回到你的浏览器窗口( [http://localhost:8080](http://localhost:8080) 以防你关闭它)并重新加载页面。同样的用户界面是存在的，并且是完全有效的。点击页面底部的所有链接。它们和以前一样功能齐全。

另外，请注意启动时间。给定完全相同的代码库，Quarkus 版本的启动时间几乎是一半(在所示的示例中，Spring 为 4.392 秒，Quarkus 为 2.748 秒)。那是超音速，亚原子，Java！

什么都没有改变，那么这一切是怎么发生的呢？一个好的魔术师不会泄露他或她的秘密！

## 疯狂

所有优秀的魔术师在表演魔术时都会使用[手法](https://en.wikipedia.org/wiki/Sleight_of_hand)来分散观众的注意力。有一些肉眼看不到的东西在这篇文章中没有显示出来。你注意到什么可疑的事情了吗？让我们仔细看看这个戏法是怎么变的。

### 执行

仔细查看您用来运行应用程序的命令:

使用 Maven:

Spring Boot: `./mvnw clean spring-boot:run`

夸库斯:`./.mvnw clean spring-boot:run`

使用 Gradle:

Spring Boot: `./gradlew clean bootRun`

夸库斯:`./.gradlew clean bootRun`

注意到什么不同了吗？Quarkus 版本中使用了不同的可执行文件(`.mvnw` / `.gradlew`)。如果您打开这些文件并仔细检查，您会注意到一些诡计正在发生:

`[.mvnw](https://github.com/edeandrea/todo-spring-quarkus/blob/main/.mvnw)`

```
#!/bin/sh
./mvnw clean quarkus:dev -Pquarkus

```

`[.gradlew](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/.gradlew)`

```
#!/bin/sh
./gradlew -Pprofile=quarkus $@

```

我们使用了一些技巧来掩饰用于启动应用程序的实际命令。稍后更多关于 Gradle 和 Maven 的简介。

### 配置

你是说夸库斯知道如何阅读和理解`src/main/resources/application.properties`里面的 Spring Boot 配置吗？嗯，不。记住，这是一个魔术。魔术师永远不会说出完全的真相。上例中没有显示的是 Quarkus 特有的配置。它被藏在文件的更深处。

如果您重新打开`[src/main/resources/application.properties](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/resources/application.properties)`并一直滚动到底部(围绕第 58 行),您会看到一些额外的配置:

```
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/tododb
quarkus.datasource.username=todo
quarkus.datasource.password=todo
quarkus.datasource.metrics.enabled=true
quarkus.hibernate-orm.database.generation=drop-and-create
quarkus.hibernate-orm.sql-load-script=import.sql

quarkus.swagger-ui.always-include=true
quarkus.swagger-ui.path=/swagger-ui
quarkus.micrometer.export.prometheus.path=/actuator/prometheus
quarkus.smallrye-health.root-path=/actuator/health
quarkus.smallrye-openapi.path=/openapi

```

该配置类似于文件顶部的 Spring Boot 配置。但是，它做的一件事是重新定义 Prometheus 和 health probe 端点的路径，使它们与 Spring Boot 执行器端点的路径相匹配。这使得屏幕底部的 **Prometheus Metrics** 和 **Health Check** 链接可以在不改变用户界面的情况下工作。

很狡猾，是吧？

### 属国

可以想象，有许多依赖项需要更改、添加或更新。幸运的是，Quarkus 提供了许多 Spring 兼容性扩展:

*   [弹簧依赖注入的 Quarkus 扩展](https://quarkus.io/guides/spring-di)
*   [弹簧腹板的 Quarkus 延伸](https://quarkus.io/guides/spring-web)
*   [Spring 数据 JPA 的 Quarkus 扩展](https://quarkus.io/guides/spring-data-jpa)
*   [Spring Boot 性质的夸库斯扩展](https://quarkus.io/guides/spring-boot-properties)

简单地用夸库人来交换一些 Spring Boot 人的附属品会有很大的帮助。还有一些其他功能，如 Prometheus metrics、OpenAPI 文档、Swagger UI 集成和健康检查，需要添加其他依赖项。幸运的是，Quarkus 也有这些能力:

*   [用于 OpenAPI 和 Swagger UI 的 Quarkus 扩展](https://quarkus.io/guides/openapi-swaggerui)
*   [用于微配置文件健康的 Quarkus 扩展](https://quarkus.io/guides/microprofile-health)
*   [千分尺的 Quarkus 扩展](https://quarkus.io/guides/micrometer)

现在你可能在想，“但是我没有做任何改变。我所做的只是运行一个 Maven 或 Gradle 命令，整个应用程序作为一个 Quarkus 应用程序而不是一个 Spring Boot 应用程序运行。这怎么可能呢？”

### 构建设置

构建文件是所有奇迹发生的地方。但是，您使用的构建工具决定了依赖关系解析魔法实际上是如何发生的。

#### 专家

在`main`分支上，打开 [`pom.xml`](https://github.com/edeandrea/todo-spring-quarkus/blob/main/pom.xml) 文件。您会立即注意到构建文件被分成多个 [Maven 概要文件](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)、`[spring](https://github.com/edeandrea/todo-spring-quarkus/blob/main/pom.xml#L26-L99)`和`[quarkus](https://github.com/edeandrea/todo-spring-quarkus/blob/main/pom.xml#L100-L195)`，其中`spring`概要文件是默认概要文件。这些概要文件定义了当您运行`./mvnw clean spring-boot:run`或`./mvnw -Pquarkus clean quarkus:dev`(在前面提到的`.mvnw`脚本中使用一些重定向来伪装)时，包含哪些依赖项和构建插件。

#### Gradle

另一方面，Gradle 没有侧写的概念。您会注意到项目的`gradle`分支中有一些`.gradle`文件:

*   `[build-common.gradle](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/build-common.gradle)`
    *   包含适用于应用程序两个版本的任何公共构建逻辑，包括设置所生产工件的 groupId/version 以及工件存储库定义。
*   `[build-spring.gradle](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/build-spring.gradle)`
    *   包含所有特定于应用程序 Spring 版本的构建逻辑、插件和依赖项。
*   `[build-quarkus.gradle](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/build-quarkus.gradle)`
    *   包含所有特定于 Quarkus 版本应用程序的构建逻辑、插件和依赖项。`build-quarkus.gradle`甚至引入了 [Spring Boot 的`bootRun`](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#running-your-application) 任务，从 [Quarkus Gradle 插件重新映射到](https://quarkus.io/guides/gradle-tooling)`[quarkusDev](https://quarkus.io/guides/gradle-tooling#development-mode)`任务。
*   `[settings.gradle](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/settings.gradle)`
    *   奇迹真正发生的地方！`settings.gradle`文件查找名为`profile`的 [Gradle 项目属性](https://docs.gradle.org/current/userguide/build_environment.html#sec:project_properties):
        *   如果找不到这个属性，它将默认值设为`spring`。
        *   然后，它将项目的构建文件`build-spring.gradle`或`build-quarkus.gradle`设置为要使用的主构建文件。

一般来说，如果您需要 Gradle 来模拟 Maven 概要文件的功能，这是一个很好的模式。

### 应用程序主类

在这篇文章的开头提到，我们希望在不改变一行代码的情况下执行迁移。每个 Spring Boot 应用程序都需要有一个“应用程序”类，该类包含一个`main`方法，并用`@SpringBootApplication`进行注释。在我们的项目中，`[src/main/java/io/quarkus/todospringquarkus/TodoApplication.java](https://github.com/edeandrea/todo-spring-quarkus/blob/main/src/main/java/io/quarkus/todospringquarkus/TodoApplication.java)`就是那个类。

Quarkus 不需要这样的类，任何 Quarkus Spring 兼容性扩展都不为这个类中引用的`@SpringBootApplication`注释和`SpringApplication`类提供解析。

那么，怎么回事？我们没有做任何代码修改，但是这些类似乎在 Quarkus 中解决得很好。

您会注意到在`[pom.xml](https://github.com/edeandrea/todo-spring-quarkus/blob/main/pom.xml#L16)`(对于 Maven)/ `[build-quarkus.gradle](https://github.com/edeandrea/todo-spring-quarkus/blob/gradle/build-quarkus.gradle#L23)`(对于 Gradle)中有一个特殊的注释，就在依赖关系`org.springframework.boot:spring-boot-autoconfigure`的依赖关系声明的上方:

```
This dependency is a hack for TodoApplication.java, which isn't required for Quarkus. Point of demo is to NOT have any code changes.

```

这是这部分魔术的关键。这种依赖性允许 Spring Boot 和夸尔库斯在构建时解析这些类。依赖关系在 Maven 中声明为`[optional](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html#optional-dependencies)`,在 Gradle 中声明为`[compileOnly](https://blog.gradle.org/introducing-compile-only-dependencies)`,这意味着它永远不会包含在 Quarkus 构建生成的应用程序二进制文件中。它将被包含在 Spring Boot 构建生成的二进制文件中，因为所有其他`spring-boot-starter-*`依赖项也依赖于它，所以它被包含在传递文件中。

## 包裹

在这篇文章中，您看到了如何在 Quarkus 上运行一个现有的 Spring Boot 应用程序，而无需对代码做任何改动。这个方法是魔法还是疯狂？也许两者都有一点？由你自己决定。

这篇文章展示了一种将一个不仅仅是“Hello World”的应用程序从 Spring Boot 迁移到 Quarkus 的方法，并且严格要求不要修改任何一行源代码。它并不代表唯一的潜在迁移途径。此外，有时应用程序可能会使用一些没有 Quarkus 等价物的库或 API。在这些情况下，可能需要更改代码或进行一些必要的重构。

非常感谢埃里克·墨菲。他是该应用程序的原始作者，并提出了魔术的想法。

## 参考

这篇文章的所有源代码可以在[这里](http://github.com/edeandrea/todo-spring-quarkus)找到。`main`分支包含 Maven 版本，而`gradle`分支包含 Gradle 版本。两个分支上的应用程序源代码是相同的。

*Last updated: October 7, 2022*