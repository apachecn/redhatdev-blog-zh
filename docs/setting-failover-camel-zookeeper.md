# 使用 Apache Camel ZooKeeper 设置故障转移场景

> 原文：<https://developers.redhat.com/blog/2018/04/03/setting-failover-camel-zookeeper>

在本文中，我们将讨论如何使用 [Apache Camel ZooKeeper](http://camel.apache.org/zookeeper.html) 组件，并演示如何轻松地为 [Apache Camel Routes](http://camel.apache.org/routes.html) 设置故障转移场景。在集群环境中工作时，会出现这样的情况:用户希望有一个备用或从属路由，只有当主路由(或当前活动路由)停止工作时，备用或从属路由才会变为活动路由。有不同的方法来实现这一点:一个可以使用[石英](http://camel.apache.org/quartz.html)来配置一个主从设置；[也可以使用 JGroups](http://camel.apache.org/jgroups.html) 。在 [Fabric8](https://fabric8.io/gitbook/overview.html) 环境中，有一个主组件可以轻松设置为故障转移场景。

使用 [ZooKeeperRoutePolicy](https://github.com/apache/camel/blob/master/components/camel-zookeeper/src/main/java/org/apache/camel/component/zookeeper/policy/ZooKeeperRoutePolicy.java) 还有一种方式在这里详细讨论。这个策略在所有实例中使用一个公共的 znode (zookeeper 注册表)路径。每个实例将在 znode 路径中注册一个唯一的 id；因此，将会有一个唯一 id 列表。每个唯一 id 代表使用 ZooKeeperRoutePolicy 路由策略部署的路由的一个实例。

位于列表顶部的节点将被视为主路由。其他剩余的节点将是休眠的和不活动的。一旦主路由停止，其中一个从路由将被选为主路由。那么它将继续服务于请求。

#### **设置项目并将 Camel Zookeeper 配置为主从路线**

现在让我们讨论编码部分。我在[红帽 JBoss Fuse](https://developers.redhat.com/products/fuse/overview/) 6.3.0 中测试过这个例子。你可以在我的[个人 github 链接](https://github.com/1984shekhar/fuse-examples-6.3/tree/master/CamelZookeeperMasterSlave)找到代码库。

1.使用以下结构创建一个 Maven 项目:

```
[cpandey@cpandey CamelZookeeperMasterSlave]$ tree
.
├── pom.xml
├── ReadMe.txt
└── src
 └── main
  ├── java
   └── com
    └── test
     └── pckg
      └── MyRouteBuilder.java
  └── resources
   ├── log4j.properties
   └── OSGI-INF
    └── blueprint
     └── blueprint.xml
```

2.pom.xml 应该具有以下依赖关系:

```
 <dependency>
 <groupId>org.apache.camel</groupId>
 <artifactId>camel-core</artifactId>
 </dependency>
 <dependency>
 <groupId>org.apache.camel</groupId>
 <artifactId>camel-blueprint</artifactId>
 </dependency>
 <dependency>
 <groupId>org.apache.camel</groupId>
 <artifactId>camel-zookeeper</artifactId>
 </dependency>
```

3.这些工件的一个版本选自 pom.xml 中提到的 BOM 版本:

```
 <dependencyManagement>
 <dependencies>
 <dependency>
 <groupId>org.jboss.fuse.bom</groupId>
 <artifactId>jboss-fuse-parent</artifactId>
 <version>6.3.0.redhat-310</version>
 <type>pom</type>
 <scope>import</scope>
 </dependency>
 </dependencies>
 </dependencyManagement>
```

4.在这里，我用`JAVA DSL`表示骆驼路线。因此在`blueprint.xml`中，我定义了 bean，它引用了定义骆驼路线的类`com.test.pckg.MyRouteBuilder`。

```
<blueprint 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.osgi.org/xmlns/blueprint/v1.0.0 https://www.osgi.org/xmlns/blueprint/v1.0.0/blueprint.xsd http://camel.apache.org/schema/blueprint http://camel.apache.org/schema/blueprint/camel-blueprint.xsd">

 <camelContext id="_context1" >
  <routeBuilder ref="myBuilder"/>
 </camelContext>
 <bean class="com.test.pckg.MyRouteBuilder" id="myBuilder"/>
</blueprint>

```

5.沿着`MyRouteBuilder.java`中定义的骆驼路线:

```
@Override
public void configure() {
System.out.println("......Inside MyRouteBuilder.....");
ZooKeeperRoutePolicy policy = new ZooKeeperRoutePolicy("zookeeper:localhost:" + 2181 + "/someapp/somepolicy", 1);
from("file:/path/to/someFolder").routePolicy(policy).id("Master").log("Body... ${body} "+new Date());
}
```

The tasks performed by this piece of code are:

*   定义一个在路径/someapp/somepolicy 创建 znode 的`ZooKeeperRoutePolicy(String uri, int enabledCount)`。ZooKeeper 服务器正在监听本地主机:2181。
*   我们希望一次有一个实例处于活动状态。对于主-从场景，路由配置有 1 个路由实例，只有列表中的第一个条目将开始它的路由。但是，如果我们需要两个或更多的路由实例同时运行，我们可以将 enabledCount 设置为 2 或更多。
*   然后在`path/to/someFolder`我们有一个基于文件的消费者轮询，其中 routePolicy 设置为 ZooKeeperRoutePolicy。最后，我们记录设置了轮询或使用的文件内容的主体。

#### **如何在具有 Fabric8 环境的 Red Hat JBoss Fuse 中部署**

1.Fabric8 使用 [Apache ZooKeeper](http://zookeeper.apache.org/) (这是一个高度可靠的分布式协调服务)作为其注册表来存储集群配置和节点注册。因此，我们不需要设置不同的 ZooKeeper 服务器。我们可以在现有的 ZooKeeper 注册表中注册 znode，并使用它来设置主从路由。

2.按照以下步骤从 Red Hat JBoss Fuse 的 Karaf 终端部署此应用程序:

```
profile-edit --feature fabric-zookeeper-commands default
container-create-child root abc
container-create-child root pqr 
profile-create testZK
profile-edit --feature camel-zookeeper --feature camel-blueprint testZK
profile-edit --bundle mvn:com.mycompany/camel-zookeeper-example/1.0.0-SNAPSHOT testZK
container-add-profile abc testZK
container-add-profile pqr testZK
```

3.按照我们的路线，在位置`/path/to/someFolder`，创建一个简单的文本文件，添加一些文本数据并保存。

4.在其中一个节点的日志中，我们可以检查以下内容:这将确保创建 znode 并且路由是活动的:

```
2018-03-25 17:30:30,075 | WARN | masterSlaveRoute | ZookeeperProducer | 191 - org.apache.camel.camel-core - 2.17.0.redhat-630329 | Node '/someapp/somepolicy/localhost.localdomain-cd65c741-4252-420a-88a4-f084a933fcaf' did not exist, creating it.
2018-03-25 17:30:30,080 | INFO | masterSlaveRoute | ZooKeeperElection | 195 - org.apache.camel.camel-zookeeper - 2.17.0.redhat-630329 | Candidate node '/someapp/somepolicy/localhost.localdomain-cd65c741-4252-420a-88a4-f084a933fcaf' has been created
2018-03-25 17:30:30,107 | INFO | masterSlaveRoute | BlueprintCamelContext | 191 - org.apache.camel.camel-core - 2.17.0.redhat-630329 | Route: election-route-localhost.localdomain-cd65c741-4252-420a-88a4-f084a933fcaf started and consuming from: Endpoint[zookeeper://localhost:2181/someapp/somepolicy]
2018-03-25 17:30:30,112 | INFO | masterSlaveRoute | Master | 191 - org.apache.camel.camel-core - 2.17.0.redhat-630329 | Body... Sun Mar 25 17:27:32 IST 2018
```

4.从 Karaf 终端使用下面的命令，我们可以很容易地识别哪些节点被注册。我们还可以在日志中识别这个唯一的编号:

```
zk:list -r /someapp/somepolicy
#Output we get is like
/someapp/somepolicy/localhost.localdomain-cd65c741-4252-420a-88a4-f084a933fcaf0000000013
```

5.如果路由或应用程序停止，此 znode 条目将被删除。

#### **如何在独立环境中部署 Red Hat JBoss Fuse**

1.从这里下载动物园管理员的最新版本。我下载了 zookeeper-3.4.11。

2.提取 zookeeper-3.4.11.tar.gz。现在转到 zookeeper-3.4.11/conf 文件夹:

```
$ vi conf/zoo.cfg

tickTime = 2000
initLimit = 10
syncLimit = 5
dataDir = /path/to/zookeeper/data
clientPort = 2181
```

3.最后，启动 ZooKeeper 服务器:

```
$ bin/zkServer.sh start
```

4.现在在 Red Hat JBoss Fuse 6.3.0 Karaf 终端上，运行以下命令:

```
features:install camel-zookeeper
osgi:install -s mvn:com.mycompany/camel-zookeeper-example/1.0.0-SNAPSHOT
```

5.按照我们的路线，在位置`/path/to/someFolder`，创建一个简单的文本文件，添加一些文本数据并保存。我们将在 Fabric8 环境中获得类似的部署日志(如上所述)。在这里，我们可以让多个 Red Hat JBoss Fuse 独立环境运行在具有相同应用程序部署的不同节点上。

6.如果部署和测试已经完成，您可以使用以下命令停止 ZooKeeper 服务器:

```
bin/zkServer.sh stop
```

就是这样。我希望它能帮助你建立一个有骆驼路线的主-从场景。这可能有助于您实现成功的故障转移场景。