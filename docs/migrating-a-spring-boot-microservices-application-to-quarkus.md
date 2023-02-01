# 将 spring boot 微服务应用迁移到 quartus

> 原文：<https://developers.redhat.com/blog/2020/04/10/migrating-a-spring-boot-microservices-application-to-quarkus>

虽然 Spring Boot 长期以来一直是用 Java 开发基于容器的应用程序的事实上的框架，但是 Kubernetes 原生框架的性能优势是难以忽视的。在本文中，我将向您展示如何快速将 Spring Boot 微服务应用程序迁移到 [Quarkus](https://developers.redhat.com/topics/quarkus/) 。迁移完成后，我们将测试应用程序，并比较原始 Spring Boot 应用程序和新 Quarkus 应用程序的启动时间。

注意:对于有兴趣从 Spring Boot 迁移到 Quarkus 的开发者来说，知道 Quarkus 不支持 Spring Boot 的所有扩展和特性是很重要的。举个例子，它只支持 Java EE [上下文和依赖注入(CDI) API](https://quarkus.io/guides/cdi-reference) 的子集。将微服务或基于容器的应用程序迁移到 Quarkus 将比迁移整体应用程序更容易。

## 关于夸特斯

Quarkus 是一个为 Java 虚拟机(JVM)如 GraalVM 和 HotSpot 定制的 Kubernetes-native Java 框架。作为 T2 的 Kubernetes 本地人，意味着 Quarkus 在 Java 应用程序开发中采用了 T4 的容器优先方法。容器优先开发中固有的较小内存占用使得 Quarkus 成为当今在 Kubernetes 和无服务器平台上运行 Java 应用程序的最佳选择之一。

## Spring Boot 应用程序

对于我们的示例应用程序，我们将使用我上一篇文章 *[中的`AccountBalance`微服务应用程序，使用 Red Hat AMQ 流](https://developers.redhat.com/blog/2019/11/21/event-based-microservices-with-red-hat-amq-streams/)* 的基于事件的微服务。`AccountBalance`服务有自己的 MongoDB 数据库，保存账户余额信息。数据库也被其他服务调用，比如`EventCorrelator`服务。

您可以在 GitHub 上找到示例应用程序的源代码。我将指导您将这个应用程序从 Spring Boot 迁移到 Quarkus 的每一步。

下面显示了示例源代码的内容。这是典型的标准 Java 项目文件结构。mvnw 是我们[生成](https://www.baeldung.com/maven-wrapper)的 Maven wrapper 插件。我们需要修改 pom.xml 和 src/main 下的源代码。

[![Sample Codes Content](img/2b5b7b0d812a29ed674f5ebaa90953f5.png "Sample Codes Content")](/sites/default/files/blog/2020/04/Quarkus-Codes-Structure.png)Sample Codes Content

img class=

样本代码内容">

### 步骤 1:为您的应用程序修改 pom.xml

从 Spring Boot 迁移到 Quarkus 最简单的方法是[引导一个示例 Quarkus 应用程序](https://quarkus.io/guides/getting-started#bootstrapping-the-project)，并使用该应用程序的`pom.xml`作为模板来修改您的 Spring Boot 应用程序中的同一个文件。

**注意**:除了示例`pom.xml`，Quarkus 团队还提供了一个[基于 web 的用户界面(UI)](https://code.quarkus.io/) ，您可以使用它进行迁移。在本文中，我不会演示如何使用 web UI。

按照[引导指令](https://quarkus.io/guides/getting-started#bootstrapping-the-project)中的描述，花一分钟来设置你的终端和项目。一旦你建立了你的项目，我们就可以开始修改 Spring Boot `pom.xml`。我们将从移除我们不再需要的 Spring Boot 配置开始，然后我们将用 Quarkus 应用程序的相应[配置替换那些元素。](https://quarkus.io/guides/config)

#### 删除 Spring Boot 配置

首先，我们从示例应用程序的`pom.xml`中移除打包配置。这不再需要，因为它由`pom.xml`配置中的`<build>`部分负责。

```
<!--- Remove the packing configuration -->

<packaging>jar</packaging>

```

`spring-boot-starter-parent`用于此处不再需要的 Spring Boot 应用程序。

```
<!--- Remove spring-boot-starter-parent -->

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.2.RELEASE</version>
</parent>

```

同样适用于`spring-cloud-dependencies`:

```
<!-- Remove the following from the <dependencyManagement> -->

<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>Brixton.SR4</version>

```

现在，我们可以删除所有剩余的 Spring Boot 依赖项:

```
<!-- Remove all Spring Boot-related dependencies -->

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-mongodb</artifactId>
 </dependency>
 <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-commons</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-messaging</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-context</artifactId>
</dependency>
<dependency>
  <groupId>javax.ws.rs</groupId>
  <artifactId>javax.ws.rs-api</artifactId>
  <version>2.0.1</version>
</dependency>
<dependency>
  <groupId>org.apache.maven</groupId>
  <artifactId>maven-model</artifactId>
  <version>3.3.9</version>
</dependency>
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.2.11</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.bind</groupId>
  <artifactId>jaxb-core</artifactId>
  <version>2.2.11</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.bind</groupId>
  <artifactId>jaxb-impl</artifactId>
  <version>2.2.11</version>
</dependency>
<dependency>
  <groupId>javax.activation</groupId>
  <artifactId>activation</artifactId>
  <version>1.1.1</version>
</dependency>

```

我们也可以去掉构建部分中的`spring-boot-maven-plugin`:

```
<!-- Remove the spring-boot-maven plugin from the build section -->

<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>

```

#### 向 pom.xml 添加 Quarkus 元素

接下来，我们将从自举的 Quarkus `pom.xml`中复制属性和依赖项，并将其粘贴到我们的示例应用程序的`pom.xml`的顶部。这一部分很重要，它告诉编译器我们希望 Quarkus 使用哪个版本的组件。Quarkus 的发布进展很快，你可能想查看一下`<quarkus.platform.version>` 的最新版本

```
<!-- Place this at the top of the pom.xml -->

<properties>
  <compiler-plugin.version>3.8.1</compiler-plugin.version>
  <maven.compiler.parameters>true</maven.compiler.parameters>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  <quarkus-plugin.version>1.2.1.Final</quarkus-plugin.version>
  <quarkus.platform.artifact-id>quarkus-universe-bom</quarkus.platform.artifact-id>
  <quarkus.platform.group-id>io.quarkus</quarkus.platform.group-id>
  <quarkus.platform.version>1.2.1.Final</quarkus.platform.version>
  <surefire-plugin.version>2.22.1</surefire-plugin.version>
</properties>

```

将此添加到`dependencyManagement`下:

```
<!-- Add this under dependencyManagement -->

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>${quarkus.platform.group-id}</groupId>
      <artifactId>${quarkus.platform.artifact-id}</artifactId>
      <version>${quarkus.platform.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

```

现在我们可以从引导文件中复制 Quarkus 依赖项，并将它们粘贴到`pom.xml`中。注意，在下面的代码中，我还手动添加了示例应用程序所需的 [MongoDB Panache](https://quarkus.io/guides/mongodb-panache) 依赖项。我手动添加了这个依赖项，因为我之前没有使用 web UI 来引导它。还要注意的是，`AccountBalance`使用一个存储库来连接和查询 MongoDB 数据库。Panache 扩展允许我们以最小的改动来移植现有的代码。(你可以在 GitHub 上找到 Panache 的[源代码。)](https://github.com/quarkusio/quarkus/blob/master/extensions/panache/mongodb-panache/runtime/src/main/java/io/quarkus/mongodb/panache/PanacheMongoRepositoryBase.java)

首先将这些依赖项添加到`pom.xml:`

```
<dependency>
  <groupId>io.quarkus</groupId>

  <!-- notice this app is using jsonb -->

  <artifactId>quarkus-resteasy-jsonb</artifactId>
</dependency>
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-junit5</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.rest-assured</groupId>
  <artifactId>rest-assured</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-mongodb-panache</artifactId>
</dependency>

```

在构建配置下添加`quarkus-maven-plugin`:

```
<!-- Add the following plugin under the build configuration -->

<build>
  <finalName>${project.artifactId}-${project.version}</finalName>
  <plugins>
    <plugin>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-maven-plugin</artifactId>
      <version>${quarkus-plugin.version}</version>
      <executions>
        <execution>
          <goals>
            <goal>build</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
        <version>${compiler-plugin.version}</version>
    </plugin>
    <plugin>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>${surefire-plugin.version}</version>
      <configuration>
        <systemProperties>
          <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
        </systemProperties>
      </configuration>
    </plugin>
  </plugins>
</build>

```

最后，添加配置文件设置:

```
<!-- Add the following profiles settings -->

<profiles>
  <profile>
    <id>native</id>
    <activation>
      <property>
        <name>native</name>
      </property>
    </activation>
    <build>
      <plugins>
        <plugin>
          <artifactId>maven-failsafe-plugin</artifactId>
            <version>${surefire-plugin.version}</version>
            <executions>
              <execution>
                <goals>
                  <goal>integration-test</goal>
                  <goal>verify</goal>
                </goals>
                <configuration>
                  <systemProperties>
                    <native.image.path>${project.build.directory}/${project.build.finalName}-runner</native.image.path>
                  </systemProperties>
                </configuration>
              </execution>
            </executions>
          </plugin>
        </plugins>
     </build>
     <properties>
       <quarkus.package.type>native</quarkus.package.type>
     </properties>
  </profile>
</profiles>

```

### 步骤 2:迁移 Spring Boot 应用程序代码

Maven POM 已经设置好了。我们现在准备将 Spring Boot 应用程序代码迁移到 Quarkus。

首先，删除`Application.java`类。我们不再需要它了。您还可以修改应用程序端口的`application.properties`和 [MongoDB 客户端属性](https://quarkus.io/guides/mongodb#configuration-reference)文件，如下所示:

```
### --- Remove the Spring Boot properties
# spring.application.name=accountbalance-service
# spring.data.mongodb.host=localhost
# spring.data.mongodb.port=27017
# spring.data.mongodb.username=checkbalance
# spring.data.mongodb.password=checkbalance
# spring.data.mongodb.database=checkbalance
# server.port=8082
### --- Replace with equivalent Quarkus MongoDB properties
quarkus.mongodb.connection-string=mongodb://checkbalance:checkbalance@localhost:27017/checkbalance
# --- Note the wrong context of database name ... planning and design before coding is so important.
quarkus.mongodb.database=checkbalance
quarkus.http.port=8082

```

#### 实体 bean 代码

接下来，我们将更改`Balance.java`中的实体 bean 代码。首先，删除与 Spring Boot 相关的 import 语句。在下一节中，我们将用 Quarkus 等效元件替换这些元件。

```
/// --- Remove the Spring Boot-related import statements

import org.bson.types.ObjectId;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

```

为 Quarkus 添加等效的导入语句。Quarkus 框架通过 MongoDB Panache 扩展为 MongoDB 提供了自己支持的组件。Quarkus 还提供了自己版本的 CDI 注释。在这里，我们为它们提供正确的导入语句。

```
/// --- Add the following Quarkus-related import statements

import io.quarkus.mongodb.panache.MongoEntity;
import io.quarkus.mongodb.panache.PanacheMongoEntity;
import org.bson.codecs.pojo.annotations.BsonProperty;
import org.bson.codecs.pojo.annotations.BsonId;
import io.quarkus.mongodb.panache.PanacheQuery;

```

用`@MongoEntity`替换现有的`@Document`注释:

```
/// --- Replace @Document annotation with @MongoEntity

// --- @Document(collection = "balance") <--- Remove this
@MongoEntity(collection="balance")
public class Balance extends PanacheMongoEntity{
/// --- More codes here are omitted ...
...
```

用夸尔库斯的`@BsonId`替换 Spring Boot 的`@Id`注释:

```
/// --- Replace @Id annotation with @BsonId

// @Id <--- Remove this
@BsonId  //In fact, we do not need this. Just leave it here for now.
private String _id;

```

### 存储库类

我们还需要修改 Spring Boot 存储库类，`BalanceRepository.java`。首先，删除以下 import 语句:

```
/// --- Remove the following import statement

import org.springframework.data.mongodb.repository.MongoRepository;

```

用这个替换它:

```
/// --- Add the following import statements

import io.quarkus.mongodb.panache.PanacheMongoRepository;
import javax.enterprise.context.ApplicationScoped;
import io.quarkus.mongodb.panache.PanacheQuery;

```

然后再做三个快速的改变:

```
/// --- Change the BalanceRepository to implement PanacheMongoRepository
// --- Change the BalanceRepository from interface to class
// --- Drop in the @ApplicationScoped annotation

@ApplicationScoped
public class BalanceRepository implements PanacheMongoRepository<Balance> {
   // --- Change the findByAccountId(String accountId) to the following implementation
   public Balance findByAccountId(String accountId){
      return find("accountId", accountId).firstResult();
   }
}

```

#### 休息类

我们还将修改`AccountBalance.java`。首先删除所有与 Spring Boot 相关的进口声明:

```
/// --- Remove all the Spring Boot-related import statements

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.core.env.Environment;
import org.springframework.data.domain.Example;
import org.springframework.data.repository.Repository;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.beans.factory.annotation.Autowired;

```

为 Quarkus 添加以下导入语句。(我们没有在`.java`文件中使用所有这些导入，但是我们现在可以把它们留下。)

```
/// --- Add the following import statements.

import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.MediaType;
import javax.inject.Inject;

```

移除 Spring Boot `@RequestMapping`和`@RestController`标注:

```
/// --- Remove the @RequestMapping and @RestController and
update as follows

// @RequestMapping("/ws/pg")      <--- Remove this
// @RestController      <--- Remove this
@Path("/ws/pg")         // <---- Add this
@Produces(MediaType.APPLICATION_JSON)         // <---- Add this
@Consumes(MediaType.APPLICATION_JSON)        // <---- Add this
public class AccountBalance{
/// --- I have omitted more code here
...

```

您还需要删除`@Autowire`和`@Inject`注释:

```
/// --- Remove @Autowire with @Inject

// @Autowired      <--- Remove this
@Inject
private BalanceRepository repository;

```

请注意，我们将`@RequestMapping`注释改为了`@Path`，并且我们还删除了`@ResponseBody`。作为我们的最后一步，我们需要将`@PathVariable`改为`@PathParam`，如下所示。(虽然没有显示，但我们将对这个 Java 类中的所有其他方法应用类似的更改。查看示例应用程序的源代码以查看所有的更改):

```
@Path("/balance/{accountid}")
@GET
public Balance get(@PathParam("accountid") String accountId) {
   Balanceresult = repository.findByAccountId(accountId);
   return result;
}

```

这就完成了迁移，尽管为了使这个练习简短，我省略了一些步骤。作为一个例子，`AccountBalance`应用程序应该被部署到 [Red Hat OpenShift](https://www.openshift.com/) 上，因此我修改了`Heathz.java`文件。您可以在应用程序源代码中查看这些更改。

### 步骤 3:测试 Quarkus 应用程序

接下来，我们将测试我们的新应用程序。图 1 显示了迁移前的 Spring Boot 执行。

[![SpringBoot at command prompt](img/3d876a827d6b02f64b4949a57061e565.png "SpringBoot at command prompt")](/sites/default/files/blog/2020/03/Screenshot-2020-03-11-at-10.07.33-AM.png)SpringBoot at command prompt

图一。Spring Boot 应用程序在 3.166 秒内启动。">

让我们看看 Quarkus 应用程序的启动与 Spring Boot 相比如何。在命令提示符下，切换到`AccountBalance`目录。执行以下命令运行迁移的应用程序:

```
mvn quarkus:dev

```

请注意，第一次运行该应用程序时，可能会比后续运行花费更长的时间。Maven 需要额外的时间将 Quarkus 库下载到您的本地机器上。图 2 显示了新的 Quarkus 应用程序的开始时间。

[![A screenshot of the Quarkus application's start time on first execution.](img/a86e409665398cd869b9a83a58545592.png "Quarkus execution")](/sites/default/files/blog/2020/03/Screenshot-2020-03-11-at-10.14.18-AM.png)Quarkus execution

图二。夸尔库斯在 1.540 秒内开始。">

这是一个进步。现在让我们做一个[夸尔库斯-原生建造](https://quarkus.io/guides/building-native-image)。记得从项目的`root`目录执行本地构建:

```
/mvnw package -Pnative

```

图 3 中的输出表明一个成功的 Quarkus-native 构建。

[![Quarkus native build completed sucessfully](img/110bea154d46fa7920c1a120cf50b0ca.png "Quarkus native build completed sucessfully")](/sites/default/files/blog/2020/03/Screenshot-2020-03-11-at-10.20.18-AM.png)Quarkus native build completed sucessfully

图 3。Quarkus-native 构建成功完成。">

输入以下内容运行本机二进制文件:

```
./target/account-service-1.0.0-runner

```

图 4 显示了 Quarkus-native 构建的执行时间。

[![A screenshot of the execution time for the Quarkus native build.](img/6961509f6ca24c626486a168afdfbbb3.png "Quarkus native execution")](/sites/default/files/blog/2020/03/Screenshot-2020-03-11-at-10.22.49-AM.png)Quarkus native execution

图 4。Quarkus-native 构建在 0.071 秒内开始。">

最后，我创建了一个简单的 UI 来调用`AccountBalance`服务并显示账户余额。如图 5 所示，迁移后的应用程序按预期工作。

[![An account balance is displayed in the AccountBalance app's UI.](img/14cdb9a2a24103ddf7118b4cef23fcc4.png "The account balance is shown on the UI apps")](/sites/default/files/blog/2020/03/Screenshot-2020-03-11-at-10.18.06-AM.png)The account balance is shown on the UI apps

图 5。AccountBalance 应用程序正常工作。">

从 Spring Boot 到夸尔库斯的迁移是成功的。

## 结论

将示例应用程序从 Spring Boot 迁移到 Quarkus 是一项非常简单的工作。对于更复杂的应用程序，迁移路径会更复杂。将我的微服务应用从 Spring Boot 迁移到 Quarkus 的最大挑战是决定使用哪些注释和库。Quarkus 的文档提供了很好的例子，但是省略了 import 语句。许多 ide 会为您处理那些配置细节，但是我选择手动迁移。

我希望这篇文章可以作为将您的 Java 应用程序从 Spring Boot 迁移到 Quarkus 的起点。如果你想把你学到的东西带到下一个层次，你可以[按照这些步骤来封装你的应用程序](https://quarkus.io/guides/building-native-image#creating-a-container)。

## 资源

查看以下资源以了解更多信息:

*   在 GitHub 上找到示例应用程序的完整源代码。
*   参见 [Quarkus 配置指南](https://quarkus.io/guides/config)，了解如何使用 Quarkus `ConfigProperty`、`application.properties`等配置您的应用。
*   了解更多关于 MongoDB Panache 扩展的信息。另外，请参见 GitHub 上的[PanacheMongoRepositoryBase](https://github.com/quarkusio/quarkus/blob/master/extensions/panache/mongodb-panache/runtime/src/main/java/io/quarkus/mongodb/panache/PanacheMongoRepositoryBase.java)库——阅读源代码是开发人员应该做的事情！
*   获取更多关于[构建本地 Quarkus 可执行文件](https://quarkus.io/guides/building-native-image)的信息。

*Last updated: April 22, 2022*