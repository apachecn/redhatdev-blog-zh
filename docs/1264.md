# OpenShift 中的结构化应用程序日志

> 原文：<https://developers.redhat.com/blog/2018/01/22/openshift-structured-application-logs>

原木如金粉。单独来看，它们可能不值钱，但是放在一起，经过熟练的金匠的加工，它们可能会变得非常值钱。OpenShift 附带了[EFK 堆栈](https://docs.openshift.org/latest/install_config/aggregate_logging.html) : Elasticsearch、Fluentd 和 Kibana。在 OpenShift 上运行的应用程序会自动汇总它们的日志，以便在测试和生产过程中提供关于它们的状态和健康状况的有价值的信息。

唯一的要求是应用程序将其日志发送到标准输出。OpenShift 完成剩下的工作。够简单！

在这篇博客中，我将介绍几个要点，这些要点可能会帮助你把你的日志从原材料变成更有价值的产品。

## 配置文件外部化

![](img/bdcba97ea0849e097fb69acde712d587.png)

第一步旨在通过在需要时轻松更改日志属性和级别来提供一定程度的灵活性。因此，日志配置需要外部化。大多数 Java 应用程序使用 Log4j 或 Logback 之类的日志框架，这些框架可以通过 XML 或属性文件进行配置。

在 OpenShift 中运行应用程序时，您不希望为了更改日志配置而重新构建容器映像。这可以通过使用一个 [configMap](https://docs.openshift.org/latest/dev_guide/configmaps.html) 很容易地避免，它与日志配置文件一起挂载到容器文件系统中。

在 Spring Boot 应用程序中，您只需在 application.properties 中指定日志配置文件的位置。

```
logging.config=file:/opt/configuration/logback.xml
```

创建配置图就像一个命令行一样简单:

```
$ oc create configmap logconfig --from-file=./src/main/resources/logback.xml
```

然后需要编辑部署配置，以便将配置映射装载到容器文件系统中:

```
[...]
volumeMounts:
- name: config-volume
  mountPath: /opt/configuration
volumes:
- name: config-volume
  configMap:
    name: logconfig 
```

您可能还希望在不需要重启应用程序/容器的情况下应用日志配置更改。在 OpenShift 中的配置映射中所做的更改会自动反映到容器文件系统中。您只需要告诉您的应用程序检查更改。这可以通过在 logback.xml 中设置 scan=true 来完成。默认扫描周期是一分钟。

## 结构化日志

![](img/7b940aad8a535def10d37222902f93b5.png)

Fluentd 使用索引名“project”将日志转发给 Elasticsearch。{project_name}。{project_uuid}.YYYY.MM.DD "根据[文档](https://docs.openshift.org/latest/install_config/aggregate_logging.html#aggregate-logging-performing-elasticsearch-maintenance-operations)。索引是自动创建的，日志内容通过有用的元数据得到了增强，比如容器、pod 和项目的名称，以及收集时间戳。

这一切都很好，并且从应用程序团队那里拿走了很多工作。但是，您可能还希望构建您编写的日志消息，以便过滤器可以索引、查询和使用其他字段。 ****

为此， [logstash-logback-encoder](https://github.com/logstash/logstash-logback-encoder) 可与 logback 一起使用。它使您能够创建 JSON 格式的日志。使用 Log4j 你只需要使用 [JSON 布局](https://logging.apache.org/log4j/2.x/manual/layouts.html#JSONLayout)。

Logback 的 JSON 库需要添加到项目 pom 中。如果使用 Spring Boot，则需要排除由 Spring Boot 启动器导入的回退依赖项的版本，因为它们是旧版本:

```
<properties>
[...]
  <ch.qos.logback.version>1.2.3</ch.qos.logback.version>
</properties>

<dependencyManagement>
  <dependencies>
[...]
  <!-- Use the latest versions compatible with logstash-logback-encoder 4.11 -->
  <!-- Older versions may be imported by Spring Boot starters that haven't been tested with 4.11 -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>${ch.qos.logback.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${ch.qos.logback.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-access</artifactId>
      <version>${ch.qos.logback.version}</version>
    </dependency>
[...] 
```

然后，Logback 配置可以使用 JSON 编码器:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE xml>
<configuration scan="true" scanPeriod="30 seconds" >
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
  </appender>
  <root level="info">
    <appender-ref ref="STDOUT" />
  </root>
</configuration> 
```

下面是你在 Elasticsearch 默认得到的结果。_source 字段可以通过修改默认模板来添加或删除。详情请见[此处](https://github.com/logstash/logstash-logback-encoder/tree/master):

```
{
  "_index": "project.test.ac2664e3-1398-11e7-acfe-5254005a078f.2018.01.14",
  "_type": "com.redhat.viaq.common",
  "_id": "AWDz2VVS0wWjxf1Icmz5",
  "_score": null,
  "_source": {
    "@timestamp": "2018-01-14T08:46:33.146088894Z",
    "@version": 1,
    "message": "Connector vm://localhost stopped",
    "logger_name": "org.apache.activemq.broker.TransportConnector",
    "thread_name": "http-nio-0.0.0.0-8081-exec-9",
    "level": "info",
    "level_value": 20000,
    "docker": {
      "container_id": "10ba060d704c1153165b0db0b38f2d294e3a2df9d7e5447023b710ac79c22d72"
    },
    "kubernetes": {
      "container_name": "spring-boot",
      "namespace_name": "test",
      "pod_name": "spring-boot-camel-amq-te-5-pphb6",
      "pod_id": "3063a039-f907-11e7-a821-5254005a078f",
      "labels": {
        "deployment": "spring-boot-camel-amq-te-5",
        "deploymentconfig": "spring-boot-camel-amq-te",
        "group": "org.jboss.fuse.fis.archetypes",
        "project": "spring-boot-camel-amq-testing",
        "provider": "fabric8",
        "version": "2.2.195.redhat-000013"
      },
      "host": "ocpnode3.sandbox.com",
      "master_url": "https://kubernetes.default.svc.cluster.local",
      "namespace_id": "ac2664e3-1398-11e7-acfe-5254005a078f"
    },
    "hostname": "ocpnode3.sandbox.com",
    "pipeline_metadata": {
      "collector": {
        "ipaddr4": "10.129.0.51",
        "ipaddr6": "fe80::858:aff:fe81:33",
        "inputname": "fluent-plugin-systemd",
        "name": "fluentd",
        "received_at": "2018-01-14T08:46:34.054763+00:00",
        "version": "0.12.39 1.6.0"
      }
    }
  },
  "fields": {
    "@timestamp": [
      1515919593146
    ],
    "pipeline_metadata.collector.received_at": [
      1515919594054
    ]
  },
  "sort": [
   1515919593146
  ]
} 
```

进一步，您可以使用 logstash encoder 通过使用[结构化参数或标记](https://github.com/logstash/logstash-logback-encoder#custom-fields)来添加结构化字段。这里有一个例子:

```
net.logstash.logback.marker.Markers.*

[...]

LOG.info(append("MyfirstField", myFirstValue).and(append("MySecondField", mySecondValue),"my message"); 
```

## 索引

![](img/5190812e619178a779c47ad505cce3ef.png)

但是，如果您发送结构化日志，您可能需要微调索引。关于如何访问 Elasticsearch API 的解释在 [OpenShift 文档](https://docs.openshift.com/container-platform/3.7/install_config/aggregate_logging.html#aggregate-logging-performing-elasticsearch-maintenance-operations)中提供。第一步是用“oc rsh”登录一个 Elasticsearch 容器。然后可以执行管理操作，例如配置字段:

```
curl -XPUT 'localhost:9200/my_index/_mapping/type_one?update_all_types&pretty' \
    -H 'Content-Type: application/json' -d '{ \
    "properties": { \
      "text": { \
        "type": "text", \
        "analyzer": "standard", \
        "search_analyzer": "whitespace" \
      } \
    } \
}' 
```

由于每天都有新的索引创建，所以最好通过一个[索引模板](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/indices-templates.html)来应用修改，而不是直接应用到一个现有的索引。这意味着修改在第二天之前是不可见的，但是也会应用到随后几天创建的指数中。尝试当前的索引，然后在对结果满意时转移到一个索引模板，这似乎是一个不错的方法。

最好是创建一个新的索引模板，而不是修改 OpenShift 提供的[模板，也可以通过查询 Elasticsearch 检索，因为 Elasticsearch 支持](https://github.com/openshift/origin-aggregated-logging/blob/master/elasticsearch/index_templates/com.redhat.viaq-openshift-project.template.json)[多个模板匹配](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/indices-templates.html#multiple-templates)。

现在，您应该在 Elasticsearch 中获得了不错的数据，并可以开始在 Kibana 中创建搜索、可视化和仪表板。

*Last updated: October 18, 2018*