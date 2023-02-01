# 基于事件的微服务和红帽 AMQ 流

> 原文：<https://developers.redhat.com/blog/2019/11/21/event-based-microservices-with-red-hat-amq-streams>

作为[红帽 AMQ](https://www.redhat.com/en/technologies/jboss-middleware/amq) 产品的一部分，红帽为传统部署和基于微服务的部署提供了基于 Kafka 的事件流解决方案，品牌为[红帽 AMQ 流](https://developers.redhat.com/blog/2019/10/03/deploy-red-hat-amq-streams-and-fuse-on-openshift-container-platform-4/)。 [Red Hat OpenShift](https://developers.redhat.com/openshift/) AMQ 流部署选项基于 [Strimzi，](https://strimzi.io/)一个开源工具，它使 Kafka 作为一个容器在 Kubernetes 平台上的部署变得容易，因为大多数部署先决条件都是通过 OpenShift [操作符框架](https://www.redhat.com/en/blog/introducing-operator-framework-building-apps-kubernetes)自动完成的。

在本文中，我们看看如何在 Red Hat OpenShift 4 上部署 Apache Kafka，使用合理的样本微服务应用程序来展示 OpenShift 和 Kafka 带来的无限创新可能性。

![AMQ Streams on OpenShift](img/18a2c6ab0ea250bbcbf65b8ab2b2a38b.png)

图 1:我们的部署环境。

图 1 展示了我们将在 OpenShift 上部署的环境，其中包含许多微服务，包括:

*   **账户余额服务**提供关于账户余额的信息，它有自己的 MongoDB 数据库服务。
*   **信用服务**执行账户之间的信用转移，将信用数据存储在`credit` Kafka 主题中。
*   **事件关联器**监听`credit`主题，对账户余额进行必要的调整，并通过 REST API 更新对账户余额服务的更改。同时，它发送这个过程的结果作为对 Kafka 主题`credit-response`的信用响应。
*   **MongoDB Kafka Connect** 监听`credit-response`主题中的内容，并将该信息传输到**信用响应 DB** 。

## 在 Red Hat OpenShift 上安装 AMQ 流

在 Red Hat OpenShift 上安装 AMQ 流非常简单。我面临的唯一问题是配置 MongoDB Kafka Connect，这主要是由于缺乏详细的文档和 Kafka Connect 中的一个 bug。所有这些问题现在都在结构上记录如下:

1.  从[红帽访问网站](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=jboss.amq.streams)下载 YAML 安装文件。

**注意:**我们正在使用集群管理安装这些 AMQ 流。AMQ 流包括几个自定义资源。默认情况下，创建、编辑和删除这些资源的权限仅限于 OpenShift 集群管理员。如果您希望允许非集群管理员管理 AMQ 流资源，您必须为他们分配 [Strimzi 管理员角色](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html/using_amq_streams_on_openshift/getting-started-str#assembly-getting-started-strimzi-admin-str)。

2.  使用 Kafka 操作符部署 Kafka 集群，该操作符可以监视 Kafka 资源的单个和多个名称空间。在我们的例子中，我们部署操作符来监视单个名称空间。下载并解压缩安装文件后，导航到根文件夹，其中包含两个文件夹:`examples`和`install`。
3.  从命令行运行以下命令，对为我们的单个 OpenShift 名称空间部署提供的 YAML 文件进行更改:

在 Linux 上:

```
$ sed -i 's/namespace: .*/namespace: my-kafka-example/' install/cluster-operator/*RoleBinding*.yaml
```

在 macOS 上:

```
$ sed -i '' 's/namespace: .*/namespace: my-kafka-example/' install/cluster-operator/*RoleBinding*.yaml
```

4.  运行以下命令，在名称空间更改后部署操作员:

```
$ oc apply -f install/cluster-operator -n my-kafka-example
```

**注意:**您还可以配置 Kafka 操作符来监视所有名称空间。详情请参考[文档](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html/using_amq_streams_on_openshift/getting-started-str#deploying-cluster-operator-to-watch-whole-cluster-str)。

5.  一旦部署了 Kafka 操作符，就部署 Kafka 集群。有两种选择:短暂的和持久的。我们将部署一个持久的 Kafka 集群。首先打开`examples/kafka/kafka-persistent.yaml`文件，并在`Kafka.metadata.name`属性中更改 Kafka 集群名称，如下所示:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: my-kafka-cluster
# ...
```

6.  如前所述，在同一个`kafka-persistent.yaml`文件中按如下方式配置主题操作符，以便能够自动创建在应用程序中配置的 Kafka 主题:

```
entityOperator:
    topicOperator:
      watchedNamespace: my-kafka-example
      reconciliationIntervalSeconds: 90
      zookeeperSessionTimeoutSeconds: 20
      topicMetadataMaxAttempts: 6
      image: registry.redhat.io/amq7/amq-streams-operator:1.3.0
```

7.  运行以下命令来部署 Kafka 集群:

```
oc apply -f examples/kafka/kafka-persistent.yaml
```

8.  部署并运行以下示例 Kafka 生成器:

```
oc run kafka-producer -ti --image=registry.redhat.io/amq7/amq-streams-kafka-23:1.3.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list kafka-cluster-kafka-bootstrap:9092 --topic my-topic
```

9.  部署并运行以下示例 Kafka 消费程序:

```
oc run kafka-consumer -ti --image=registry.redhat.io/amq7/amq-streams-kafka-23:1.3.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server kafka-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

10.  一旦所有的 pod 和资源都准备好了，就验证 Kafka 集群是否按预期工作，这意味着看看是否可以从生产者向消费者发送消息。

## 部署示例应用程序依赖项

由于 AMQ 流的原因，在部署代理的名称空间之外无法直接访问代理。我们将在与 Kafka 集群相同的名称空间中部署所有示例应用程序。这个问题对于所有部署为容器的 Kafka 代理都是一样的，这不是 OpenShift 的限制。所有外部连接都由 Kafka bootstrap 组件处理和路由。

**注意:**如果您希望从外部访问经纪人，请参考此[文章](https://developers.redhat.com/blog/2019/06/10/accessing-apache-kafka-in-strimzi-part-3-red-hat-openshift-routes/)中关于如何启用此选项的内容。

### 部署 MongoDB

This is our Credit Response database. Run the following command to deploy MongoDB using the provided template:

```
oc new-app -f https://raw.githubusercontent.com/chengkuangan/creditresponsemongodb/master/mongodb-deployment-template.yaml
```

在撰写本文期间，由于硬编码的数据库身份验证源，当 Kafka Connect 试图向 MongoDB 发送数据时，我在 Kafka Connect 容器日志中收到了以下错误(`source='admin'`):

```
2019-11-07 12:23:07,617 INFO Cluster created with settings {hosts=[creditresponse:27017], mode=SINGLE, requiredClusterType=UNKNOWN, serverSelectionTimeout='30000 ms', maxWaitQueueSize=500} (org.mongodb.driver.cluster) [task-thread-mongodb-sink-0]
2019-11-07 12:23:07,842 INFO Cluster description not yet available. Waiting for 30000 ms before timing out (org.mongodb.driver.cluster) [task-thread-mongodb-sink-0]
2019-11-07 12:23:07,861 INFO Opened connection [connectionId{localValue:1, serverValue:220}] to creditresponse:27017 (org.mongodb.driver.connection) [cluster-ClusterId{value='5dc40cab2efd9074c7742e33', description='null'}-creditresponse:27017]
2019-11-07 12:23:07,863 INFO Monitor thread successfully connected to server with description ServerDescription{address=creditresponse:27017, type=STANDALONE, state=CONNECTED, ok=true, version=ServerVersion{versionList=[3, 2, 10]}, minWireVersion=0, maxWireVersion=4, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=null, roundTripTimeNanos=1014605} (org.mongodb.driver.cluster) [cluster-ClusterId{value='5dc40cab2efd9074c7742e33', description='null'}-creditresponse:27017]
2019-11-07 12:23:07,892 INFO Closed connection [connectionId{localValue:2}] to creditresponse:27017 because there was a socket exception raised by this connection. (org.mongodb.driver.connection) [task-thread-mongodb-sink-0]
2019-11-07 12:23:07,894 ERROR Error on mongodb operation (com.mongodb.kafka.connect.sink.MongoSinkTask) [task-thread-mongodb-sink-0]
com.mongodb.MongoSecurityException: Exception authenticating MongoCredential{mechanism=SCRAM-SHA-1, userName='creditresponse', source='admin', password=<hidden>, mechanismProperties={}}
  at com.mongodb.internal.connection.SaslAuthenticator.wrapException(SaslAuthenticator.java:173)
```

要解决这个问题，请在 MongoDB pod 中执行`oc rsh`操作，使用以下详细信息创建一个新的`creditresponse`用户帐户:

```
mongo --port 27017 -u admin -p creditresponse --authenticationDatabase admin
use admin
db.runCommand({createRole:"listDatabases",privileges:[{resource:{cluster:true}, actions:["listDatabases"]}],roles:[]})
db.createUser({
"user" : "creditresponse",
"pwd" : "creditresponse",
    "roles" : [
        {
            "role" : "listDatabases",
            "db" : "admin"
        },
        {
            "role" : "readWrite",
            "db" : "creditresponse"
        },
        {
            "role" : "read",
            "db" : "local"
        }
    ]
})
```

### 部署和配置 MongoDB Kafka Connect

AMQ 流 Kafka Connect 只带有`FileStreamSourceConnector`和`FileStreamSinkConnector`。为了部署 MongoDB Kafka Connect，我们需要使用 MongoDB Kafka Connect JAR 文件和 Red Hat 支持的 AMQ 流基础映像来构建容器映像:

1.  继续创建包含以下内容的必要 docker 文件:

```
[source]
FROM registry.redhat.io/amq7/amq-streams-kafka-23:1.3.0
USER root:root
COPY ./mongo-plugins/ /opt/kafka/plugins/
USER kafka:kafka
```

2.  从 [MongoDB 网站](https://www.mongodb.com/kafka-connector)下载 MongoDB Kafka Connect JAR 文件。
3.  解压缩 JAR 文件并将其复制到`mongo-plugins`文件夹中。
4.  确保您拥有有效的 Red Hat 帐户，以便登录并访问`registry.redhat.io`。
5.  建立形象:

```
docker login registry.redhat.io

docker build -t chengkuan/amq-streams-kafka-connect-23:1.3.0
```

6.  将`kafka-connect.yaml`改为下面的`spec.image`、*、`spec.bootstrapServers`、*、`spec.tls.trustedCertificates.secretName`。注意 *`spec.bootstrapServers`* 的端口号是 9093，这是默认的`clienttls`端口:

```
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaConnect
metadata:
  name: mongodb-connect-cluster
spec:
  version: 2.3.0
  replicas: 1
  bootstrapServers: kafka-cluster-kafka-bootstrap:9093
  tls:
    trustedCertificates:
      - secretName: kafka-cluster-cluster-ca-cert
        certificate: ca.crt
  image: docker.io/chengkuan/amq-streams-kafka-connect-23:1.3.0
```

7.  部署 Kafka Connect:

```
oc apply -f examples/kafka-connect/kafka-connect.yaml
```

8.  从本地 PC 到 OpenShift 的 connect API 服务的端口转发，因为 Kafka Connect 的 pod 无法从外部访问:

```
oc port-forward service/mongodb-connect-cluster-connect-api 8083:8083
```

9.  使用`curl`或 web 浏览器运行以下命令，验证 MongoDB Connect 插件加载成功:

```
curl http://localhost:8083/connector-plugins
```

您将看到 MongoDB Connect 插件如下所示:

```
[{"class":"com.mongodb.kafka.connect.MongoSinkConnector","type":"sink","version":"0.2"},{"class":"com.mongodb.kafka.connect.MongoSourceConnector","type":"source","version":"0.2"},{"class":"org.apache.kafka.connect.file.FileStreamSinkConnector","type":"sink","version":"2.3.0.redhat-00003"},{"class":"org.apache.kafka.connect.file.FileStreamSourceConnector","type":"source","version":"2.3.0.redhat-00003"}]
```

要配置 MongoDB Kafka Connect，请下载 [`connect-mongodb-sink.json`](https://raw.githubusercontent.com/chengkuangan/creditresponsemongodb/master/connect-mongodb-sink.json) ，并相应地修改以下内容:

```
{
    "name": "mongodb-sink",
    "config": {
        "connector.class": "com.mongodb.kafka.connect.MongoSinkConnector",
        "tasks.max": 1,
        "topics": "credit-response",
        "connection.uri": "mongodb://creditresponse:creditresponse@creditresponse:27017",
        "database": "creditresponse",
        "collection": "response",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable": false,
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": false,
        "max.num.retries": 3
    }
}
```

接下来，发布到 Kafka Connect REST API:

```
curl -d connect-mongodb-sink.json -H "Content-Type: application/json" -X POST http://localhost:8083/connectors
```

**注意:**在撰写本文时，由于 JSON 格式错误，我无法使用`curl`命令将内容成功`POST`连接到 Kafka。然而，我可以使用 Postman 正确无误地`POST`相同的内容。

最后，运行下面的`curl`命令来验证配置:

```
curl http://localhost:8083/connectors/mongodb-sink
```

结果是:

```
{
    "name": "mongodb-sink",
    "config": {
        "connector.class": "com.mongodb.kafka.connect.MongoSinkConnector",
        "key.converter.schemas.enable": "false",
        "database": "creditresponse",
        "tasks.max": "1",
        "topics": "credit-response",
        "max.num.retries": "3",
        "connection.uri": "mongodb://creditresponse:creditresponse@creditresponse:27017",
        "value.converter.schemas.enable": "false",
        "name": "mongodb-sink",
        "collection": "response",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter"
    },
    "tasks": [
        {
            "connector": "mongodb-sink",
            "task": 0
        }
    ],
    "type": "sink"
}
```

### 部署帐户余额服务应用程序

运行以下命令来部署单元:

```
oc new-app https://raw.githubusercontent.com/chengkuangan/accountbalance/master/templates/deployment-templates.yaml
```

### 部署信用服务应用程序

运行以下命令来部署信用服务。确保`KAFKA_BOOTSTRAP_SERVER`指向正确的服务器:

```
oc new-app https://raw.githubusercontent.com/chengkuangan/creditservice/master/templates/creditservice.json -p KAFKA_BOOTSTRAP_SERVER=kafka-cluster-kafka-bootstrap:9092
```

### 部署事件关联器服务

Run the following command to deploy the Event Correlator Service. Ensure that `KAFKA_BOOTSTRAP_SERVER` points to the correct server:

```
oc new-app -f https://raw.githubusercontent.com/chengkuangan/eventcorrelator/master/templates/deployment-templates.yaml -p KAFKA_BOOTSTRAP_SERVER=kafka-cluster-kafka-bootstrap:9092
```

## 观看应用程序的运行

Now, let us use our application. Create account balance records:

```
curl -H "Content-Type: application/json" -X POST http://accountbalance-my-kafka-example.apps.demo.ocp.internal/ws/pg/balance -d '{"accountId": "20191108-MY-00000001", "balance": 500.00, "lastUpdatedDate": 1563178274158 }'
curl -H "Content-Type: application/json" -X POST http://accountbalance-my-kafka-example.apps.demo.ocp.internal/ws/pg/balance -d '{"accountId": "20191108-MY-00000002", "balance": 700.00, "lastUpdatedDate": 1563178274158 }'
```

查询上面创建的帐户余额条目:

```
curl http://accountbalance-my-kafka-example.apps.demo.ocp.internal/ws/pg/balance/all
```

结果是:

```
[
   {
      "_id":"5dc52069a3c1080001ebd539",
      "accountId":"20191108-MY-00000001",
      "balance":500.0,
      "lastUpdatedDate":1563178274158
   },
   {
      "_id":"5dc52076a3c1080001ebd53a",
      "accountId":"20191108-MY-00000002",
      "balance":700.0,
      "lastUpdatedDate":1563178274158
   }
]
```

执行信用转账:

```
curl -H "Content-Type: application/json" -X POST http://creditservice-my-kafka-example.apps.demo.ocp.internal/ws/pg/credits -d '{"amount": 10.50, "sourceAccount": "20191108-MY-00000001", "targetAccount": "20191108-MY-00000002"}'
```

查询贷记转账后的余额:

```
curl http://accountbalance-my-kafka-example.apps.demo.ocp.internal/ws/pg/balance/all
```

结果是:

```
[
   {
      "_id":"5dc52069a3c1080001ebd539",
      "accountId":"20191108-MY-00000001",
      "balance":489.5,"lastUpdatedDate":1573200543543
   },
   {
      "_id":"5dc52076a3c1080001ebd53a",
      "accountId":"20191108-MY-00000002",
      "balance":710.5,
      "lastUpdatedDate":1573200543543
   }
]
```

执行`oc rsh`进入信用响应 MongoDB。使用`db.response.find()`查看信用响应是否被捕获:

```
mongo --port 27017 -u admin -p creditresponse --authenticationDatabase admin

>use creditresponse
switched to db creditresponse

>show collections
response

>db.response.find()
```

结果:

```
{ 
   "_id" : ObjectId("5dc523f536d41402601d01a4"), 
   "sourceAccountRecordId" : "5dc52069a3c1080001ebd539", 
   "targetAccountRecordId" : "5dc52076a3c1080001ebd53a", 
   "sourceAccountId" : "20191108-MY-00000001", 
   "targetAccountId" : "20191108-MY-00000002", 
   "sourceAccountBalance" : 489.5, 
   "targetAccountBalance" : 710.5, 
   "creditRecordId" : "ykvlkqk2puzc5u", 
   "creditAmount" : 10.5, 
   "transactionDate" : NumberLong("1573200543543") 
}
```

### 参考

*   [open shift 文档中的红帽 AMQ 流](https://access.redhat.com/documentation/en-us/red_hat_amq/7.5/html/using_amq_streams_on_openshift/)
*   [Apache Kafka Connect 文档](https://kafka.apache.org/documentation/#connect)

*Last updated: July 1, 2020*