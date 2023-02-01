# 采用 OpenAPI 和 Red Hat Fuse 的 API 优先设计

> 原文：<https://developers.redhat.com/blog/2019/07/09/api-first-design-with-openapi-and-red-hat-fuse>

API 优先设计是一种常用的方法，在这种方法中，您在提供实际的实现之前为应用程序定义接口。这种方法给你带来很多好处。例如，在投入大量时间实现 API 之前，您可以测试您的 API 是否具有正确的结构，并且您可以提前与其他团队分享您的想法，以获得有价值的反馈。在这个过程的后期，后端开发中的延迟不会对前端开发人员造成太大的影响，因为从 API 定义创建服务的模拟实现很容易。

关于 API 优先设计的好处已经写了很多，所以本文将关注如何有效地获取一个 [OpenAPI](https://swagger.io/docs/specification/about/) 定义，并用 [Red Hat Fuse](https://developers.redhat.com/products/fuse) 将其带入代码。

假设已经设计了一个用于公开 beer API 的 API。正如你在描述 API 的 [JSON 文件中看到的，它是一个](https://github.com/microcks/api-lifecycle/blob/master/beer-catalog-demo/api-contracts/beer-catalog-api-swagger.json) [OpenAPI](https://www.openapis.org/) 定义，每个操作由一个 *operationId* 标识。在实际实现时，这将被证明是很方便的。这个 API 非常简单，由三个操作组成:

*   买啤酒——按名字买啤酒。
*   FindBeersByStatus—根据啤酒的状态查找啤酒。
*   list beers—获取数据库中的所有啤酒。

## 将生成的代码与实现分开

我们不想编写所有的 dto 和样板代码，因为那非常耗时而且琐碎。因此，我们将使用 [Camel REST DSL Swagger Maven 插件](https://github.com/apache/camel/blob/master/tooling/maven/camel-restdsl-swagger-plugin/src/main/docs/camel-restdsl-swagger-plugin.adoc)来生成所有这些内容。

我们希望将 swagger 插件生成的代码与我们的实现分开，原因有几个，包括:

*   代码生成消耗时间和资源。将代码生成与编译分开，可以让我们花更少的时间等待，从而有更多的时间与同事一起喝咖啡，以各种方式发挥创造力。
*   我们不必担心开发人员会不小心将一些实现内容放入自动生成的类中，从而在下次重新生成存根时丢失有价值的工作。当然，我们的一切都在版本控制之下，但是解决所做的事情、移动代码等仍然很耗时。
*   其他项目可以独立于实现来引用生成的工件。

为了将生成的存根与实现分开，我们有以下初始结构:

```
.
+-- README.md
│-- fuse-impl
│   +-- pom.xml
│   `-- src
│       │-- main
│       │   │-- java
│       │   `-- resources
│       `-- test
│           │-- java
│           `-- resources
`-- stub
    │-- pom.xml
    `-- src
        `-- spec

```

文件夹`stub`包含生成工件的项目。文件夹`fuse-impl`包含我们实际服务的实现。

## 使用 Swagger 设置代码生成

首先，通过在存根项目的`pom.xml`文件中添加以下内容来配置 Swagger 插件:

```
…
<dependencies>
  <dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-swagger-java-starter</artifactId>
  </dependency>
…
</dependencies>
…
<plugins>
  <plugin>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-restdsl-swagger-plugin</artifactId>
    <version>2.23.0</version>
    <executions>
      <execution>
        <goals>
          <goal>generate-xml-with-dto</goal><!-- 1 -->
        </goals>
      </execution>
    </executions>
    <configuration>
      <specificationUri><!-- 2 -->
        ${project.basedir}/src/spec/beer-catalog-API.json
      </specificationUri>
      <fileName>camel-rest.xml</fileName><!-- 3 -->
      <outputDirectory><!-- 4 -->
              ${project.build.directory}/generated-sources/src/main/resources/camel-rest
      </outputDirectory>
      <modelOutput>
        ${project.build.directory}/generated-sources
      </modelOutput>
      <modelPackage>com.example.beer.dto</modelPackage><!-- 5 -->
    </configuration>
  </plugin>
</plugins>
...

```

这个插件很容易配置:

1.  目标设置为`generate-xml-with-dto`，这意味着 rest DSL XML 文件与我的数据传输对象一起从定义中生成。还有其他选项，包括为界面生成 Java 客户端的选项。
2.  指向我的 API 定义的位置。
3.  要生成的 rest DSL XML 文件的名称。
4.  将生成的 rest DSL XML 文件输出到哪里。如果放在这个位置，Camel 会自动拾取它(如果包含在项目中)。
5.  dto 的包名。

在`pom.xml`中，我们还需要为编译器改变源文件和资源文件的位置。最后，我们需要将 API 规范复制到我们之前选择的位置。这里没有描述，因为这是已知的东西，但是你可以根据需要参考[源代码](https://github.com/rh-demos/apicurio-fuse)来了解细节。现在，我们准备为 REST 服务生成存根。

到目前为止，我们在`stub`项目中有以下文件结构:

```
.
`-- stub
    +-- pom.xml
    `-- src
        `-- spec
            `-- beer-catalog-API.json

```

在`stub`目录中运行`mvn install`,存根会自动生成、编译、放入 jar 文件并放入本地 Maven 存储库中。dto 在我们之前选择的包中生成。此外，还为 REST 端点创建了一个 XML 文件。

文件`stub/target/generated-sources/src/main/resources/camel-rest/camel-rest.xml`的内容:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<rests >
    <restConfiguration component="servlet"/>
    <rest>
        <get id="GetBeer" uri="/beer/{name}">
            <description>Get beer having name</description>
            <param dataType="string" description="Name of beer to retrieve" name="name" required="true" type="path"/>
            <to uri="direct:GetBeer"/>
        </get>
        <get id="FindBeersByStatus" uri="/beer/findByStatus/{status}">
            <description>Get beers having status</description>
            <param dataType="string" description="Status of beers to retrieve" name="status" required="true" type="path"/>
            <param dataType="number" description="Number of page to retrieve" name="page" required="false" type="query"/>
            <to uri="direct:FindBeersByStatus"/>
        </get>
        <get id="ListBeers" uri="/beer">
            <description>List beers within catalog</description>
            <param dataType="number" description="Number of page to retrieve" name="page" required="false" type="query"/>
            <to uri="direct:ListBeers"/>
        </get>
    </rest>
</rests>

```

需要注意的重要一点是，每个 REST 操作都被路由到一个名为`direct:operatorId`的`uri`，其中`operatorId`与 API 定义文件中的操作符相同。这使我们能够轻松地为每个操作提供一个实现。

## 提供 API 的实现

对于示例实现，我们选择在 [Spring Boot](https://developers.redhat.com/topics/spring-boot/all?page=1?page=1) 容器中运行的 Fuse，以使其易于在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 中部署。

除了通常的样板代码之外，我们唯一要做的事情就是向项目添加一个依赖项，该依赖项包含我们的`fuse-impl`项目的`pom.xml`文件中的存根:

```
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>beer</artifactId>
      <version>1.0</version>
    </dependency>

```

现在我们都设置好了，我们可以提供我们的三个操作的实现。作为实现的示例，请考虑以下内容。

`fuse-impl/src/main/java/com/example/beer/routes/GetBeerByNameRoute.java:`的内容

```
package com.example.beer.routes;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.model.dataformat.JsonLibrary;
import org.springframework.stereotype.Component;

import java.math.BigDecimal;
import com.example.beer.service.BeerService;
import com.example.beer.dto.Beer;
import org.apache.camel.BeanInject;

@Component
public class GetBeerByNameRoute extends RouteBuilder {
  @BeanInject
  private BeerService mBeerService;

  @Override
  public void configure() throws Exception {
    from("direct:GetBeer").process(new Processor() {

      @Override
      public void process(Exchange exchange) throws Exception {
        String name = exchange.getIn().getHeader("name", String.class);
        if (name == null) {
          throw new IllegalArgumentException("must provide a name");
        }
        Beer b = mBeerService.getBeerByName(name);

        exchange.getIn().setBody(b == null ? new Beer() : b);
      }
    }).marshal().json(JsonLibrary.Jackson);
  }
}

```

这里我们注入了一个`BeerService`，它保存了关于不同啤酒的信息。然后我们定义一个直接端点，它提供了 REST 调用被路由到的端点(还记得前面提到的`operationId`吗？).处理器试图查找啤酒。如果没有找到啤酒，则返回一个空的啤酒对象。要尝试该示例，您可以运行:

```
cd fuse-impl
mvn package
java -jar target/beer-svc-impl-1.0-SNAPSHOT.jar
#in a separate terminal
curl http://localhost:8080/rest/beer/Carlsberg
{"name":"Carlsberg","country":"Denmark","type":"pilsner","rating":5,"status":"available"}

```

我们可能不得不一次又一次地这样做。在这种情况下，我们可以为这两个项目创建一个 Maven 原型。或者，我们可以克隆一个包含所有样板代码的模板项目，并在那里进行必要的修改。不过，这要多做一点工作，因为我们必须重命名 Maven 模块和 Java 类，但这不会太麻烦。

## 结论

使用 API 优先的方法，您可以在实际实现之前设计和测试您的 API。你可以从使用你的 API 的人那里得到早期的反馈，而不必提供一个实际的实现。这样，你可以节省时间和金钱。

使用 [Red Hat Fuse](https://developers.redhat.com/products/fuse) ，从设计到实际实施都很容易。只需使用 [Camel REST DSL Swagger Maven 插件](https://github.com/apache/camel/blob/master/tooling/maven/camel-restdsl-swagger-plugin/src/main/docs/camel-restdsl-swagger-plugin.adoc)生成一个存根，就可以提供实际的实现了。如果您想亲自尝试，请使用[示例代码](https://github.com/rh-demos/apicurio-fuse)作为起点。

*Last updated: January 18, 2022*