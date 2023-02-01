# 使用红帽 AMQ 流构建 Apache Kafka 流应用程序:第 1 部分

> 原文：<https://developers.redhat.com/blog/2019/06/17/building-apache-kafka-streams-applications-using-red-hat-amq-streams-part-1>

这个 [Apache Kafka](https://developers.redhat.com/videos/youtube/CZhOJ_ysIiI/) 项目包括一个[流](https://kafka.apache.org/22/documentation/streams/developer-guide/dsl-api.html) [域特定语言(](https://kafka.apache.org/22/documentation/streams/developer-guide/dsl-api.html) [DSL)](https://kafka.apache.org/22/documentation/streams/developer-guide/dsl-api.html) 构建在底层[流处理器 API](https://kafka.apache.org/10/documentation/streams/developer-guide/processor-api.html) 之上。这个 DSL 为开发人员提供了执行数据处理操作的简单抽象。然而，如何用 Kafka 在容器化的环境中构建流处理管道还不清楚。这个由两部分组成的系列文章描述了使用[红帽 AMQ 流](https://developers.redhat.com/products/amq/overview)构建自己的 Apache Kafka 流应用程序的步骤。

在本系列的第一部分中，我们提供了一个简单的例子作为第二部分的构建模块。首先，要设置 AMQ 流，请按照 [AMQ 流入门文档](https://access.redhat.com/documentation/en-us/red_hat_amq/7.3/html/using_amq_streams_on_openshift_container_platform/getting-started-str)中的说明部署 Kafka 集群。本文档的其余部分假设您遵循了这些步骤。

## 创建一个生产者

首先，我们需要一个流入卡夫卡的数据流。这些数据可能来自各种不同的来源，但出于本例的目的，让我们使用延迟发送的`String`生成样本数据。

接下来，我们需要配置 Kafka producer，以便它与 Kafka 代理对话(参见本文中的[以获得更深入的解释)，以及提供要写入的主题名称和其他信息。由于我们的应用程序将被容器化，我们可以将这个过程从内部逻辑中抽象出来，并从环境变量中读取。这种读取可以在单独的配置类中完成，但是对于这个简单的示例，以下内容就足够了:](https://developers.redhat.com/blog/?p=601077)

```
private static final String DEFAULT_BOOTSTRAP_SERVERS = "localhost:9092";
private static final int DEFAULT_DELAY = 1000;
private static final String DEFAULT_TOPIC = "source-topic";

private static final String BOOTSTRAP_SERVERS = "BOOTSTRAP_SERVERS";
private static final String DELAY = "DELAY";
private static final String TOPIC = "TOPIC";

private static final String ACKS = "1";

String bootstrapServers = System.getenv().getOrDefault(BOOTSTRAP_SERVERS, DEFAULT_BOOTSTRAP_SERVERS);
long delay = Long.parseLong(System.getenv().getOrDefault(DELAY, String.valueOf(DEFAULT_DELAY)));
String topic = System.getenv().getOrDefault(TOPIC, DEFAULT_TOPIC);

```

我们现在可以使用环境变量(或者我们设置的默认值)为 Kafka 生成器创建适当的属性:

```
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
props.put(ProducerConfig.ACKS_CONFIG, ACKS);
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

```

接下来，我们使用配置属性创建生成器:

```
KafkaProducer<String,String> producer = new KafkaProducer<>(props);

```

我们现在可以将数据流传输到提供的`topic`，在每个消息之间分配所需的`delay`:

```
int i = 0;
while (true) {
    String value = String.format("hello world %d", i);
    ProducerRecord<String, String> record = new ProducerRecord<>(topic, value);
    log.info("Sending record {}", record);
    producer.send(record);
    i++;
    try {
        Thread.sleep(delay);
    } catch (InterruptedException e) {
        // sleep interrupted, continue
    }
}

```

既然我们的应用程序已经完成，我们需要将它打包成一个 [Docker](https://developers.redhat.com/jboss-docker/) 映像，并推送到 [Docker Hub](https://hub.docker.com) 。然后，我们可以使用这个映像在我们的 [Kubernetes](https://kubernetes.io) 集群中创建一个新的部署，并通过 YAML 传递环境变量:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: basic-example
  name: basic-producer
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: basic-producer
    spec:
      containers:
      - name: basic-producer
        image: <docker-user>/basic-producer:latest
        env:
          - name: BOOTSTRAP_SERVERS
            value: my-cluster-kafka-bootstrap:9092
          - name: DELAY
            value: 1000
          - name: TOPIC
            value: source-topic

```

我们可以通过消费数据来检查数据是否到达我们的 Kafka 主题:

```
$ oc run kafka-consumer -ti --image=registry.access.redhat.com/amq7/amq-streams-kafka:1.1.0-kafka-2.1.1 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic source-topic --from-beginning

```

输出应该如下所示:

```
  hello world 0
  hello world 1
  hello world 2
  ...

```

## 创建流应用程序

Kafka Streams 应用程序通常从一个或多个输入主题读取/获取数据，并向一个或多个输出主题写入/发送数据，充当流处理器。我们可以像以前一样设置属性和配置，但是这次我们需要指定一个`SOURCE_TOPIC`和一个`SINK_TOPIC`。

首先，我们创建`source`流:

```
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> source = builder.stream(sourceTopic);

```

我们现在可以对这个`source`流执行操作。例如，我们可以创建一个名为`filtered`的新流，它只包含具有偶数索引的记录:

```
KStream<String, String> filtered = source
        .filter((key, value) -> {
            int i = Integer.parseInt(value.split(" ")[2]);
            return (i % 2) == 0;
        });

```

这个新的流然后可以输出到`sinkTopic`:

```
filtered.to(sinkTopic);

```

我们现在已经创建了定义我们的流应用程序操作的拓扑，但是我们还没有让它运行。让它运行需要创建 streams 对象，设置一个关闭处理程序，并启动流:

```
final KafkaStreams streams = new KafkaStreams(builder.build(), props);
final CountDownLatch latch = new CountDownLatch(1);

// attach shutdown handler to catch control-c
Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
    @Override
    public void run() {
        streams.close();
        latch.countDown();
    }
});

try {
    streams.start();
    latch.await();
} catch (Throwable e) {
    System.exit(1);
}
System.exit(0);

```

给你。让您的 Streams 应用程序运行起来就是这么简单。将应用程序构建到 Docker 映像中，以类似于生产者的方式进行部署，您可以观察输出的`SINK_TOPIC`。

### 阅读更多

使用红帽 AMQ 流构建 Apache Kafka 流应用程序:第 2 部分

*Last updated: September 3, 2019*