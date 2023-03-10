# 利用 Quarkus 和 Red Hat 数据网格构建嵌入式缓存集群

> 原文：<https://developers.redhat.com/blog/2020/12/17/build-embedded-cache-clusters-with-quarkus-and-red-hat-data-grid>

在[微服务](https://developers.redhat.com/topics/microservices/)系统中配置缓存有多种方式。根据经验，您应该只在一个地方使用缓存；例如，您不应该在 HTTP 和应用层都使用缓存。分布式缓存既提高了云原生应用的性能，又最大限度地减少了创建新微服务的开销。

[Infinispan](https://infinispan.org/) 是一个开源的内存数据网格，可以作为分布式缓存或 NoSQL 数据存储运行。您可以将它用作缓存，比如用于会话集群，或者用作数据库前面的数据网格。 [Red Hat Data Grid](https://developers.redhat.com/products/datagrid/overview) 构建于 Infinispan 之上，具有额外的特性并支持企业生产环境。

数据网格允许您通过嵌入式的 Java 库或独立于语言的远程服务来访问分布式缓存。远程服务使用飞车手罗德、REST 和 Memcached 等协议。在本文中，您将学习如何用 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 和数据网格构建一个分布式缓存系统。我们将使用 Quarkus 来集成部署到[红帽 OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview) (RHOCP)的两个集群化嵌入式数据网格缓存。图 1 显示了这个例子的分布式缓存架构。

[![Clustered embedded RHDG](img/8ceb6f72822ea3e08b55ae1395867404.png "Clustered embedded RHDG")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-10.21.07-AM.png)Clustered embedded RHDG

Figure 1: Clustered embedded Data Grid caches in an OpenShift Container Platform deployment.

## 用 Quarkus 和数据网格构建分布式缓存系统

在本例中，我们将创建一个简单的记分卡应用程序，将数据存储在两个集群嵌入式缓存中。我们将一步一步地完成这个过程。

### 步骤 1:创建并配置 Quarkus 应用程序

第一步是设置 Quarkus 应用程序。您可以使用 [Quarkus 工具](https://quarkus.io/blog/march-of-ides/)在您喜欢的 IDE 中生成一个 Quarkus 项目。查看[完整源代码](https://github.com/danieloh30/embedded-caches-quarkus)中的 Quarkus 应用示例。

### 步骤 2:添加 Infinispan-embedded 和 OpenShift 扩展

在您用`artifactId` ( `embedded-caches-quarkus`)配置了 Quarkus 项目之后，您将需要添加所需的 Quarkus 扩展。将以下依赖项添加到您的 Maven `pom.xml`中:

```
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-infinispan-embedded</artifactId>
</dependency>
<dependency>
  <groupId>org.infinispan.protostream</groupId>
  <artifactId>protostream-processor</artifactId>
</dependency>
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-openshift</artifactId>
</dependency>

```

或者，从 Quarkus 项目的基本目录运行以下 Maven 插件命令:

```
./mvnw quarkus:add-extension -Dextensions="infinispan-embedded,openshift"

```

### 步骤 3:创建缓存服务

接下来，我们将创建一个缓存服务。Red Hat Data Grid 为创建、修改和管理集群缓存提供了一个`EmbeddedCacheManager`接口。`CacheManager`接口与客户端应用程序运行在同一个 Java 虚拟机(JVM)中。在您的 Quarkus 项目中创建一个名为`ScoreService.java`的新服务，然后在示例应用程序中添加以下代码来保存、删除和检索分数数据:

```
EmbeddedCacheManager cacheManager;
public List<Score> getAll() {
  return new ArrayList<>(scoreCache.values());
}

public void save(Score entry) {
  scoreCache.put(getKey(entry), entry);
}

public void delete(Score entry) {
  scoreCache.remove(getKey(entry));
}
```

### 步骤 4:初始化缓存节点

接下来，您将定义一个`GlobalConfigurationBuilder`来在启动时初始化所有缓存节点。创建新节点时，将自动复制现有的缓存数据。在`ScoreService.java class`中增加以下`onStart`方法:

```
void onStart(@Observes @Priority(value = 1) StartupEvent ev) {

   GlobalConfigurationBuilder global = GlobalConfigurationBuilder.defaultClusteredBuilder();
   global.transport().clusterName("ScoreCard");
   cacheManager = new DefaultCacheManager(global.build());

   ConfigurationBuilder config = new ConfigurationBuilder();
   config.expiration().lifespan(5, TimeUnit.MINUTES).clustering().cacheMode(CacheMode.REPL_SYNC);

   cacheManager.defineConfiguration("scoreboard", config.build());
   scoreCache = cacheManager.getCache("scoreboard");
   scoreCache.addListener(new CacheListener());

   log.info("Cache initialized");

}

```

### 步骤 5:配置传输层

Red Hat Data Grid 使用一个 [JGroups](http://www.jgroups.org/) 库来加入集群并通过传输层复制数据。要部署带有嵌入式 Infinispan 缓存的 Quarkus 应用程序，我们需要在`resources/default-configs/jgroups-kubernetes.xml`文件中添加以下`DNS_PING`配置:

```
<dns.DNS_PING dns_query="jcache-quarkus-ping" num_discovery_runs="3" />

```

### 步骤 6:添加 CacheListener

Red Hat Data Grid 为客户端提供了`CacheListener`API，以获得关于缓存事件的通知，比如当一个新的缓存条目被创建时。我们可以通过向普通的旧 Java 对象(POJO)类添加`@Listener`注释来实现一个监听器。让我们创建一个新的`CacheListener.java`类，并向其中添加以下代码:

```
@Listener
public class CacheListener {

   @*CacheEntryCreated*
   public *void* entryCreated(*CacheEntryCreatedEvent*<String, Score> *event*) {
       System.out.printf("-- Entry for %s created \n", event.getType());
   }

   @*CacheEntryModified*
   public *void* entryUpdated(*CacheEntryModifiedEvent*<String, Score> *event*){
       System.out.printf("-- Entry for %s modified\n", event.getType());
   }

}

```

### 步骤 7:构建 Quarkus 应用程序并将其部署到 OpenShift

我们之前添加的 OpenShift 扩展是一个包装器扩展，它结合了默认配置的 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 和`container-image-s2i`扩展。该扩展使得在 OpenShift 上开始使用 Quarkus 变得很容易。

Quarkus 允许我们基于默认或用户提供的配置自动生成 OpenShift 资源。要将我们的 Quarkus 应用程序部署到 OpenShift，我们只需将以下配置添加到应用程序的`application.properties`文件中:

```
quarkus.openshift.expose=true
quarkus.kubernetes-client.trust-certs=true
quarkus.container-image.build=true
quarkus.kubernetes.deploy=true

```

使用以下命令登录到 OpenShift 集群:

```
$ oc login --token=<YOUR_USERNAME_TOKEN> --server=<OPENSHIFT API URL>

```

然后，执行以下 Maven 插件命令:

```
./mvnw clean package

```

### 步骤 8:扩展到两个单元

假设我们已经成功部署了 Quarkus 应用程序，我们的下一步是扩展到两个 pod。运行以下命令:

```
oc scale dc/embedded-caches-quarkus --replicas=2

```

要验证 pod，请转到开发人员控制台中的 OpenShift 拓扑视图。检查这两个 pod 是否在`embedded-caches-quarkus`项目中运行，如图 2 所示。

[![The two pods are shown in OpenShift's topology view.](img/e6302ae84f2e9f9a3184d64ae50efa55.png "figure2")](/sites/default/files/blog/2020/12/figure2.png)

Figure 2: The two pods  are running in our Quarkus application

### 步骤 9:测试应用程序

最后一步，让我们测试应用程序。我们可以从使用 RESTful API 调用开始，在集群嵌入式缓存中添加一些分数:

```
curl --header "Content-Type: application/json" --request PATCH -d '{"card":[5,4,4,10,3,0,0,0,0,0,0,0,0,0,0,0,0,0],"course":"Bethpage","currentHole":4,"playerId":"4","playerName":"Daniel"}' <APP_ROUTE_URL>

```

如果您使用的是[示例应用程序](https://github.com/danieloh30/embedded-caches-quarkus)，您需要运行一个预定义的 bash 脚本(在本例中是`sh scripts/load.sh`)。

返回拓扑视图，点击**查看日志**，在各自的网络浏览器中查看每个 pod 的日志。这允许您从`CacheListener`开始监控缓存事件。您应该在两个数据网格节点中看到相同的缓存条目日志。图 3 显示了节点 1 的输出。

[![The CacheListener output from Node 1.](img/566e33780266773131f1c89504f78c8a.png "Screen Shot 2020-12-04 at 10.50.14 PM")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-04-at-10.50.14-PM.png)

Figure 3: Monitoring cache events from the CacheListener (Node 1).

图 4 显示了节点 2 的输出。

[![The CacheListener output from Node 2.](img/c82b43f7e3f2ea9a151c2dabdeeabe44.png "Screen Shot 2020-12-04 at 10.50.21 PM")](/sites/default/files/blog/2020/12/Screen-Shot-2020-12-04-at-10.50.21-PM.png)

Figure 4: Monitoring cache events from the CacheListener (Node 2).

## Quarkus 和 Red Hat 数据网格的下一步是什么

[红帽数据网格](https://www.redhat.com/en/technologies/jboss-middleware/data-grid?extIdCarryOver=true&sc_cid=701f2000000u72fAAA) 增加应用程序响应时间，让开发人员大幅提高性能，同时提供可用性、可靠性和弹性规模。使用 [Quarkus 的前端无服务器功能](https://quarkus.io/guides/funqy)和[外部 Red Hat 数据网格服务器](https://developers.redhat.com/blog/2020/10/15/securely-connect-quarkus-and-red-hat-data-grid-on-red-hat-openshift/)，您可以将这些优势集成到为高性能、自动扩展和超快速响应时间而设计的无服务器架构中。

*Last updated: May 17, 2021*