# OpenShift 上 AMQ 流的 Hello World

> 原文：<https://developers.redhat.com/products/amq/hello-world-amq-streams-openshift>

## 安装 AMQ 流自定义资源定义

### 下载并提取

1.  下载最新版本的红帽 AMQ 流[安装示例](https://developers.redhat.com/download-manager/file/amq-streams-1.6.0-ocp-install-examples.zip)。在撰写本文时，最新版本是红帽 AMQ 流 1.6。

2.  将红帽 AMQ 流安装和示例资源解压缩到任何目的地。

    *   在 Windows 或 Mac 上，您可以通过双击 ZIP 文件来提取 ZIP 存档的内容。

    *   在 Red Hat Enterprise Linux 上，在目标机器中打开一个终端窗口，并导航到 ZIP 文件的下载位置。通过执行以下命令提取 ZIP 文件:

        `# unzip amq-streams-1.6.0-ocp-install-examples.zip`

### 安装自定义资源定义(CRD)

**安装自定义资源定义**

1.  以集群管理员权限登录 OpenShift 集群，例如:

    `# oc login -u system:admin`

2.  默认情况下，安装文件在`myproject`名称空间中工作。根据您将安装 AMQ 流 Kafka 集群操作符的名称空间修改安装文件，例如`kafka`。

    *   在 Linux 上，使用:

        `# sed -i 's/namespace: .*/namespace: *kafka*/' install/cluster-operator/*RoleBinding*.yaml`

    *   在 Mac 上:

        `# sed -i '' 's/namespace: .*/namespace: *kafka*/' install/cluster-operator/*RoleBinding*.yaml`

3.  部署自定义资源定义(CRD)和基于角色的访问控制(RBAC)资源来管理 CRD。

    `# oc new-project kafka`
    

4.  创建您想要部署 Kafka 集群的项目，例如`my-kafka-project`。

    `# oc new-project my-kafka-project`

5.  授予您的非管理员用户`developer`访问权限。

    `# oc adm policy add-role-to-user admin developer -n my-kafka-project`

6.  使集群操作员能够监视该名称空间。

    `# oc set env deploy/strimzi-cluster-operator STRIMZI_NAMESPACE=kafka,my-kafka-project -n kafka
    # oc apply -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml -n my-kafka-project
    # oc apply -f install/cluster-operator/032-RoleBinding-strimzi-cluster-operator-topic-operator-delegation.yaml -n my-kafka-project
    # oc apply -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml -n my-kafka-project`

7.  创建新的集群角色`strimzi-admin`。

    ```
    # oc apply -f install/strimzi-admin/
    ```

8.  向非管理员用户`developer`添加角色。

    `# oc adm policy add-cluster-role-to-user strimzi-admin developer`

## 创建您的第一个 Apache Kafka 集群

### 创建集群和主题资源

集群操作员现在将监听新的 Kafka 资源。

1.  以普通用户身份登录，例如:

    `# oc login -u developer`

    `# oc project my-kafka-project`

2.  使用`ephemeral`存储创建具有 3 个 zookeeper 和 3 个 broker 节点的新`my-cluster` Kafka 集群，并使用路由将 Kafka 集群暴露在 OpenShift 集群之外:

    ```
    # cat << EOF | oc create -f -
    apiVersion: kafka.strimzi.io/v1beta1
    kind: Kafka
    metadata: 
      name: my-cluster
    spec:
      kafka:
        replicas: 3
        listeners:
          - name: external
            port: 9092
            type: route
            tls: true
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

3.  现在我们的集群正在运行，我们可以创建一个主题来从我们的外部客户端发布和订阅。在`my-cluster` Kafka 集群中使用 3 个副本和 3 个分区创建以下`my-topic`主题自定义资源定义:

    ```
    # cat << EOF | oc create -f -
    apiVersion: kafka.strimzi.io/v1beta1
    kind: KafkaTopic
    metadata:
      name: my-topic
      labels:
        strimzi.io/cluster: my-cluster 
    spec:
      partitions: 3
      replicas: 3
    EOF
    ```

您现在可以开始发送和接收消息了。

## 从一个主题开始发送和接收

### 使用外部 Red Hat 保险丝应用进行测试。您需要安装 Maven 和 Java 8 JDK。

1.  克隆这个 [git repo](https://github.com/hguerrero/amq-examples.git) 来测试对新 Kafka 集群的访问:

    `$ git clone https://github.com/hguerrero/amq-examples.git`

2.  Switch to the `camel-kafka-demo` folder.

    `$ cd amq-examples/camel-kafka-demo/`

3.  As we are using **Routes** for external access to the cluster, we need the cluster CA certificate to enable TLS in the client. Extract the public certificate of the broker certification authority.

    `$ oc extract secret/my-cluster-cluster-ca-cert --keys=ca.crt --to=- > src/main/resources/ca.crt`

4.  Import the trusted cert to a Keystore.

    `$ keytool -import -trustcacerts -alias root -file src/main/resources/ca.crt -keystore src/main/resources/keystore.jks -storepass password -noprompt`

5.  Now you can run the Red Hat Fuse application to send and receive messages to the Kafka cluster using the following maven command (this step currently requires Java 8):

    `$ mvn -Drun.jvmArguments="-Dbootstrap.server=`oc get routes my-cluster-kafka-external-bootstrap -o=jsonpath='{.status.ingress[0].host}{"\n"}'`:443" clean package spring-boot:run`

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

    你完了！按`Ctrl + C`停止正在运行的程序。

*Last updated: April 1, 2021*