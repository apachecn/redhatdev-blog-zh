# 在 Red Hat OpenShift 上集成系统与 Apache Camel 和 Quarkus

> 原文：<https://developers.redhat.com/articles/2021/05/17/integrating-systems-apache-camel-and-quarkus-red-hat-openshift>

十多年来，Apache Camel 一直是集成异构系统的一个非常成功的工具。您可能遇到过这样一种情况，您有两个系统，它们不是为相互通信而设计的，但是仍然需要交换数据。这正是 Camel 及其集成管道可以提供帮助的情况。这篇文章展示了[夸尔库斯](/products/quarkus/getting-started)如何使用骆驼。

图 1 显示了 Camel 的基本操作:来自一个系统的数据通过为该系统设计的传输传递到为接收系统设计的传输。

[![Diagram shows data passing from System A through a System A Transport, and data passing from System B through a System B Transport. The two systems are connected via a Route.](img/9a6e6535e943a8c568ef97c4af956e6e.png)](/sites/default/files/blog/2021/05/system1-system2.png)

Figure 1: A basic Camel integration route.

传统上，Camel 集成已经部署在各种 [Java](/topics/enterprise-java) 平台上，比如 [Spring Boot](/topics/spring-boot) ，Karaf，以及[红帽 JBoss 企业应用平台](/products/eap/overview) (JBoss EAP)。多亏了在 Camel Quarkus 社区项目中所做的工作，现在可以在 Quarkus 上运行 Camel 集成了。主要的好处是集成启动更快，消耗的内存更少。但是 Quarkus `dev`模式也带来了开发者生产力的好处。不要忘记[集装箱优先](/topics/containers)的理念。

本文展示了如何在 [Red Hat OpenShift](/products/openshift/overview) 上使用 Camel 和 Quarkus。

## 定义骆驼路线

让我们浏览一下 Camel 集成的所有部分，并通过一个例子更详细地了解它们。示例的源代码是 GitHub 上提供的[。](https://github.com/jboss-fuse/camel-quarkus-examples/tree/camel-quarkus-examples-1.6.0-product/file-bindy-ftp)

假设您需要处理外部系统存储在本地目录中的 CSV 文件。这些文件包含由书名、作者名和流派组成的图书记录。您希望将唱片按流派分割成单独的 CSV 文件，并将它们存储在远程 SFTP 服务器上。

为了完成这个项目，定义一些骆驼路线。第一种方法是辅助性的，模拟生成包含图书数据的 CSV 文件的外部系统:

```
import org.apache.camel.Exchange;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.model.dataformat.BindyType;
import org.apache.camel.processor.aggregate.GroupedBodyAggregationStrategy;

public class Routes extends RouteBuilder {

    @Override
    public void configure() throws Exception {
        // Route 1: Generate some book objects with random data
        from("timer:generateBooks?period={{timer.period}}&delay={{timer.delay}}")
                .log("Generating randomized books CSV data")
                .process("bookGenerator")
                // Marshal each book to CSV format
                .marshal().bindy(BindyType.Csv, Book.class)
                // Write CSV data to file
                .to("file:{{csv.location}}");

        // More route definitions come here...

    }
}
```

第二条路线使用 [Camel Bindy](https://camel.apache.org/components/latest/dataformats/bindy-dataformat.html) 将发现的 CSV 文件解析成一个普通旧 Java 对象(POJOs)列表。这个列表通过`split(body())`被分割成单个的`Book`对象，并被传递给另一个叫做`direct:aggregateBooks`的 Camel 端点:

```
 ...

        // Route 2: Consume book CSV files
        from("file:{{csv.location}}?delay=1000")
                .log("Reading books CSV data from ${header.CamelFileName}")
                .unmarshal().bindy(BindyType.Csv, Book.class)
                .split(body())
                .to("direct:aggregateBooks");

        // More route definitions come here...
```

第三条路线选择前一条路线产生的`Book`对象，进行聚合(产生每种风格的`Book`列表)，并将它们传递给最后一条路线:

```
 ...

        // Route 3: Aggregate books based on their genre
        from("direct:aggregateBooks")
                .setHeader("BookGenre", simple("${body.genre}"))
                .aggregate(simple("${body.genre}"), new GroupedBodyAggregationStrategy()).completionInterval(5000)
                .log("Processed ${header.CamelAggregatedSize} books for genre '${header.BookGenre}'")
                .to("seda:processed");

        // One more route definition comes here...
```

第四种方法是将`Book`记录的聚合列表序列化回 CSV，并将它们存储在远程 SFTP 服务器上:

```
 ...

        // Route 4: Marshal books back to CSV format
        from("seda:processed")
                .marshal().bindy(BindyType.Csv, Book.class)
                .setHeader(Exchange.FILE_NAME, simple("books-${header.BookGenre}-${exchangeId}.csv"))
                // Send aggregated book genre CSV files to an FTP host
                .to("sftp://{{ftp.username}}@{{ftp.host}}:{{ftp.port}}/uploads/books?password={{ftp.password}}")
                .log("Uploaded ${header.CamelFileName}");
```

## pom.xml 文件

对于一个简单的 Camel 项目，您必须列出 Camel 组件工件(`camel-bindy`、`camel-seda`等)。)作为您的`pom.xml`文件中的依赖项。对于一个骆驼夸库项目，你必须列出他们的骆驼夸库对应项目(`camel-quarkus-bindy`、`camel-quarkus-seda`等)。).导入`org.apache.camel.quarkus:camel-quarkus-bom`来管理依赖项的版本:

```
<project>
    ...

    <properties>
        <camel-quarkus.version>1.6.0.fuse-jdk11-800006-redhat-00001</camel-quarkus.version>
        <quarkus.version>1.11.6.Final-redhat-00001</quarkus.version>
        ...
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- Import BOM -->
            <dependency>
                <groupId>org.apache.camel.quarkus</groupId>
                <artifactId>camel-quarkus-bom</artifactId>
                <version>${camel-quarkus.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.apache.camel.quarkus</groupId>
            <artifactId>camel-quarkus-bean</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.camel.quarkus</groupId>
            <artifactId>camel-quarkus-bindy</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.camel.quarkus</groupId>
            <artifactId>camel-quarkus-direct</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.camel.quarkus</groupId>
            <artifactId>camel-quarkus-file</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.camel.quarkus</groupId>
            <artifactId>camel-quarkus-ftp</artifactId>
        </dependency>

        ...

    </dependencies>

    ...
</project>
```

## 运行时先决条件

要运行该示例，您需要一台 SFTP 服务器。对于测试，您可以使用 Docker 容器，如下所示:

```
$ docker run -ti --rm -p 2222:2222 \
-e PASSWORD_ACCESS=true \
-e USER_NAME=ftpuser \
-e USER_PASSWORD=ftppassword \
-e DOCKER_MODS=linuxserver/mods:openssh-server-openssh-client \
linuxserver/openssh-server
```

## Quarkus 开发模式

一切就绪后，您可以构建项目并以`dev`模式启动 Quarkus:

```
$ mvn clean compile quarkus:dev
```

这种模式让 Quarkus 工具监视工作区中的变化，并在发生任何变化时重新编译和重新部署应用程序。更多详情请参考 Camel Quarkus 用户指南的[开发模式部分。](https://camel.apache.org/camel-quarkus/latest/first-steps.html#_development_mode)

您应该开始看到控制台上出现日志消息，如下所示:

```
[route1] (Camel (camel-1) thread #3 - timer://generateBooks) Generating randomized books CSV data
[route2] (Camel (camel-1) thread #1 - file:///tmp/books) Reading books CSV data from 89A0EE24CB03A69-0000000000000000
[route3] (Camel (camel-1) thread #0 - AggregateTimeoutChecker) Processed 34 books for genre 'Action'
[route3] (Camel (camel-1) thread #0 - AggregateTimeoutChecker) Processed 31 books for genre 'Crime'
[route3] (Camel (camel-1) thread #0 - AggregateTimeoutChecker) Processed 35 books for genre 'Horror'
```

你可以试着在`Routes.java`中改变一些东西，并看到应用程序在保存文件后被实时重新加载。

## 打包和运行应用程序

完成开发后，您可以打包并运行应用程序:

```
$ mvn clean package -DskipTests
$ java -jar target/*-runner.jar
```

## 部署到 OpenShift

要将应用程序部署到 OpenShift，请运行以下命令:

```
$ mvn clean package -DskipTests -Dquarkus.kubernetes.deploy=true
```

检查 pod 是否正在运行:

```
$ oc get pods

NAME READY STATUS RESTARTS AGE
camel-quarkus-examples-file-bindy-ftp-5d48f4d85c-sjl8k 1/1 Running 0 21s
ssh-server-deployment-5c667bccfc-52xfz 1/1 Running 0 21s
```

跟踪应用程序日志，您应该会看到类似于`dev`模式中的消息:

```
$ oc logs -f camel-quarkus-examples-file-bindy-ftp-5d48f4d85c-sjl8k
```

## 关于 camel quartus 技术预览

Camel Quarkus 在[红帽集成 2021 Q2](/integration) 中作为技术预览(TP)组件提供。技术预览功能提供了对即将推出的产品创新的早期访问，使您能够在开发过程中测试功能并提供反馈。随着我们在今年晚些时候迈向全面上市(GA ),每个预览版都将关注关键用例。

此技术预览版中包含以下 Quarkus 扩展:

*   `camel-quarkus-bean`
*   `camel-quarkus-bindy`
*   `camel-quarkus-core`
*   `camel-quarkus-direct`
*   `camel-quarkus-file`
*   `camel-quarkus-ftp`
*   `camel-quarkus-log`
*   `camel-quarkus-main`
*   `camel-quarkus-microprofile-health`
*   `camel-quarkus-mock`
*   `camel-quarkus-seda`
*   `camel-quarkus-timer`

这些扩展仅在 JVM 模式下受支持。

注意，更多的 Camel Quarkus 扩展是由 Apache Camel 社区提供的。这些扩展可以与 Red Hat Integration 提供的扩展相结合。

更多详情，请参考[发行说明](https://access.redhat.com/documentation/en-us/red_hat_integration/2021.q2/html/release_notes_for_red_hat_integration_2021.q2/camel-quarkus-relnotes_integration)。

欢迎您通过 [Red Hat 支持门户](https://access.redhat.com/support)提供反馈。

*Last updated: August 15, 2022*