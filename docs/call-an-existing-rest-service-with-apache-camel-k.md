# 使用 Apache Camel K 调用现有的 REST 服务

> 原文：<https://developers.redhat.com/blog/2020/09/28/call-an-existing-rest-service-with-apache-camel-k>

随着 Apache Camel K 的[发布，可以创建和部署与现有应用程序的集成，这比以往任何时候都更快、更轻量级。在许多情况下，调用现有的 REST 端点是将新系统连接到现有系统的最佳方式。以一家供应咖啡的咖啡馆为例。当咖啡馆希望允许顾客使用 GrubHub 这样的送货服务时会发生什么？你只需要引入一个](https://developers.redhat.com/blog/2020/06/18/camel-k-1-0-the-serverless-integration-platform-goes-ga/) [Camel K](https://developers.redhat.com/blog/2020/05/12/six-reasons-to-love-camel-k/) 集成来连接 cafe 和 GrubHub 系统。

在本文中，我将向您展示如何创建一个调用现有 REST 服务并使用其现有数据格式的 [Camel K 集成](https://developers.redhat.com/videos/youtube/51x9BewGCYA)。对于数据格式，我有一个用 [Java](https://developers.redhat.com/topics/enterprise-java) 对象配置的 Maven 项目。理想情况下，您应该将它打包并放在 Nexus 存储库中。出于演示的目的，我使用了 [JitPack](https://jitpack.io/) ，这让我可以直接从我的 GitHub 代码中获得我的依赖项。参见与本演示相关的 [GitHub 库](https://github.com/jeremyrdavis/quarkus-cafe-demo/tree/kamel-1.0.0/grubhub-cafe-core)以获取数据格式代码和将其导入 JitPack 的说明。

## 先决条件

为了进行演示，您需要在开发环境中安装以下软件:

*   `oc`命令行工具
*   [骆驼 K 客户端 1.0.0](https://github.com/apache/camel-k/releases)
*   一个[红帽 OpenShift 4.4 集群](https://developers.redhat.com/products/openshift/overview)

## 创建骆驼 K 路线

首先，我们创建 Camel K route 文件，我将其命名为`RestWithUndertow.java`。在这里，我们打开文件并创建类结构:

```
public class RestWithUndertow extends org.apache.camel.builder.RouteBuilder {
    @Override
    public void configure() throws Exception {
    }
}
```

接下来，我们创建 REST 端点，并且创建我们将使用的数据格式。在这种情况下，我们将接收 REST 请求作为一个`GrubHubOrder`。然后我们将把它转换成一个`CreateOrderCommand`，我们将把它发送给已经在使用的 REST 服务:

```
import org.apache.camel.model.rest.RestBindingMode;
import com.redhat.quarkus.cafe.domain.CreateOrderCommand;
import com.redhat.grubhub.cafe.domain.GrubHubOrder;
import org.apache.camel.component.jackson.JacksonDataFormat;

public class RestWithUndertow extends org.apache.camel.builder.RouteBuilder {
    @Override
    public void configure() throws Exception {
        JacksonDataFormat df = new JacksonDataFormat(CreateOrderCommand.class);

        rest()
            .post("/order").type(GrubHubOrder.class).consumes("application/json")
            .bindingMode(RestBindingMode.json)
            .produces("application/json")
            .to("direct:order");
    }
}

```

## 创建数据转换

现在我们可以在同一个文件中创建一个方法，这将有助于从`GrubHubOrder`到`CreateOrderCommand`的数据转换:

```
    public void transformMessage(Exchange exchange){
        Message in = exchange.getIn();
        GrubHubOrder gho = in.getBody(GrubHubOrder.class);
        List oi = gho.getOrderItems();
        List list = new ArrayList();
        for(GrubHubOrderItem i : oi){
            LineItem li = new LineItem(Item.valueOf(i.getOrderItem()),i.getName());
            list.add(li);
        }
        CreateOrderCommand coc = new CreateOrderCommand(list, null);
        in.setBody(coc);
    }

```

还要确保将以下导入添加到文件中:

```
import org.apache.camel.Exchange;
import org.apache.camel.Message;
import com.redhat.quarkus.cafe.domain.LineItem;
import com.redhat.quarkus.cafe.domain.Item;
import java.util.List;
import java.util.ArrayList;
import com.redhat.grubhub.cafe.domain.GrubHubOrderItem;

```

## 从您的 Camel K REST 端点调用现有服务

既然我们有了完成转换的方法，我们就可以实现 Camel K REST 端点的其余部分，并让它调用现有的服务。在您目前拥有的代码下面添加以下内容:

```
        from("direct:order")
            .log("Incoming Body is ${body}")
            .log("Incoming Body after unmarshal is ${body}")
            .bean(this,"transformMessage")
            .log("Outgoing pojo Body is ${body}")
            .marshal(df) //transforms the java object into json
            .setHeader(Exchange.HTTP_METHOD, constant("POST"))
            .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
            .setHeader("Accept",constant("application/json"))
            .log("Body after transformation is ${body} with headers: ${headers}")
            .to("http://?bridgeEndpoint=true&throwExceptionOnFailure=false")
            .setHeader(Exchange.HTTP_RESPONSE_CODE,constant(200))
            .transform().simple("{Order Placed}");

```

注意，这个例子包含了大量的日志记录，以显示正在做的事情。我已经将`BridgeEndpoint`选项设置为`true`，这允许我们忽略`HTTP_URI`传入头，并使用我们指定的完整 URL。这在接收一个将调用新请求的传入 REST 请求时非常重要。你可以在这里阅读更多关于[的`BridgeEndpoint`选项。](https://camel.apache.org/components/latest/http-component.html)

## 保存路线文件

您完整的 Camel K route 文件应该如下所示:

```
import org.apache.camel.Exchange;
import org.apache.camel.Message;
import org.apache.camel.model.rest.RestBindingMode;
import com.redhat.quarkus.cafe.domain.LineItem;
import com.redhat.quarkus.cafe.domain.Item;
import java.util.List;
import java.util.ArrayList;
import com.redhat.quarkus.cafe.domain.CreateOrderCommand;
import com.redhat.grubhub.cafe.domain.GrubHubOrder;
import com.redhat.grubhub.cafe.domain.GrubHubOrderItem;
import org.apache.camel.component.jackson.JacksonDataFormat;

public class RestWithUndertow extends org.apache.camel.builder.RouteBuilder {

    @Override
    public void configure() throws Exception {
        JacksonDataFormat df = new JacksonDataFormat(CreateOrderCommand.class);

        rest()
            .post("/order").type(GrubHubOrder.class).consumes("application/json")
            .bindingMode(RestBindingMode.json)
            .produces("application/json")
            .to("direct:order");

        from("direct:order")
            .log("Incoming Body is ${body}")
            .log("Incoming Body after unmarshal is ${body}")
            .bean(this,"transformMessage")
            .log("Outgoing pojo Body is ${body}")
            .marshal(df)
            .setHeader(Exchange.HTTP_METHOD, constant("POST"))
            .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
            .setHeader("Accept",constant("application/json"))
            .log("Body after transformation is ${body} with headers: ${headers}")
            //need to change url after knowing what the cafe-web url will be
            .to("http://sampleurl.com?bridgeEndpoint=true&throwExceptionOnFailure=false")
            .setHeader(Exchange.HTTP_RESPONSE_CODE,constant(200))
            .transform().simple("{Order Placed}");
    }

    public void transformMessage(Exchange exchange){
        Message in = exchange.getIn();
        GrubHubOrder gho = in.getBody(GrubHubOrder.class);
        List oi = gho.getOrderItems();
        List list = new ArrayList();
        for(GrubHubOrderItem i : oi){
            LineItem li = new LineItem(Item.valueOf(i.getOrderItem()),i.getName());
            list.add(li);
        }
        CreateOrderCommand coc = new CreateOrderCommand(list, null);
        in.setBody(coc);
    }
}
```

保存这个文件，为最后一步做准备。

## 运行集成

要运行您的集成，您需要安装 Camel K 操作符，并确保它能够访问 JitPack 中的依赖项。执行以下操作，为集成准备好您的基础架构:

*   使用`oc`命令行登录 OpenShift。
*   通过 OpenShift OperatorHub 安装 Camel K 操作器。默认选项就可以了。
*   确保您拥有 [Kamel CLI 工具](https://camel.apache.org/camel-k/latest/cli/cli.html)
*   使用`oc`和`kamel`工具以及下面的命令创建一个集成平台，提供对 JitPack 的访问:

    ```
    kamel install --olm=false --skip-cluster-setup --skip-operator-setup --maven-repository https://jitpack.io@id=jitpack@snapshots

    ```

一旦您看到 OpenShift 控制台中的一切都准备好了(您可以随时输入`oc get pods`进行检查)，就部署集成。再次使用您的 Kamel 工具，确保您登录到 OpenShift 和适当的项目，并运行以下命令:

```
kamel run --name=rest-with-undertow --dependency=camel-jackson --dependency=mvn:com.github.jeremyrdavis:quarkus-cafe-demo:1.5-SNAPSHOT --dependency=mvn:com.github.jeremyrdavis.quarkus-cafe-demo:grubhub-cafe-core:1.5-SNAPSHOT --dependency=camel-openapi-java RestWithUndertow.java

```

此命令确保您的路线从所有适当的从属关系开始。参见本文的 [GitHub 知识库](https://github.com/jeremyrdavis/quarkus-cafe-demo/tree/kamel-1.0.0/camel-k-grub-hub)以获得该代码及其调用的服务的完整工作版本。

## 结论

Camel K 允许您快速高效地开发集成，同时保持较小的占用空间。即使您的集成有一些依赖项，您也可以利用 Camel K 中的 Knative 技术来降低集成的资源密集度，并允许更快的部署。

*Last updated: April 1, 2022*