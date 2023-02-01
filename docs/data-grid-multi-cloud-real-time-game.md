# 使用 Red Hat 数据网格为多云实时游戏提供动力

> 原文：<https://developers.redhat.com/blog/2018/06/26/data-grid-multi-cloud-real-time-game>

在红帽峰会 2018 演示期间为观众开发的[寻宝游戏](https://developers.redhat.com/blog/2018/05/10/red-hat-summit-2018-burr-sutter-demo/)使用红帽数据网格作为除参与者拍摄的图片之外的所有内容的存储。使用跨站点复制在三个不同的云环境中存储数据。在这篇博文中，我们将看看数据是如何在数据网格中流动的，并解释驱动游戏功能不同方面的数据网格特性。

在其最简单的形式中，数据网格公开了一个键/值映射 API。该 API 在整个游戏中用于存储和检索信息，如游戏任务、每个站点的活跃用户、玩家、图片/任务尝试(也称为事务)和分数。

一些数据，如球员信息，包括每个球员的分数，被编入索引以提高检索速度。这样做是为了方便计算排行榜，即得分最高的前 10 名玩家。所以，从玩家的数据来看，我们只需要索引它的分数。我们可以使用 Infinispan Protostream 注释非常容易地做到这一点(完整的源代码可以在[这里](https://github.com/rhdemo/scavenger/blob/master/microservices/scavenger-microservice/src/main/java/me/escoffier/keynote/Player.java)找到)，例如:

```
@ProtoDoc("@Indexed")
@ProtoMessage(name = "Player")
public class Player {

private int score;

…
@ProtoDoc("@IndexedField")
@ProtoField(number = 10, required = true)
public int getScore() {
return score;
}
...

}

```

计算排行榜很容易。数据网格客户机只需为它构建查询，将其发送到数据网格并处理结果。该查询看起来像这样:

```
Query query = queryFactory.from(Player.class)
.orderBy("score", SortOrder.DESC)
.maxResults(10)
.build();

```

值得注意的是，玩家数据和索引本身都是跨站点复制的。这意味着无论你碰到哪一片云，你都会得到相同的、一致的结果。

即使它没有最终入围，我们也计划让用户能够计算出他们在排行榜中的个人位置。提出这样一个查询有点复杂，但是我们可以通过使用这个查询来实现:

```
int playerScore = ...
Query query = qf.from(Player.class)
.select(count("score"))
.orderBy("score", SortOrder.DESC)
.having("score").gt(playerScore)
.groupBy("score")
.build();

List list = query.list();
final long playerRank = list.size() + 1;

```

该查询将不同分数分组，按降序排序，并计算有多少分数大于给定玩家的分数。返回的数字加上 1，将给出玩家的位置。这意味着如果几名球员得分相同，他们将分享相同的位置，但这很常见，例如高尔夫锦标赛排名。

在不同的云中保存索引数据并不容易。我们必须处理的一个问题是缺少索引数据模式的默认跨站点复制。为了能够索引数据，Data Grid 必须能够分解从客户端收到的二进制数据，以发现各个字段，确定要索引哪些字段，等等。为了解决这个问题，我们创建了一个 schema keeper 组件(代码可以在[这里](https://github.com/rhdemo/scavenger-schemaer)找到),它检查玩家的模式是否存在于给定的云中，如果不存在，就注册它。这个组件被部署为带有每个数据网格实例的 sidecar。数据网格团队正在努力避免将来需要这样的组件。

每当一张图片被上传并被评分，分数将被单独存储在数据网格中。这将触发数据网格使用远程客户端侦听器发送一个事件，该事件将被游戏选中并转发给用户。请记住，数据网格将数据存储在键/值对结构中，分数将存储在值部分。但是，默认情况下，远程客户端侦听器事件只传送密钥(和版本)信息。为了避免额外的查找，远程客户机监听器配置了一个名为`key-value-with-previous-converter-factory`的转换器工厂，它是现成可用的。通过这样做，每个事件都被转换为包含值部分和键。示例:

```
@ClientListener(converterFactoryName = "key-value-with-previous-converter-factory", ...)
public class RemoteCacheListener {
...
}
```

对于每个新的分数，我们希望触发一个事件。然而，由于远程客户端监听器的工作方式，默认情况下，每个分数将触发尽可能多的可用云/站点事件。为了避免这种情况，我们使用远程客户端监听器过滤器来创建一个服务器端部署的过滤器，该过滤器会将分数的来源云(AWS、Azure 或 Private)与执行过滤器的云进行比较。

例如，如果一个分数来自 AWS，那么只有运行在 AWS 云中的过滤器才允许向客户端触发事件。当这个分数到达 Azure 或私有云时，过滤器将检测到事件源自不同的云，并且不会触发它。过滤器的代码可以在[这里](https://github.com/rhdemo/infinispan-listener-optimizations/blob/master/src/main/java/fn/dg/os/filters/SiteFilterFactory.java)找到。

要将其添加到服务器，需要将过滤器部署到服务器中。对于演示，我们用包含过滤器和相关文件的 JAR 扩展了基本数据网格图像。

最后，为了将过滤器应用于监听器，我们将过滤器工厂的名称添加到客户机监听器注释的`filterFactoryName`属性中。示例:

```
@ClientListener(
converterFactoryName = "key-value-with-previous-converter-factory",
filterFactoryName = "site-filter-factory"
)
public class RemoteCacheListener {
…
}
```

这篇博文到此结束，在这篇博文中，我们看到了寻宝游戏的应用层如何使用 Red Had 数据网格来存储和公开游戏中使用的元数据信息。

红帽数据网格团队

*Last updated: June 25, 2018*