# 使用 GeoJSON 和 Apache Camel K 进行空间数据转换

> 原文：<https://developers.redhat.com/blog/2020/11/23/using-geojson-with-apache-camel-k-for-spatial-data-transformation>

在本文中，我们将定义并运行一个工作流，演示 Apache [Camel K](https://developers.redhat.com/topics/camel-k) 如何与标准化 [GeoJSON 格式](https://geojson.org/)的空间数据进行交互。虽然这个例子被简化了，但是您可以使用相同的工作流来[处理大数据和更复杂的数据转换](https://developers.redhat.com/blog/2020/05/05/working-with-big-spatial-data-workflows-or-what-would-john-snow-do/)。

您将学习如何使用 Camel K 转换 XML 和 JSON 等常见格式的数据。您还将看到如何连接到数据库并从中提取所需的数据。在我们定义了工作流之后，我们将在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift) 上运行集成。

**注**:关于使用 [Visual Studio 代码](https://developers.redhat.com/blog/category/vs-code/)、 [OpenShift](https://developers.redhat.com/openshift) 和 [Apache Camel K](https://github.com/apache/camel-k/releases/tag/1.0.0) 建立演示环境的分步说明，请参见[扩展示例](https://github.com/openshift-integration/camel-k-example-transformations)。

## 使用地理空间格式

地理信息有多种编码方式。在地理空间格式中，我们希望捕获数据和位置上下文，通常是在地理坐标中。存储的图像，如来自卫星的照片，被称为*光栅数据*。这种类型的图像通常包含元数据属性，以及定义照片拍摄位置的坐标。

当我们想要表示字母数字数据时，我们使用*矢量数据*。矢量数据包括特征:可以被放置或与空间坐标相关联的对象或元素。一个*特征*可以是一家银行、一座城市、一条道路等等。这些元素中的每一个都有定义其地理位置的相关点、线或多边形。

两种最常用的矢量数据格式是基于 XML 的[锁眼标记语言](https://en.wikipedia.org/wiki/Keyhole_Markup_Language)(KML)；还有 GeoJSON，基于 JSON。

## 杰奥森

GeoJSON 是一种众所周知的用于编码数据结构的标准。任何理解 JSON 的应用程序或库都可以使用它。GeoJSON 包括地理空间上下文的特定属性。例如，下面的代码片段定义了一个点:

```
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands",
    "country": "Philippines",
    "type": "Province"
  }
}
```

该示例包括一个 JSON 属性`type`，它告诉我们正在查看一个表示对象的特性。另一个属性`geometry`定义了空间中的一个点。还有一个包含`properties`的对象，它们是与空间中给定点相关联的属性。据`properties`所述，该点位于菲律宾一个名为迪纳加特群岛的省份。GeoJSON 没有定义属性的语义；这取决于每个用例。

我简要概述了使用地理空间数据和 GeoJSON。现在，让我们探索一下我们的数据转换工作流。

## 数据转换工作流

我们从读取一个 CSV 文件开始，并独立地遍历每一行。我们将查询一个 XML API，向每行中的每个元素添加更多数据。最后，我们将收集和聚合所有行来构建 GeoJSON 文件，并将其存储在一个 [PostgreSQL](https://www.postgresql.org/) 数据库中。虽然没有演示，但我们正在使用 [PostGIS](https://postgis.net/) 扩展向 PostgreSQL 数据库添加空间功能。图 1 描述了工作流程。

[![Workflow diagram](img/5d792116d1e6be8ede5469f7833906ab.png "flux_diagram")](/sites/default/files/blog/2020/07/flux_diagram.png)Workflow diagram

Figure 1: The spatial data transformation workflow.

在我们定义了工作流之后，我们将使用一个 [Java](https://developers.redhat.com/topics/enterprise-java) 文件来用 [Camel K](https://github.com/apache/camel-k/) 实现它。注意 [Camel K 除了 Java 还支持很多语言](https://camel.apache.org/camel-k/latest/languages/languages.html)。您可以使用任何受支持的语言来实现这个工作流。

## 步骤 1:启动数据转换工作流

为简单起见，我们从计时器开始。一个*定时器*是一种步骤，它根据给定的持续时间定义何时运行一个工作流。例如，我们可以每 120 秒运行一次工作流:

```
from("timer:java?period=120s")
```

我们可以用一个触发器来代替这个计时器，就像现实生活工作流程中的[卡夫卡](https://kafka.apache.org/)或 [RabbitMQ](https://camel.apache.org/components/latest/rabbitmq-component.html) 消息。使用消息作为触发器可以确保数据在更新时立即得到处理。

## 步骤 2:处理 CSV 文件中的原始数据集

接下来，我们读取 CSV 文件。对于这个例子，我们使用的是由欧洲环境局维护的空气质量数据集的一个小子集。我们收集托管在远程服务器上的数据，并将其作为 CSV 文件进行处理:

```
.to("{{source.csv}}")
.unmarshal("customCSV")

```

这个命令告诉 Camel K 将 CSV 数据转换为元素列表，每行一个元素。每个元素都是一个映射，其键是列名。我们希望迭代每一行，所以我们分割主体，并开始将其流式传输到工作流的其余部分:

```
.split(body()).streaming()
```

分割是我们处理 CSV 文件中每一行的方式。现在，每一行都是列表中的一个元素。

## 步骤 3:提取数据

在每一步，我们都想提取出我们感兴趣的数据。最好的方法是使用处理器来提取这些值。我们从 CSV 处理器开始:

```
.process(processCsv)
```

请记住，地图代表每个元素。对于每个元素，我们调用一个[nomim](https://nominatim.openstreetmap.org/)服务，查询进行测量的地址:

```
.setBody().constant("")
.setHeader(Exchange.HTTP_METHOD, constant("GET"))
.setHeader(Exchange.HTTP_QUERY, simple("lat=${exchangeProperty.lat}&lon=${exchangeProperty.lon}&format=xml"))
.to("https://nominatim.openstreetmap.org/reverse")

```

响应是 XML 格式的，我们可以对其进行解组以便于处理:

```
.unmarshal().jacksonxml()
```

图 2 显示了 XML 格式的 nomim 响应。

[![Nominatim example response, xml](img/c0fddb637fdaa96f4a3eb000ddfc9a28.png "nominatim_xml")](/sites/default/files/blog/2020/07/nominatim_xml.png)Nominatim example response, xml

Figure 2: The Nominatim example response in XML.

我们现在可以使用另一个处理器来提取更多我们感兴趣的数据，并将其添加到我们存储的数据集合中。这一次，我们将使用 XML 处理器:

```
.process(processXML)

```

我们还可以在 PostgreSQL 数据库中查询更多数据:

```
.setBody().simple("SELECT info FROM descriptions WHERE id like '${exchangeProperty.pollutant}'")
.to("jdbc:postgresBean?readSize=1")

```

我们将使用另一个处理器从数据库中提取我们想要的数据:

```
.process(processDB)
```

至此，我们已经处理了原始 CSV 文件的每一行，并添加了来自远程服务和数据库的数据。

## 步骤 4:将数据聚合到一个 GeoJSON 文件中

现在，我们将聚合所有的行来创建一个 GeoJSON 文件。该文件包含一个*特征集合*，其中每个特征都是从原始 CSV 文件的一行构建的:

```
.aggregate(constant(true), aggregationStrategy)
.completionSize(5)
.process(buildGeoJSON)
.marshal().json(JsonLibrary.Gson)

```

我们将 GeoJSON 结果存储在 PostgreSQL 数据库中:

```
.setBody(simple("INSERT INTO measurements (geojson) VALUES (ST_GeomFromGeoJSON('${body}'))"))
.to("jdbc:postgresBean")

```

## 步骤 5:运行与 Camel K 的集成

到目前为止，我们已经定义了一个工作流，但是我们还没有运行它。这是魔法开始的地方。

为了部署我们的集成，我们需要一个像 OpenShift 这样的 Kubernetes(T2)环境。假设我们已经设置了 OpenShift 环境，我们可以使用`kamel`客户端来启动我们的集成:

```
kamel run OurIntegration.java

```

当我们运行集成时，我们的工作流部署在一个容器上并无缝运行。

## 步骤 6:查看输出

作为我们的最后一步，我们可以查询存储在数据库中的数据，将其可视化在地图上，如图 3 所示。因为我们使用标准格式(GeoJSON)，所以可以在任何地理空间应用程序中使用它。在这种情况下，我们使用的是免费开源的桌面地理信息系统。

[![](img/b134a1769e190b7241b2cc5af10c1c1e.png "qgis")](/sites/default/files/blog/2020/06/qgis-1.png)

Figure 3: QGIS querying the database with GeoJSON.

## 结论

本文简要介绍了 GeoJSON，并向您展示了如何使用它来记录 Camel K 的空间数据。无论您的用例有多具体，Camel K 都可以适应任何情况。

*Last updated: November 19, 2020*