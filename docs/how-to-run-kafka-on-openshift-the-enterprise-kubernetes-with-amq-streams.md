# 如何在 Openshift 上运行 Kafka，企业 Kubernetes 与 AMQ 流

> 原文：<https://developers.redhat.com/blog/2018/10/29/how-to-run-kafka-on-openshift-the-enterprise-kubernetes-with-amq-streams>

10 月 25 日红帽 [宣布](https://access.redhat.com/announcements/3667151) 他们的 AMQ 流 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 运营商为阿帕奇 Kafka。 [红帽 AMQ 流](https://access.redhat.com/products/red-hat-amq-streams) 专注于在 Openshift 上运行 Apache Kafka，提供大规模可扩展、分布式、高性能的数据流平台。基于 Apache Kafka 和[strim zi](http://strimzi.io/)项目的 AMQ 流提供了一个分布式主干，允许[微服务](https://developers.redhat.com/topics/microservices/)和其他应用以极高的吞吐量共享数据。该主干支持:

*   **发布和订阅:** 以容错、持久的方式进行多对多传播。
*   **可回放事件:** 作为微服务的存储库，构建源数据的内存副本，直到任意时间点。
*   **长期数据保留:** 高效地存储数据，以便即时访问，只受磁盘空间的限制。
*   **对消息进行分区，以实现更大的横向可伸缩性:** 允许组织消息以实现最大的并发访问。

开发人员和架构师最常问的一个问题是如何开始使用一个简单的部署选项进行测试。在本指南中，我们将使用基于 minishift 的[Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/)，在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 上启动一个 Apache Kafka 集群。

要在 Openshift 上从头开始设置 Kafka 集群，请遵循以下步骤或观看此短片:

https://vimeo.com/middlewarepro/amq-streams-getting-started

https://vimeo.com/302934145

## 设置 Red Hat 容器开发套件(CDK)

1.  如果你以前没有在你的笔记本电脑上设置过 CDK (minishift)。
    1.  [下载 CDK](https://developers.redhat.com/products/cdk/download/) 。
    2.  按照[Hello World](https://developers.redhat.com/products/cdk/hello-world)安装配置 CDK。
2.  最新版本的 CDK 利用了配置文件的概念，因此我们将使用它们来避免更改其他配置。创建新的`streams`档案:

T2`$ minishift profile set streams`

3.  在这个新配置文件中配置系统要求(我们建议 8GB 和至少 2 个 vCPUs 可用于平稳运行:

【T2`$ minishift config set cpus 2`
`$ minishift config set memory 8192`

4.  在我的例子中，我使用 VirtualBox 作为虚拟机驱动程序，替换您正在使用的虚拟机管理程序:

T2`$ minishift config set vm-driver virtualbox`

5.  由于 Zookeeper 对用户的依赖性，我们将需要移除 CDK 自带的 anyuid 附加组件:

T2`$ minishift addons disable anyuid`

注意:如果你正在运行 CDK，这是关键的一步。如果插件没有被禁用，你会得到一个错误时，试图启动 Zookeeper TLS 边车。

6.  启动 CDK 环境

T2`$ minishift start`

如果一切正常，您将看到以下输出:

```
OpenShift server started.
The server is accessible via web console at:

 https://192.168.99.100:8443

You are logged in as:

 User:     developer
 Password: <any value>

To login as administrator:

 oc login -u system:admin

-- Applying addon 'xpaas':..
XPaaS imagestream and templates for OpenShift installed
See https://github.com/openshift/openshift-ansible/tree/release-3.10/roles/openshift_examples/files/examples/v3.10
-- Applying addon 'admin-user':..
-- Exporting of OpenShift images is occuring in background process with pid 35470.
```

## 设置 AMQ 流

1.  [从](https://access.redhat.com/node/3667151/423/0) 红帽客户门户 下载红帽 AMQ 流安装和示例资源。
2.  导航到解压后的文件夹，以访问 yaml 文件
3.  解压下载的`install_and_examples_0.zip`文件。

T2`$ cd <your_download_folder>/install_and_examples_0`

2.  以管理员权限登录 OpenShift 集群:

T2`$ oc login -u system:admin`

3.  应用客户资源定义(CRD)和管理 CRD 所需的角色绑定。

T2`$ oc apply -f install/cluster-operator/`

4.  最后一步将创建 Kafka CRD，并开始部署集群操作员。该操作员将跟踪您的 kafka 资源，并提供或更新对这些资源的更改。打开一个新的浏览器选项卡，导航到您的 web 控制台 URL:

T2`https://<your-ip>:8443/console/project/myproject/overview`

检查发出`minishift ip`命令的指定 IP，或者运行`minishift console`并导航至`My Project`。

5.  登录到 OpenShift web 控制台查看部署情况。使用`developer` / `developer`作为用户名和密码。如果您之前没有这样做，请在浏览器中接受自签名证书。 ![](img/b00835a1a508a1025d739236ed8c1d47.png)
6.  您将在项目工作区中看到新部署的集群操作符正在运行。【T2![](img/fa8a9f16f1862d5bfc59b66d3ae9891b.png)

## 设置您的第一个 Apache Kafka 集群

集群运营商现在将监听新的 Kafka 资源。让我们创建一个配置了外部访问的简单 Kafka 集群，这样我们就能够从 OpenShift 集群外部进行连接。

1.  使用`ephemeral`存储:

    ```
    $ cat << EOF | oc create -f -
    apiVersion: kafka.strimzi.io/v1alpha1
    kind: Kafka
    metadata: 
     name: my-cluster
    spec:
     kafka:
     replicas: 3
     listeners:
     external:
     type: route
     storage:
     type: ephemeral
     zookeeper:
     replicas: 3
     storage:
     type: ephemeral
     entityOperator:
     topicOperator: {}
    EOF
    ```

    创建带有 3 个 zookeeper 和 3 个 kafka 节点的新`my-cluster` kafka 集群
2.  等待几分钟，之后你会看到 Zookeeper 和 Kafka 资源以及主题操作符的部署。【T2![](img/d39f093d0771dc8a8f36da5e50331c63.png)
3.  现在我们的集群正在运行，我们可以创建一个主题来从我们的外部客户端发布和订阅。在`my-cluster` Kafka 集群中使用 3 个副本和 3 个分区创建以下`my-topic`主题自定义资源定义:

```
$ cat << EOF | oc create -f -
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaTopic
metadata:
 name: my-topic
 labels:
 strimzi.io/cluster: "my-cluster"
spec:
 partitions: 3
 replicas: 3
EOF
```

现在，您可以开始发送和接收信息了。

## 使用外部应用程序进行测试

1.  克隆这个 [git repo](https://github.com/hguerrero/amq-examples.git) 来测试从到您的新 Kafka 集群的访问:

T2`$ git clone https://github.com/hguerrero/amq-examples.git`

2.  切换到`camel-kafka-demo`文件夹

T2`$ cd amq-examples/camel-kafka-demo/`

3.  由于我们使用**路由**从外部访问集群，我们需要 CA 证书来启用客户端中的 TLS。提取经纪人认证机构的公共证书

T2`$ oc extract secret/my-cluster-cluster-ca-cert --keys=ca.crt --to=- > src/main/resources/ca.crt`

4.  将可信证书导入密钥库

T2`$ keytool -import -trustcacerts -alias root -file src/main/resources/ca.crt -keystore src/main/resources/keystore.jks -storepass password -noprompt`

5.  现在您可以使用 maven 命令运行 Fuse 应用程序:

T2`$ mvn -Drun.jvmArguments="-Dbootstrap.server=`oc get routes my-cluster-kafka-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'`:443" clean package spring-boot:run`

完成清理和打包阶段后，您将看到 Spring Boot 应用程序开始创建一个生产者和消费者，从“我的主题”Kafka 主题发送和接收消息。

```
14:36:18.170 [main] INFO  com.redhat.kafkademo.Application - Started Application in 12.051 seconds (JVM running for 12.917)
14:36:18.490 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  o.a.k.c.c.i.AbstractCoordinator - [Consumer clientId=consumer-1, groupId=6de87ffa-c7cf-441b-b1f8-e55daabc8d12] Discovered coordinator my-cluster-kafka-1-myproject.192.168.99.100.nip.io:443 (id: 2147483646 rack: null)
14:36:18.498 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  o.a.k.c.c.i.ConsumerCoordinator - [Consumer clientId=consumer-1, groupId=6de87ffa-c7cf-441b-b1f8-e55daabc8d12] Revoking previously assigned partitions []
14:36:18.498 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  o.a.k.c.c.i.AbstractCoordinator - [Consumer clientId=consumer-1, groupId=6de87ffa-c7cf-441b-b1f8-e55daabc8d12] (Re-)joining group
14:36:19.070 [Camel (MyCamel) thread #3 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-2
14:36:19.987 [Camel (MyCamel) thread #4 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-4
14:36:20.982 [Camel (MyCamel) thread #5 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-6
14:36:21.620 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  o.a.k.c.c.i.AbstractCoordinator - [Consumer clientId=consumer-1, groupId=6de87ffa-c7cf-441b-b1f8-e55daabc8d12] Successfully joined group with generation 1
14:36:21.621 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  o.a.k.c.c.i.ConsumerCoordinator - [Consumer clientId=consumer-1, groupId=6de87ffa-c7cf-441b-b1f8-e55daabc8d12] Setting newly assigned partitions [my-topic-0, my-topic-1, my-topic-2]
14:36:21.959 [Camel (MyCamel) thread #6 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-8
14:36:21.973 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  consumer-route - consumer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-8
14:36:22.970 [Camel (MyCamel) thread #7 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-11
14:36:22.975 [Camel (MyCamel) thread #1 - KafkaConsumer[my-topic]] INFO  consumer-route - consumer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-11
14:36:23.968 [Camel (MyCamel) thread #8 - KafkaProducer[my-topic]] INFO  producer-route - producer >>> Hello World from camel-context.xml with ID ID-hguerrer-osx-1540578972584-0-14

```

大功告成！按`Ctrl + C`停止正在运行的程序。

您已经看到了在 OpenShift 中创建一个 Apache Kafka 集群，并准备好让您的应用程序使用它发送和消费消息是多么容易。如果您想查看更多高级配置，可以在官方入门指南中找到更多信息。

不久，我将发表另一篇如何配置 Kafka Connect 和 Kafka Streams 与 OpenShift 和 AMQ 流的文章。

*Last updated: December 21, 2021*