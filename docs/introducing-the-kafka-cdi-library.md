# 介绍卡夫卡-CDI 图书馆

> 原文：<https://developers.redhat.com/blog/2018/05/31/introducing-the-kafka-cdi-library>

在现代事件驱动的应用程序中使用 Apache Kafka 非常流行。为了更好地体验 Apache Kafka 的云原生，强烈建议查看 Red Hat [AMQ 流](https://developers.redhat.com/blog/2018/05/07/announcing-amq-streams-apache-kafka-on-openshift/)，它在 Red Hat OpenShift 上提供了 Apache Kafka 集群的简单安装和管理。

本文展示了 Kafka-CDI 库如何处理困难的设置任务，并使为 MicroProfile 和 Jakarta EE 创建 Kafka 驱动的事件驱动应用程序变得非常容易。

## 香草阿帕奇卡夫卡消费者

虽然 Kafka 生产者和消费者的概念很简单，但是编写实际的代码可能相当麻烦，并且需要一些样板代码，如下所示:

```
final Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "my-cluster-kafka:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "demo-group");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
final KafkaConsumer<String, String> consumer = new KafkaConsumer(props); 
...
consumer.subscribe(Collections.singleton("topic"));
...
final ConsumerRecords<String, String> records = consumer.poll(500);
for (final ConsumerRecord<String, String<> record : records) {
    logger.info(record.value());
}

```

上面的代码显示了一个`KafkaConsumer`的简单配置，它留给开发人员一些任务，比如手动定义键的实际类型和消费记录的值。除此之外，消费者也不是线程安全的，开发人员需要使用 Java 并发 API 来解决这个问题。

## CDI 扩展到救援

在各种企业 Java 社区中，例如 [Eclipse MicroProfile](https://microprofile.io/) 或 [Jakarta EE](https://jakarta.ee/) ，CDI 是管理依赖项及其配置的自然选择。该 API 还允许开发人员创建能够利用整个“Java EE”生命周期及其强大平台 API 的扩展。

AeroGear 项目的 Kafka-CDI 库就是这样一个扩展，它使得为 MicroProfile 或 Jakarta EE 创建基于 Kafka 的应用程序变得非常容易。

## 卡夫卡-CDI 在行动

设置很简单，只需要几行 Maven 坐标:

```
<dependency>
  <groupId>org.aerogear.kafka</groupId>
  <artifactId>kafka-cdi-extension</artifactId>
  <version>0.1.0</version>
</dependency>
```

## CDI 消费者

创建作为消息消费者的 CDI 管理的 beans 现在非常容易，只需要很少的代码:

```
@KafkaConfig(bootstrapServers ="#{KAFKA_SERVICE_HOST}:#{KAFKA_SERVICE_PORT}")
public class MyAwesomeConsumer {

    private static final Logger LOGGER = LoggerFactory.getLogger(MyAwesomeConsumer.class);

    @Consumer(topics = "topic", groupId = "demo-group")
    public void onMessage(final String key, final String value) {
        LOGGER.info("We got this value: " + value);
    }
}
```

通过一个`@Consumer`注释，bean 就变成了消息消费者。Kafka-CDI 扩展处理所有的配置，比如 Kafka 记录的键和值的类型反序列化，以及幕后的线程处理。对于每个应用程序，需要一个`KafkaConfig`注释来标识可用的引导服务器列表。在这个例子中，`topic`、`groupId`和`bootstrapServers`的值可以使用环境变量或系统属性来解析。

## CDI 生产商

编写生产者也很容易:

```
@Producer
private SimpleKafkaProducer<String, String> myproducer;
...
myproducer.send("topic", myKey, myValue);

```

任何 bean 都可以被注入一个方便的生产者(`SimpleKafkaProducer`)，该生产者可以在任何向 Kafka 集群发送消息的方法中使用。与消费者示例类似，CDI 扩展处理键和值的类型序列化。

## 带 CDI 的 KStreams

最后但同样重要的是，该库对使用 [Kafka Streams](http://kafka.apache.org/11/documentation/streams/) API 提供了一些初始支持:

```
@KafkaStream(input = "input.topic", output = "output.topic")
public KTable<String, Long> successfullMessagesPerJobTransformer(KStream<String, String> source) {
  return successCountsPerJob = source.filter((key, value) -> value.equals("Success"))
    .groupByKey()
    .count("successCounter");
}
```

当用`@KafkaStream`注释一个方法时，库通过执行注释的方法来处理传递的`KStream`对象。来自`@KafkaStream`注释的`input`和`output`属性告诉库从哪个主题读取，以及流处理作业应该写入哪个主题。为了方便开发人员，类型序列化和反序列化再次由框架处理。

## 支持的类型

该库支持 JSON 序列化和反序列化的各种方式。它带有用于`javax.json.Json`类型的`Serializer`和`Deserializer`，并且它有一个自动使用 Jackson 库用于任何未知类型的回退机制。对 Apache Avro 的支持还没有实现，但是已经在特性路线图上了。

# 结论

本文是对 Kafka-CDI 图书馆的介绍。开始使用这个库只需要几行代码。重点是用户简单性，而所有困难的设置任务都由 CDI 扩展处理。

通过使用这个库，使用微概要实现如 [Thorntail](https://thorntail.io/) 来编写事件驱动的应用程序是显而易见的。

*感谢 Gunnar Morling 的宝贵意见！*

*Last updated: August 21, 2019*