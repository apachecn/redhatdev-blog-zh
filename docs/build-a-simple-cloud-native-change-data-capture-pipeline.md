# 构建一个简单的云原生变更数据捕获管道

> 原文：<https://developers.redhat.com/blog/2020/07/02/build-a-simple-cloud-native-change-data-capture-pipeline>

变更数据捕获(CDC)是一种为系统建立的完善的软件设计模式，用于监控和捕获数据变更，以便其他软件可以响应这些事件。使用 KafkaConnect，以及 [Debezium 连接器](https://debezium.io)和 [Apache Camel Kafka 连接器](https://camel.apache.org/camel-kafka-connector/latest/index.html)，我们可以构建一个配置驱动的数据管道，以连接传统数据存储和新的[事件驱动架构](https://developers.redhat.com/topics/event-driven/)。

本文介绍了一个简单的例子。

## 为什么要使用变更数据捕获？

与简单的基于轮询或基于查询的流程相比，CDC 的优势在于:

*   **捕获所有更改**:否则可能会错过两次轮询循环之间的中间更改(即更新和删除)。
*   **低开销**:对数据变化的近乎实时的反应避免了因频繁轮询而增加的 CPU 负载。
*   **没有数据模型影响**:不再需要时间戳列来确定最后一次数据更新。

## 这个例子

在这个例子中，我们从头开始构建一个简单的云原生 CDC 管道。目标是将一个简单的`customers`表中的每一个变化发送到一个消息队列中，以便进一步处理。一旦提供了基础设施，我们将使用配置文件实现数据管道，而无需编写任何代码。这是体系结构的高级概述:

```
MySQL --> KafkaConnect [Worker0JVM(TaskA0, TaskB0, TaskB1),...] --> AMQ
                                |
                    Kafka (offsets, config, status)

```

即使这是一个简单的用例，它也包括在 OpenShift 4 之上部署和设置不同的集成产品:AMQ 7.7(消息代理)、AMQ 流 1.5 (Kafka 和 KafkaConnect)和 Debezium (CDC 引擎)。当将这个简单的例子扩展成一组通过发送事件相互通信的[微服务](https://developers.redhat.com/topics/microservices/)时，这个基础设施就有意义了。使用 Debezium 的好处是您不需要改变应用程序的逻辑。

你可以在这里找到所有需要的配置文件。

**注:**为了简单起见，我们使用不适合生产使用的不安全组件。您可能希望添加 TLS 加密设置，并为任何重要用途增加资源。

### 安装

为了给我们的 CDC 管道提供基础设施，我们必须建立 OpenShift 项目、MySQL 源系统、AMQ 汇系统、AMQ 流和 Debezium 连接器。您也可以使用[Red Hat code ready Containers(CRC)](https://developers.redhat.com/products/codeready-containers/overview)，但要确保保留八个内核和至少 14 GB 的内存。

我们将从命令行完成所有这些工作，所以请打开您最喜欢的 shell。没有添加任何提示，以便您可以轻松地复制和粘贴您在这里找到的代码。首先，设置您的环境变量:

```
API_ENDPOINT="https://api.crc.testing:6443"
ADMIN_NAME="kubeadmin"
ADMIN_PASS="7z6T5-qmTth-oxaoD-p3xQF"
USER_NAME="developer"
USER_PASS="developer"
PROJECT_NAME="cdc"

```

接下来，创建一个新项目:

```
TMP="/tmp/ocp" && rm -rf $TMP && mkdir -p $TMP
oc login -u $ADMIN_NAME -p $ADMIN_PASS $API_ENDPOINT
oc new-project $PROJECT_NAME
oc adm policy add-role-to-user admin $USER_NAME

```

然后设置 Red Hat 注册表认证(在此使用您自己的凭证):

```
REG_SECRET="registry-secret"
oc create secret docker-registry $REG_SECRET \
    --docker-server="registry.redhat.io" \
    --docker-username="my-user" \
    --docker-password="my-pass"
oc secrets link default $REG_SECRET --for=pull
oc secrets link builder $REG_SECRET --for=pull
oc secrets link deployer $REG_SECRET --for=pull

```

### MySQL 设置(源系统)

我们的源系统是 MySQL。这里，我们使用一个定制的 DeploymentConfig，它包含一个生命周期后挂钩来初始化我们的数据库，并为 Debezium 用户启用二进制日志访问。首先，创建您的配置图和密码:

```
oc create configmap db-config --from-file=./mysql/my.cnf
oc create configmap db-init --from-file=./mysql/initdb.sql
oc create secret generic db-creds \
    --from-literal=database-name=cdcdb \
    --from-literal=database-user=cdcadmin \
    --from-literal=database-password=cdcadmin \
    --from-literal=database-admin-password=cdcadmin

```

接下来，部署您的资源:

```
oc create -f ./mysql/my-mysql.yaml

```

检查状态:

```
MYSQL_POD=$(oc get pods | grep my-mysql | grep Running | cut -d " " -f1)
oc exec -i $MYSQL_POD -- /bin/sh -c 'MYSQL_PWD="cdcadmin" $MYSQL_PREFIX/bin/mysql -u cdcadmin cdcdb -e "SELECT * FROM customers"'

```

进行一些数据更改:

```
oc exec -i $MYSQL_POD -- /bin/sh -c 'MYSQL_PWD="cdcadmin" $MYSQL_PREFIX/bin/mysql -u cdcadmin cdcdb -e \
    "INSERT INTO customers (first_name, last_name, email) VALUES (\"John\", \"Doe\", \"jdoe@example.com\")"'
oc exec -i $MYSQL_POD -- /bin/sh -c 'MYSQL_PWD="cdcadmin" $MYSQL_PREFIX/bin/mysql -u cdcadmin cdcdb -e \
    "UPDATE customers SET first_name = \"Jane\" WHERE id = 1"'
oc exec -i $MYSQL_POD -- /bin/sh -c 'MYSQL_PWD="cdcadmin" $MYSQL_PREFIX/bin/mysql -u cdcadmin cdcdb -e \
    "INSERT INTO customers (first_name, last_name, email) VALUES (\"Chuck\", \"Norris\", \"cnorris@example.com\")"'

```

### AMQ 代理设置(接收系统)

我们的目标系统是 AMQ 消息代理，你可以从开发者门户免费下载红帽 AMQ 代理。这里，我们简单地创建了一个实例代理和我们的目标队列:

```
mkdir $TMP/amq
unzip -qq /path/to/amq-broker-operator-7.7.0-ocp-install-examples.zip -d $TMP/amq
AMQ_DEPLOY="$(find $TMP/amq -name "deploy" -type d)"

```

部署操作员:

```
oc create -f $AMQ_DEPLOY/service_account.yaml
oc create -f $AMQ_DEPLOY/role.yaml
oc create -f $AMQ_DEPLOY/role_binding.yaml
sed -i -e "s/v2alpha1/v2alpha2/g" $AMQ_DEPLOY/crds/broker_v2alpha1_activemqartemis_crd.yaml
sed -i -e "s/v2alpha1/v2alpha2/g" $AMQ_DEPLOY/crds/broker_v2alpha1_activemqartemisaddress_crd.yaml
oc apply -f $AMQ_DEPLOY/crds
oc secrets link amq-broker-operator $REG_SECRET --for=pull
oc create -f $AMQ_DEPLOY/operator.yaml

```

部署资源:

```
oc create -f ./amq/my-broker.yaml

```

将其配置为仅在 broker pod 运行时创建地址:

```
oc create -f ./amq/my-address.yaml

```

检查状态:

```
oc get activemqartemises
oc get activemqartemisaddresses

```

### AMQ 流设置(卡夫卡)

您还可以在同一个门户页面上下载红帽 AMQ 流，这是运行 KafkaConnect 所必需的。这里我们创建了一个只有一个节点的简单集群。这里不需要创建任何主题，因为 Debezium 会处理这些，用模式`serverName.databaseName.tableName`创建它的内部主题和我们的目标主题。

```
mkdir $TMP/streams
unzip -qq /path/to/amq-streams-1.5.0-ocp-install-examples.zip -d $TMP/streams
STREAMS_DEPLOY="$(find $TMP/streams -name "install" -type d)"

```

部署操作员:

```
sed -i -e "s/namespace: .*/namespace: $PROJECT_NAME/g" $STREAMS_DEPLOY/cluster-operator/*RoleBinding*.yaml
oc apply -f $STREAMS_DEPLOY/cluster-operator
oc set env deploy/strimzi-cluster-operator STRIMZI_NAMESPACE=$PROJECT_NAME
oc secrets link builder $REG_SECRET --for=pull
oc secrets link strimzi-cluster-operator $REG_SECRET --for=pull
oc set env deploy/strimzi-cluster-operator STRIMZI_IMAGE_PULL_SECRETS=$REG_SECRET
oc apply -f $STREAMS_DEPLOY/strimzi-admin
oc adm policy add-cluster-role-to-user strimzi-admin $USER_NAME

```

部署资源:

```
oc apply -f ./streams/my-kafka.yaml

```

检查状态:

```
oc get kafkas

```

### AMQ 流设置(KafkaConnect)

从同一个 AMQ 流包中，我们还可以部署我们的 KafkaConnect 自定义映像。我们将在官方版本的基础上构建它，添加我们特定的连接器插件。在这种情况下，我们将添加 Debezium MySQL 连接器和 Camel Kafka SJMS2 连接器(为了方便起见，我使用的是最新的上游版本，但您可以从门户下载 Red Hat 版本)。

**注意:**这个新的 Camel 子项目的好处是，您可以使用所有 300 多个组件作为 Kafka 连接器，与几乎任何外部系统集成。

一旦连接器启动并运行(查看来自`describe`命令的状态)，您就可以对`customers`表进行其他更改，并通过使用代理 web 控制台查看它们是否被传输到队列:

```
KAFKA_CLUSTER="my-kafka-cluster"
CONNECTOR_URLS=(
    "https://repo.maven.apache.org/maven2/io/debezium/debezium-connector-mysql/1.1.2.Final/debezium-connector-mysql-1.1.2.Final-plugin.zip"
    "https://repository.apache.org/content/groups/public/org/apache/camel/kafkaconnector/camel-sjms2-kafka-connector/0.3.0/camel-sjms2-kafka-connector-0.3.0-package.zip"
)

CONNECTORS="$TMP/connectors" && mkdir -p $CONNECTORS
for url in "${CONNECTOR_URLS[@]}"; do
    curl -sL $url -o $CONNECTORS/file.zip && unzip -qq $CONNECTORS/file.zip -d $CONNECTORS
done
sleep 2
rm -rf $CONNECTORS/file.zip

```

部署资源:

```
oc create secret generic debezium-config --from-file=./streams/connectors/mysql.properties
oc create secret generic camel-config --from-file=./streams/connectors/amq.properties
oc apply -f ./streams/my-connect-s2i.yaml

```

仅在连接群集运行时启动自定义映像构建:

```
oc start-build my-connect-cluster-connect --from-dir $CONNECTORS --follow

```

检查状态:

```
oc get kafkaconnects2i

```

到目前为止，这些都是正在运行的 pod:

```
amq-broker-operator-5d4559677-dpzf5                 1/1     Running     0          21m
my-broker-ss-0                                      1/1     Running     0          19m
my-connect-cluster-connect-2-xr58q                  1/1     Running     0          6m28s
my-kafka-cluster-entity-operator-56c9868474-kfdx2   3/3     Running     1          14m
my-kafka-cluster-kafka-0                            2/2     Running     0          14m
my-kafka-cluster-zookeeper-0                        1/1     Running     0          15m
my-mysql-1-vmbbw                                    1/1     Running     0          25m
strimzi-cluster-operator-666fcd8b96-q8thc           1/1     Running     0          15m

```

现在我们已经准备好了基础架构，我们终于可以配置 CDC 管道了:

```
oc apply -f ./streams/connectors/mysql-source.yaml
oc apply -f ./streams/connectors/amq-sink.yaml

```

检查状态:

```
oc get kafkaconnectors
oc describe kafkaconnector mysql-source
oc describe kafkaconnector amq-sink
CONNECT_POD=$(oc get pods | grep my-connect-cluster | grep Running | cut -d " " -f1)
oc logs $CONNECT_POD

oc get kafkatopics
oc exec -i $KAFKA_CLUSTER-kafka-0 -c kafka -- bin/kafka-console-consumer.sh \
    --bootstrap-server my-kafka-cluster-kafka-bootstrap:9092 --topic my-mysql.cdcdb.customers --from-beginning

```

打开 AMQ web 控制台路由以检查队列的内容:

```
echo http://$(oc get routes my-broker-wconsj-0-svc-rte -o=jsonpath='{.status.ingress[0].host}{"\n"}')/console

```

## 清除

当您完成对新的云原生 CDC 管道的试验后，您可以使用以下命令删除整个项目并释放一些资源:

```
rm -rf $TMP
oc delete project $PROJECT_NAME
oc delete crd/activemqartemises.broker.amq.io
oc delete crd/activemqartemisaddresses.broker.amq.io
oc delete crd/activemqartemisscaledowns.broker.amq.io
oc delete crd -l app=strimzi
oc delete clusterrolebinding -l app=strimzi
oc delete clusterrole -l app=strimzi

```

## 考虑

无论使用何种技术，变更数据捕获过程都必须作为单线程运行，以保持有序。由于 Debezium 异步记录日志偏移量，这些更改的最终消费者必须是[幂等的](https://en.wikipedia.org/wiki/Idempotence)。

以分布式模式运行在 KafkaConnect 之上的一个好处是，您拥有一个容错的 CDC 进程。Debezium 提供了很好的性能，因为它可以访问数据源的内部事务日志，但是它没有标准，所以对实现的更改可能需要插件重写。这也意味着每个数据源都有自己的访问事务日志的过程。

连接器配置允许您通过使用[单消息转换(SMTs)](https://kafka.apache.org/documentation/#connect_transforms) 来转换消息负载。这些可以用自定义实现来链接和扩展，但它们实际上是为简单的修改而设计的。长的 SMT 链很难维护和理解。此外，转换是同步的，并应用于每个消息，因此您可以通过繁重的处理或外部服务调用来降低流管道的速度。

如果您需要进行繁重的处理、分割、丰富、聚合记录或调用外部服务，您应该在连接器之间使用流处理层，如 Kafka Streams 或只是 [plain Camel](https://github.com/fvaleri/cdc/tree/blog/camel-cdc) 。请记住，Kafka Streams 创建内部主题，您被迫将转换后的数据放回 Kafka(数据复制)，而这种方法只是使用 Camel 时的一种选择。

*Last updated: June 26, 2020*